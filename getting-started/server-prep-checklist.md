---
description: >-
  This is a checklist to follow chronologically in order to setup Servers from
  scratch
icon: square-check
---

# Server Prep Checklist

* [ ] Clean Install Debian OS on all Server Nodes
* [ ] Configure GRUB Settings
* [ ] Configure Automount Settings
* [ ] Install Required Packages
* [ ] Install Docker
* [ ] Exclude Docker from APT Updates
* [ ] Install Wireguard and Configure it for Mesh Network
* [ ] Install KeepAlived on Worker Cluster Nodes and Configure a VIP
* [ ] Install MariaDB on all nodes
* [ ] Configure Galera Cluster on all nodes
* [ ] Install NGINX with MariaDB as Data Source and Configure Reverse Proxy on BurgerWorker
* [ ] Configure APP DATA sync between the worker cluster nodes
* [ ] After the initial sync, install NGINX on other worker nodes
*
