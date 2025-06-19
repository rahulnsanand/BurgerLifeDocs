---
description: >-
  This page contains all the essential packages you'll need to setup/install
  before we proceed with any further application configuration and server
  creation.
---

# Essential Packages

#### Important Must-Have Packages

1. **curl** — Command-line tool for transferring data with URLs.
2. **git** — Version control system.
3. **wget** — Non-interactive network downloader.
4. **sudo** — Execute commands as another user (usually root).
5. **nano** — List open files.
6. **lsof** — List open files.
7. **tar** — Archive utility.
8. **rsync** — Fast file transfer and synchronization tool.
9. **locate** — Quickly find files by name.
10. **iptables -** To Help with Wireguard and set some rules
11. **python3.11-venv -** For Python Virtual Environments
12. **util-linux -** For Flock which makes sure a service runs only once at a time
13. **sqlite3** - for support on SQLite

#### Install Command

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install sqlite3 curl git wget sudo nano lsof tar rsync locate util-linux python3.11-venv iptables -y
```

