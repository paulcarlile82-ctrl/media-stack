# Enterprise Media Automation Stack

An enterprise-grade, containerized media automation pipeline deployed on Rocky Linux 9. Designed for high availability, absolute configuration isolation, and zero-overhead media processing through native hardlinking.

---

## Overview & Background

The evolution of this stack was driven by the classic infrastructure bottleneck: a legacy, single-node Jellyfin deployment running on traditional hard disk drives that buckled under heavy concurrent streaming and post-processing loads. Initial deployments suffered from fragmented volume paths, where download clients and media managers operated in isolated filesystem silos. This forced the system into slow, disk-heavy copy-and-delete operations upon every import, saturating I/O and slowing down overall node performance.

To solve this permanently, the entire architecture was torn down, re-engineered, and codified into a reproducible infrastructure-as-code pipeline using Docker Compose. The migration shifted the stack onto optimized SSD scratch space for high-velocity torrent ingestion, established a unified container path hierarchy to unlock zero-copy hardlinks, and baked strict Rocky Linux security compliance directly into the deployment logic.

---

## Quick Start & Cloning

To clone this repository and prepare your local environment for deployment, run:

```bash
git clone [https://github.com/paulcarlile82-ctrl/media-stack.git](https://github.com/paulcarlile82-ctrl/media-stack.git) && cd media-stack/media-stack
