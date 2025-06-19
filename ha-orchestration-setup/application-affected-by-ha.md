---
description: >-
  This is a list of applications and services that will be affected by High
  Availability Failover/Failback maneuver. These Application will be
  stopped/started according to the orchestration scripts.
icon: rotate
---

# Application Affected by HA

### Docker Containers  (Restart Policy - On-Failure)

1. **Immich PostgreSQL** - Manual Backups Needed
   1. Master Node Up | Manager Node Down
      1. Failover: Not Applicable - All Services Point to Master
      2. Failback: Once Manager Node connects to Master, Manager node will assume it's backup status
   2. Master Node Down | Manage Node Up
      1. Failover: Manager becomes the Primary PostgreSQL Node Automatically
      2. Failback: Once Master Node connects to Manager, Manager node's container will stop, giving master node it's primary status (\~15 seconds). Once done, Manager node will assume it's backup status
2. **Nginx Proxy Manager**
   1. Master Node Up | Manager Node Down
      1. Failover: Not Applicable - All Services Point to Master
      2. Failback: Not Applicable - All Services Point to Master
   2. Master Node Down | Manage Node Up
      1. Failover: Nginx container must be started
      2. Failback: Once Master Node connects to Manager, Manager node's container will stop
3. **Vaultwarden** - Manual Backups Needed
   1. Master Node Up | Manager Node Down
      1. Failover: Not Applicable - All Services Point to Master
      2. Failback: Not Applicable - All Services Point to Master
   2. Master Node Down | Manage Node Up
      1. Failover: Nginx container must be started
      2. Failback: Once Master Node connects to Manager, Manager node's container will stop
4. **NextCloud** (TBD) - Manual Backups Needed <mark style="color:red;">- MANUAL DOCKER COMMAND TO START/STOP</mark>
   1. Master Node Up | Manager Node Down
      1. Failover: Not Applicable - All Services Point to Master
      2. Failback: Not Applicable - All Services Point to Master
   2. Master Node Down | Manage Node Up
      1. Failover: Nginx container must be started
      2. Failback: Once Master Node connects to Manager, Manager node's container will stop
5. **Mealie** <mark style="color:red;">- CONVERT TO POSTGRESQL</mark>

### Linux Services  (Systemd Service Disabled)

1. SyncThing
   1. Master Node Up | Manager Node Down
      1. Failover: Not Applicable - All Services Point to Master
      2. Failback: Not Applicable - All Services Point to Master
   2. Master Node Down | Manage Node Up
      1. Failover: Manager Nodes "Receive Only" has Syncthing Running and Ready&#x20;
      2. Failback: Once Master is back, RSYNC will send back all updated data back to Master and only after that, Master's Syncthing will start syncthing "Send Only" data to Manager who will "Receive Only"
2. BurgerLife API - Manual Backups Needed
   1. Master Node Up | Manager Node Down
      1. Failover: Not Applicable - All Services Point to Master
      2. Failback: Not Applicable - All Services Point to Master
   2. Master Node Down | Manage Node Up
      1. Failover: Nginx container must be started
      2. Failback: Once Master Node connects to Manager, Manager node's container will stop

