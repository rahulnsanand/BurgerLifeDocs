---
icon: square-terminal
---

# Master Orchestration Scripts

## RUN THE ORCHESTRATION AS ROOT

### **On \[**<mark style="color:red;">**MASTER**</mark>**] Server**

### Keepalived Trigger - `notify_master.sh`

This script is triggered when the Master's Keepalived transitions to `MASTER` state. Its primary role here is to initiate the failback process on the Manager and _wait_ for the Manager to complete its part.

```bash
su
mkdir -p /etc/keepalived/scripts
nano /etc/keepalived/scripts/notify_master.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

```bash
#!/bin/bash
set -euo pipefail

LOGFILE="/var/log/keepalived_failback.log"
MANAGER_IP="10.21.22.20" # Replace with Manager's Wireguard IP
MASTER_IP="10.21.22.10"   # Replace with Master's Wireguard IP
MANAGER_USER="root"        # SSH user on Manager
MASTER_APP_RESTART_SCRIPT="/usr/local/bin/master_restart_apps.sh" # Script to start apps on Master
SYNC_COMPLETE_FILE="/tmp/rsync_complete_flag" # Temporary flag file on Manager

echo "$(date): Master transitioned to MASTER state. Initiating failback procedure." | tee -a "$LOGFILE"

# Clean up any stale sync complete flag from previous attempts (if any)
ssh "$MANAGER_USER@$MANAGER_IP" "rm -f $SYNC_COMPLETE_FILE" | tee -a "$LOGFILE"

# --- Step 1: Master triggers Manager for Rsync ---
# Execute the Manager's rsync script remotely and wait for it to complete.
# The Manager's script will create SYNC_COMPLETE_FILE on itself upon success.
echo "$(date): SSHing to Manager ($MANAGER_IP) to start rsync and wait for completion." | tee -a "$LOGFILE"
if ! ssh "$MANAGER_USER@$MANAGER_IP" "bash -c '/usr/local/bin/manager_perform_rsync.sh && touch $SYNC_COMPLETE_FILE'" | tee -a "$LOGFILE"; then
    echo "$(date): ERROR: Failed to initiate or complete rsync on Manager." | tee -a "$LOGFILE"
    exit 1
fi

# --- Step 2: Master waits for Manager's rsync and then starts its own containers ---
echo "$(date): Rsync initiated on Manager. Waiting for $SYNC_COMPLETE_FILE on Manager..." | tee -a "$LOGFILE"

# Wait for the flag file to appear on the Manager
TIMEOUT=300 # 5 minutes timeout
ELAPSED=0
while [ ! -f "$SYNC_COMPLETE_FILE" ]; do
    echo "$(date): Waiting for $SYNC_COMPLETE_FILE on Manager... (Elapsed: ${ELAPSED}s)" | tee -a "$LOGFILE"
    sleep 5
    ELAPSED=$((ELAPSED+5))
    if [ $ELAPSED -ge $TIMEOUT ]; then
        echo "$(date): ERROR: Timeout waiting for rsync completion flag from Manager." | tee -a "$LOGFILE"
        exit 1
    fi
    # Check if the flag file exists on the Manager via SSH
    if ssh "$MANAGER_USER@$MANAGER_IP" "test -f $SYNC_COMPLETE_FILE"; then
        break
    fi
done

echo "$(date): Rsync complete flag detected on Manager. Starting local application services." | tee -a "$LOGFILE"
if ! "$MASTER_APP_RESTART_SCRIPT" | tee -a "$LOGFILE"; then
    echo "$(date): ERROR: Failed to restart applications on Master." | tee -a "$LOGFILE"
    exit 1
fi

# --- Step 3: Master tells Manager to release VIP ---
# After Master's apps are up, tell Manager to stop Keepalived and release VIP.
echo "$(date): Master apps restarted. Telling Manager ($MANAGER_IP) to release VIP." | tee -a "$LOGFILE"
if ! ssh "$MANAGER_USER@$MANAGER_IP" "/usr/local/bin/manager_release_vip.sh" | tee -a "$LOGFILE"; then
    echo "$(date): ERROR: Failed to instruct Manager to release VIP." | tee -a "$LOGFILE"
    exit 1
fi

echo "$(date): Failback orchestration complete. VIP should now be on Master." | tee -a "$LOGFILE"

exit 0
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /etc/keepalived/scripts/notify_master.sh
```

### Master's App Restart Script - `master_restart_apps.sh`&#x20;

This script is called by `notify_master.sh` after the data sync.

```bash
nano /usr/local/bin/master_restart_apps.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

```bash
#!/bin/bash
set -euo pipefail

LOGFILE="/var/log/keepalived_failback.log"

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

# --- Configuration: List of Systemd services to manage ---
# Ensure these service names are correct for your system.
# For Syncthing, remember to use the full service name, e.g., "syncthing@<username>.service"
declare -a SYSTEMD_SERVICES=("syncthing@root.service")


echo "$(date): Initiating application service restart on Master." | tee -a "$LOGFILE"

# --- Stop Docker Containers ---
echo "$(date): Stopping specified Docker containers (if running)..." | tee -a "$LOGFILE"
for container_name in "${DOCKER_CONTAINERS[@]}"; do
    if docker ps -a --format '{{.Names}}' | grep -Eq "^${container_name}$"; then
        echo "$(date): Stopping Docker container: ${container_name}" | tee -a "$LOGFILE"
        docker stop "$container_name" || {
            echo "$(date): WARNING: Failed to stop container ${container_name}. Continuing..." | tee -a "$LOGFILE"
        }
    else
        echo "$(date): Container ${container_name} not found or not running. Skipping stop." | tee -a "$LOGFILE"
    fi
done
# Give containers a moment to fully shut down
sleep 3

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
        echo "$(date): ERROR: Docker container ${container_name} failed to start on Master after multiple attempts." | tee -a "$LOGFILE"
        exit 1 # Exit if any critical container fails to start
    fi
done

# --- Start Systemd Services ---
echo "$(date): Starting specified Systemd services..." | tee -a "$LOGFILE"
for service_name in "${SYSTEMD_SERVICES[@]}"; do
    echo "$(date): Starting Systemd service: ${service_name}" | tee -a "$LOGFILE"
    systemctl start "$service_name"
    if ! systemctl is-active --quiet "$service_name"; then
        echo "$(date): ERROR: Systemd service ${service_name} failed to start on Master." | tee -a "$LOGFILE"
        exit 1 # Exit if any critical service fails to start
    fi
    echo "$(date): Systemd service ${service_name} started successfully." | tee -a "$LOGFILE"
done

echo "$(date): All specified application services processed successfully on Master." | tee -a "$LOGFILE"

exit 0
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /usr/local/bin/master_restart_apps.sh
```
