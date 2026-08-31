# Enterprise Media Automation Stack

An enterprise-grade, containerized media automation pipeline deployed on Rocky Linux 9. Designed for high availability, absolute configuration isolation, and zero-overhead media processing through native hardlinking.

---

## Overview & Background

The evolution of this stack was driven by the classic infrastructure bottleneck: a legacy, single-node Jellyfin deployment running on traditional hard disk drives that buckled under heavy concurrent streaming and post-processing loads. Initial deployments suffered from fragmented volume paths, where download clients and media managers operated in isolated filesystem silos. This forced the system into slow, disk-heavy copy-and-delete operations upon every import, saturating I/O and slowing down overall node performance.

To solve this permanently, the entire architecture was torn down, re-engineered, and codified into a reproducible infrastructure-as-code pipeline using Docker Compose. The migration shifted the stack onto optimized SSD scratch space for high-velocity torrent ingestion, established a unified container path hierarchy to unlock zero-copy hardlinks, and baked strict Rocky Linux security compliance directly into the deployment logic.

---

## Architecture Stack

- **Jellyfin**: High-performance media server serving streaming front-end logic via unified mount configurations (`/data/media`).
- **Sonarr & Radarr**: Automated TV series and movie management engines configured for instant atomic hardlink ingestion.
- **Prowlarr**: Centralized indexer aggregator managing tracker synchronization and health checks across the pipeline.
- **qBittorrent**: Dedicated download client running behind secured DNS resolvers, routing raw data directly to high-speed NVMe/SSD storage arrays (`/data/downloads`).

---

## Technical Breakdown & Troubleshooting

### 1. Hardlink & Atomic Move Optimization
* **The Problem:** Separating container volume mount paths (e.g., mapping downloads to `/downloads` and movies to `/movies`) causes the host system to view them across distinct virtual filesystems. When Sonarr or Radarr processes a completed download, it cannot create a hardlink (which requires files to reside on the same physical inode-sharing filesystem); instead, it performs a resource-intensive physical file copy.
* **The Solution:** The compose architecture was restructured to introduce a shared root path baseline (`/data/media` and `/data/downloads`) inside the media management containers. This tricks the *arr suite into treating downloads and libraries as a single cohesive filesystem, resulting in instantaneous file relocation with zero disk duplication.

### 2. Rocky Linux 9 SELinux Compliance (`:z`)
* **The Problem:** Rocky Linux enforces strict Security-Enhanced Linux (SELinux) policies by default, blocking container runtimes (Podman/Docker) from reading or writing to host-mounted directories due to permission context mismatches (resulting in standard `Permission Denied` errors on startup).
* **The Solution:** Every host volume mount across the stack explicitly incorporates the `:z` SELinux context suffix. This instructs the host runtime to automatically relabel the target directories, ensuring seamless container interaction without having to globally weaken system-wide security enforcements.

### 3. State Management & Configuration Security
* **Persistence Layer:** All critical application state, database records, and user preferences are mapped to isolated, externally managed Docker volumes (`jellyfin-config`, `sonarr-config`, etc.), decoupling state from ephemeral container lifecycles.
* **Repository Hygiene:** Environment variables, sensitive credentials, and local data nodes are strictly excluded via `.gitignore`, ensuring the public repository remains a pristine, enterprise-ready infrastructure template.

---

## Operational Maintenance & Day-2 Playbook

If you are planning to deploy this stack on a live server, please review the operational maintenance guidelines below to ensure long-term system stability and handle routine updates safely:

### 1. Routine Maintenance & Upgrades
* **Container Lifecycle Updates:** Upgrading container images should be handled sequentially to prevent database locking or config corruption:
  ```bash
  podman compose pull
  podman compose down
  podman compose up -d 

Storage & Inode Monitoring: Periodically check inode utilization and disk space on your high-speed storage paths:
Bash

    df -h /mnt/ssd/downloads
    df -i /var/lib/media

2. Common Failure Modes & Recovery

    SELinux Contexts: If permission errors occur after modifying host files, relabel the volume paths using:
    Bash

restorecon -Rvv /var/lib/media
restorecon -Rvv /mnt/ssd/downloads

Runtime Inspection: When a service stalls or crashes during startup, inspect persistent container logs rather than relying on standard output:
Bash

    podman logs -f <container_name>
