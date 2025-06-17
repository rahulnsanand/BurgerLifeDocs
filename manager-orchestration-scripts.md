---
icon: square-terminal
---

# Manager Orchestration Scripts

## RUN THE ORCHESTRATION AS ROOT

### **On \[**<mark style="color:green;">**MANAGER**</mark>**] Server**

### Manager's Master Takeover Script - `manager_start_apps_on_takeover.sh`

This script will be responsible for starting your Nginx Docker container (and potentially Syncthing or other services) on the Manager when it becomes active.

```bash
su
mkdir -p /etc/keepalived/scripts
```

```bash
nano /etc/keepalived/scripts/manager_start_apps_on_takeover.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

```bash
#!/bin/bash
set -euo pipefail

LOGFILE="/var/log/keepalived_manager_takeover.log" # Dedicated log for Manager's takeover actions

# --- Configuration: List of Docker containers to manage ---
# Add or remove container names from this array as needed.
declare -a DOCKER_CONTAINERS=(
    "plex" 
    "jellyfin" 
    "asherslife-tunnel"
    "burgerlife-tunnel"
    "rahulanand-tunnel"
    "bookstack"
    "immich"
)

echo "$(date): Manager transitioned to MASTER state. Starting local application services." | tee -a "$LOGFILE"

# --- Start Docker Containers ---
echo "$(date): Starting specified Docker containers..." | tee -a "$LOGFILE"
for container_name in "${DOCKER_CONTAINERS[@]}"; do
    echo "$(date): Starting Docker container: ${container_name}" | tee -a "$LOGFILE"
    docker start "$container_name"

    # Add a health check for each container
    HEALTH_CHECK_RETRIES=10
    HEALTH_CHECK_DELAY=2
    CONTAINER_STARTED=false
    for i in $(seq 1 $HEALTH_CHECK_RETRIES); do
        if docker inspect -f '{{.State.Running}}' "$container_name" | grep -q "true"; then
            echo "$(date): Container ${container_name} is running." | tee -a "$LOGFILE"
            CONTAINER_STARTED=true
            break
        fi
        echo "$(date): Waiting for container ${container_name} to start... (${i}/${HEALTH_CHECK_RETRIES})" | tee -a "$LOGFILE"
        sleep "$HEALTH_CHECK_DELAY"
    done

    if [ "$CONTAINER_STARTED" = false ]; then
        echo "$(date): ERROR: Docker container ${container_name} failed to start on Manager after multiple attempts." | tee -a "$LOGFILE"
        exit 1 # Exit if any critical container fails to start
    fi
done

echo "$(date): All specified application services started successfully on Manager." | tee -a "$LOGFILE"

exit 0
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /etc/keepalived/scripts/manager_start_apps_on_takeover.sh
```

### Manager's Rsync Script - `manager_perform_rsync.sh`

This script is called remotely by the Master.

```bash
nano /usr/local/bin/manager_perform_rsync.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

