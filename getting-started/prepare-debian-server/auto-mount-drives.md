---
description: >-
  This guide will walk you through the entire process of auto-mounting a drive
  at boot using /etc/fstab in the safest manner possible without bricking your
  server in case drive is missing or corrupted
---

# Auto Mount Drives

#### Identify the Drive's UUID

```bash
lsblk -f
# or
sudo blkid
```

#### Create a Mount Point

```bash
sudo mkdir -p /mnt/primary
```

#### Backup existing `/etc/fstab`

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

#### Edit `/etc/fstab`

```bash
sudo nano /etc/fstab
```

#### Add your mount entry at the end of the file:

```bash
UUID=05c35f75-9662-42aa-b6c6-8bafe6a1acff /mnt/primary ext4 defaults,nofail,x-systemd.automount,x-systemd.device-timeout=10s 0 2
```

* **Explanation of options:**
  * `UUID=...` — Unique identifier for the partition.
  * `/mnt/primary` — Mount point.
  * `ext4` — Filesystem type.
  * `defaults` — Standard mount options.
  * `nofail` — Boot continues even if the drive is missing.
  * `x-systemd.automount` — Mount on first access, not at boot.
  * `x-systemd.device-timeout=10s` — Wait 10 seconds for device.
  * `0 2` — Dump and fsck options (standard for non-root ext4).

> Remember, this mount configuration is safe and robust, it does not mount the drive to the mount location UNLESS a user or an application tries to access that mount location.&#x20;

#### Save and close the file (`Ctrl+O`, `Enter`, then `Ctrl+X` in nano).

#### Reload Daemon to reflect changes and Test mounts

```bash
sudo systemctl daemon-reload
sudo mount -a
```

#### Check if drive is mounted

```bash
df -h | grep /mnt/primary
```

#### &#x20;



