---
icon: circle-exclamation
---

# Important Commands

### Archive a folder using TAR

```bash
sudo tar -czvf /mnt/backup/opt_borg_earlyjune.tar.gz /mnt/backup/opt_borg/
```

### Un-Archive a tar.gz

```bash
sudo tar -xzf /mnt/backup/opt_borg_earlyjune.tar.gz
```

### RSYNC File To Another Server (Create Destination Directory, if needed)

```bash
sudo rsync -avz --mkpath /mnt/backup/opt_borg_earlyjune.tar.gz burger@10.10.10.20:/opt/
```

```bash
sudo rsync -av --mkpath /mnt/backup/opt_borg/ burger@10.10.10.20:/opt/opt_borg/
```

### Move Multiple Folders to Same Destination

```bash
sudo mv -t /mnt/backup/ projects/work/report projects/personal/photos projects/archive/old_docs
```

### KeepAlived Failover/Failback Logs

```bash
sudo cat /var/log/keepalived_logs.log
```

### Network Interface IP Check

```bash
ip addr show wg0
```

### MariaDB Galera Cluster Check

```bash
sudo mysql -e 'SHOW STATUS LIKE "wsrep_cluster_size";'
```