```bash
#!/bin/bash
set -euo pipefail

LOGFILE="/var/log/keepalived_failback.log"
MASTER_IP="10.21.22.15" # Replace with Master's Wireguard IP
MASTER_USER="root"     # SSH user on Master (ensure this user has appropriate permissions on Master)

# --- Configuration: Docker containers to temporarily stop before rsync ---
# These containers will be stopped on the MANAGER to ensure data consistency during rsync.
# List containers whose data directories are being synced.
declare -a CONTAINERS_TO_STOP_FOR_SYNC=(
    "plex" 
    "jellyfin" 
    "asherslife-tunnel"
    "burgerlife-tunnel"
    "rahulanand-tunnel"
    "bookstack"
    "immich"
)

# --- Configuration: Directories to rsync from Manager to Master ---
# List source directories on the MANAGER that need to be synced to the MASTER.
# Ensure the SSH user on Master has write permissions to the destination paths.
declare -a DIRECTORIES_TO_SYNC=("/opt/plex/" 
    "/opt/immich/machine_learning/" 
    "/opt/immich/redis/"
    "/opt/bookstack/"
    "/opt/BurgerLifeData/"
    "/opt/filebrowser/"
    "/opt/FileShare/"
    "/opt/gluetun/"
    "/opt/n8n/"
    "/opt/homepage/"
    "/opt/onlyoffice/"
)

echo "$(date): Manager received request to perform rsync to Master ($MASTER_IP)." | tee -a "$LOGFILE"

# --- Stop Docker Containers for Data Consistency (on Manager) ---
echo "$(date): Stopping specified Docker containers on Manager for data consistency during rsync..." | tee -a "$LOGFILE"
for container_name in "${CONTAINERS_TO_STOP_FOR_SYNC[@]}"; do
    if docker ps -a --format '{{.Names}}' | grep -Eq "^${container_name}$"; then
        echo "$(date): Stopping container: ${container_name}." | tee -a "$LOGFILE"
        docker stop "$container_name" || {
            echo "$(date): WARNING: Failed to stop container ${container_name}. This might affect data consistency." | tee -a "$LOGFILE"
        }
    else
        echo "$(date): Container ${container_name} not found or not running. Skipping stop." | tee -a "$LOGFILE"
    fi
done
# Give containers a moment to fully shut down
sleep 3

# --- Perform Rsync for each specified directory ---
echo "$(date): Starting rsync for specified directories from Manager to Master ($MASTER_IP)." | tee -a "$LOGFILE"
for source_dir in "${DIRECTORIES_TO_SYNC[@]}"; do
    # Ensure source_dir ends with a slash for rsync to copy contents, not the directory itself
    # Check if the source directory exists
    if [ ! -d "$source_dir" ]; then
        echo "$(date): WARNING: Source directory '${source_dir}' not found on Manager. Skipping rsync for this path." | tee -a "$LOGFILE"
        continue # Skip to the next directory
    fi

    # Determine destination path. Assumes same path on Master.
    destination_dir="${source_dir}"

    echo "$(date): Rsyncing '${source_dir}' to '${MASTER_USER}@${MASTER_IP}:${destination_dir}'" | tee -a "$LOGFILE"
    if ! rsync -avz --delete "$source_dir" "$MASTER_USER@$MASTER_IP:${destination_dir}" | tee -a "$LOGFILE"; then
        echo "$(date): ERROR: Rsync failed for directory '${source_dir}'." | tee -a "$LOGFILE"
        exit 1 # Exit on first rsync failure, as data might be inconsistent
    fi
    echo "$(date): Rsync completed for '${source_dir}'." | tee -a "$LOGFILE"
done

echo "$(date): All specified rsync operations completed successfully on Manager." | tee -a "$LOGFILE"

# The Master's script (notify_master.sh) handles creating the sync_complete_flag
# on the Manager after this script successfully finishes.
# No need to touch it here explicitly.

exit 0
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /usr/local/bin/manager_perform_rsync.sh
```

### Manager's VIP Release Script - `manager_release_vip.sh`&#x20;

This script is called by `notify_master.sh` after the data sync.

```bash
nano /usr/local/bin/manager_release_vip.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

```bash
#!/bin/bash
set -euo pipefail

LOGFILE="/var/log/keepalived_failback.log"

echo "$(date): Manager received request to release VIP. Stopping Keepalived for 15 seconds." | tee -a "$LOGFILE"

# Gracefully stop Keepalived to release the VIP
# This will cause the Master (with `nopreempt`) to take over if its priority is higher.
systemctl stop keepalived
if ! systemctl is-active --quiet keepalived; then
    echo "$(date): Keepalived service successfully stopped on Manager." | tee -a "$LOGFILE"
else
    echo "$(date): WARNING: Keepalived service might not have stopped completely." | tee -a "$LOGFILE"
fi

# Wait for a period to ensure VIP has fully transferred and network converges
echo "$(date): Waiting for 15 seconds for VIP transition." | tee -a "$LOGFILE"
sleep 15

# Restart Keepalived on Manager to bring it back as BACKUP
echo "$(date): Restarting Keepalived on Manager to resume BACKUP role." | tee -a "$LOGFILE"
systemctl start keepalived
if ! systemctl is-active --quiet keepalived; then
    echo "$(date): ERROR: Keepalived service failed to restart on Manager." | tee -a "$LOGFILE"
    exit 1
fi

echo "$(date): Keepalived restarted on Manager. VIP should now be on Master." | tee -a "$LOGFILE"

exit 0
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /usr/local/bin/manager_release_vip.sh
```
