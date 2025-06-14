---
description: >-
  This page covers the comprehensive plan for data security, redundancy and
  backup plan for this homelab server.
icon: database
---

# Backup Plan

#### Overview

<table><thead><tr><th width="264.20001220703125">Storage Solution</th><th width="121.800048828125">Kind of Data</th><th>Backup/Replication</th><th>Application Used</th></tr></thead><tbody><tr><td>Master 4TB NAS HDD</td><td>Media Files</td><td>Replicated - 4TB</td><td>Syncthing</td></tr><tr><td>Manager 4TB NAS HDD</td><td>Media Files</td><td>Replicated - 4TB</td><td>Syncthing/Rsync</td></tr><tr><td>Master 500GB NVME</td><td>App Data</td><td>Replicated - NVME</td><td>Syncthing</td></tr><tr><td>Manager 500GB NVME</td><td>App Data</td><td>Replicated - NVME</td><td>Syncthing/Rsync</td></tr><tr><td>Worker 500GB NVME</td><td>App Data</td><td>Replicated - NVME</td><td>Syncthing</td></tr><tr><td>WorkerManager 64GB SD Card</td><td>App Data</td><td>Replicated - NVME</td><td>Syncthing</td></tr><tr><td>WorkerSlave 64GB SD Card</td><td>App Data</td><td>Replicated - NVME</td><td>Syncthing</td></tr></tbody></table>

#### [Scenario 1 - Normal Usage](scenario-1-normal-usage.md)

#### [Scenario 2 - Master Server Down](scenario-2-master-server-down.md)

#### [Scenario 3 - Manager Server Down](scenario-3-manager-server-down.md)

#### [Scenario 4 - Worker Server Down](scenario-4-worker-server-down.md)

#### [Scenario 5 - Bangalore Off Network](scenario-5-bangalore-off-network.md)

#### [Scenario 6 - Hosur Off Network](scenario-6-hosur-off-network.md)

#### [Scenario 7 - Master NAS HDD Down](scenario-7-master-nas-hdd-down.md)

#### [Scenario 8 - Manager NAS HDD Down](scenario-8-manager-nas-hdd-down.md)



