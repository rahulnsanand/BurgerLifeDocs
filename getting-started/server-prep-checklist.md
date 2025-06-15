---
description: >-
  This is a checklist to follow chronologically in order to setup Servers from
  scratch
icon: square-check
---

# Server Prep Checklist

* [x] \[All Nodes] Clean Install Debian OS
* [x] \[Master/Manager Nodes] Configure GRUB Settings
* [x] \[Master/ManagerNodes] Configure Automount Settings
* [x] \[All Nodes] Install Required Packages
* [x] \[All Nodes] Install Docker
* [x] \[All Nodes] Exclude Docker from APT Updates
* [x] \[All Nodes] Install Wireguard and Configure it for Mesh Network
* [x] \[Worker Cluster] Install KeepAlived and Configure a VIP
* [x] \[All Nodes] Install MariaDB
* [x] \[All Nodes] Configure Galera Cluster on all nodes
* [x] \[Worker Cluster] Install NGINX with MariaDB as Data Source and Configure Reverse Proxy
* [x] \[Worker Cluster] After the initial sync, install NGINX
* [x] \[Worker Cluster] Configure APP DATA sync
*
*
*
*
