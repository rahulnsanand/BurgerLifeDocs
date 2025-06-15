---
icon: database
---

# Setup PostgreSQL

**Create the `deploy_immich_pg.sh` file:** Use `nano` to create the file in the `/tmp` directory on each node:

```bash
nano /tmp/deploy_immich_pg.sh
```

**Paste the dynamic script content:** Copy the following content and paste it into the `nano` editor.

```bash
#!/bin/bash

# Define ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
RESET='\033[0m'

# Function to print colored messages
print_message() {
  local color="$1"
  local message="$2"
  echo -e "${color}${message}${RESET}"
}

# --- Configuration Variables (Fixed or Derived) ---
CONTAINER_NAME="immich_postgres"
IMAGE_NAME="rahulnsanand/burgerlife:pg_rpm_vct_1.0"
PG_PORT="5432"
VOLUME_PATH="/opt/immich/postgresql"
POSTGRESQL_DB_NAME="immichdb" # Keeping this fixed as per your YAML

# --- Secure Input Prompts ---
print_message "${YELLOW}" "--- Repmgr PostgreSQL Node Configuration ---"
read -p "Enter this node's IP address (e.g., 192.168.0.51 or 10.0.0.1): " NODE_IP
read -p "Enter this node's unique name (e.g., burgermaster-0, burgermaster-1): " NODE_NAME
read -p "Enter this node's unique ID (e.g., 51, 52, 53, 54, 55): " NODE_ID
read -p "Enter this node's repmgr priority (e.g., 150 for primary, 100 for standby, 50 for another standby): " NODE_PRIORITY
read -p "Enter the IP address of the *initial primary* host (this node's IP if it's the primary, otherwise the active primary's IP): " PRIMARY_HOST_IP
read -p "Enter ALL partner node IPs, comma-separated (e.g., 192.168.0.51,192.168.0.52,192.168.0.53,192.168.0.54,192.168.0.55): " PARTNER_NODES_RAW

read -s -p "Enter the REPMGR user password (securely): " REPMGR_PASS
echo "" # New line after silent input
read -s -p "Enter the PostgreSQL 'postgres' user password (securely): " PG_PASS
echo "" # New line after silent input

# Format PARTNER_NODES for REPMGR_PARTNER_NODES environment variable
# Converts "ip1,ip2" to "ip1:5432,ip2:5432"
REPMGR_PARTNER_NODES=$(echo "$PARTNER_NODES_RAW" | sed "s/,/:$PG_PORT,/g" | sed "s/$/:$PG_PORT/")

# --- Dynamic Docker Compose Content ---
DOCKER_COMPOSE_CONTENT=$(cat <<EOF
version: '3.8'
services:
  immich_postgres:
    image: $IMAGE_NAME
    container_name: $CONTAINER_NAME
    ports:
      - "$PG_PORT:$PG_PORT"
    environment:
      REPMGR_PARTNER_NODES: "$REPMGR_PARTNER_NODES"
      REPMGR_NODE_NAME: "$NODE_NAME"
      REPMGR_NODE_NETWORK_NAME: "$NODE_IP"
      REPMGR_NODE_ID: "$NODE_ID"
      REPMGR_PRIMARY_HOST: "$PRIMARY_HOST_IP"
      REPMGR_PRIMARY_PORT: $PG_PORT
      REPMGR_PASSWORD: "$REPMGR_PASS"
      POSTGRESQL_PASSWORD: "$PG_PASS"
      REPMGR_NODE_TYPE: "data" # This is typical for repmgr nodes
      POSTGRESQL_SHARED_PRELOAD_LIBRARIES: 'repmgr, pgaudit, vectors.so'
      POSTGRESQL_DATABASE: "$POSTGRESQL_DB_NAME"
      REPMGR_NODE_PRIORITY: "$NODE_PRIORITY"
    volumes:
      - "$VOLUME_PATH:/bitnami/postgresql"
    restart: unless-stopped
EOF
)

# Define the Docker Compose file path
COMPOSE_FILE="/tmp/immich_postgres_docker-compose.yml"

# --- Deployment Steps ---

# Stop and remove existing container if it exists
print_message "${YELLOW}" "Stopping and removing any existing '$CONTAINER_NAME' container..."
sudo docker-compose -f "$COMPOSE_FILE" down --remove-orphans &>/dev/null || sudo docker rm -f "$CONTAINER_NAME" &>/dev/null || true

# Create the Docker Compose file
print_message "${YELLOW}" "Creating Docker Compose file: $COMPOSE_FILE..."
echo "$DOCKER_COMPOSE_CONTENT" | sudo tee "$COMPOSE_FILE" > /dev/null
print_message "${GREEN}" "Docker Compose file created successfully."

# Deploy the container
print_message "${YELLOW}" "Deploying '$CONTAINER_NAME' container..."
sudo docker-compose -f "$COMPOSE_FILE" up -d || { print_message "${RED}" "Docker Compose deployment failed!"; exit 1; }
print_message "${GREEN}" "Container '$CONTAINER_NAME' deployed."

# --- Initial Repmgr Setup Guidance ---
print_message "${YELLOW}" "--- IMPORTANT REPMGR INITIALIZATION ---"
print_message "${YELLOW}" "If this is the FIRST node (your initial primary), after this script finishes, you need to:"
print_message "${YELLOW}" "1. Wait a few minutes for the container to fully initialize PostgreSQL."
print_message "${YELLOW}" "2. Register it as the primary. From your SSH session, run:"
print_message "${GREEN}" "   sudo docker exec $CONTAINER_NAME repmgr primary register -f /opt/bitnami/postgresql/conf/repmgr/repmgr.conf"
print_message "${YELLOW}" ""
print_message "${YELLOW}" "For SUBSEQUENT nodes (replicas), after this script finishes, you need to:"
print_message "${YELLOW}" "1. Wait a few minutes for the container to fully initialize PostgreSQL."
print_message "${YELLOW}" "2. Clone from the primary and register. From your SSH session, run (replace <PRIMARY_IP>):"
print_message "${GREEN}" "   sudo docker exec $CONTAINER_NAME repmgr standby clone -h $PRIMARY_HOST_IP -d $POSTGRESQL_DB_NAME -U repmgr -F --force"
print_message "${GREEN}" "   sudo docker exec $CONTAINER_NAME repmgr standby register -f /opt/bitnami/postgresql/conf/repmgr/repmgr.conf"
print_message "${YELLOW}" ""
print_message "${YELLOW}" "You can verify cluster status from any node using:"
print_message "${GREEN}" "   sudo docker exec $CONTAINER_NAME repmgr cluster show"

print_message "${GREEN}" "--- PostgreSQL Repmgr setup initiated. Follow the above steps. ---"

# Unset sensitive variables
unset REPMGR_PASS
unset PG_PASS
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Add execute permissions:**&#x42;ash

```bash
chmod +x /tmp/deploy_immich_pg.sh
```

#### **Run the Script (on EACH Node) - FOLLOW DEPLOYMENT ORDER!**

**Crucial Deployment Order:**

1. **Choose your initial Primary Node.** This will be the first node on which you run this script.
2. **Run the script on the Primary Node first.**
3. **Then, run the script on each Replica Node.**

**Execution Steps (Repeat for each node):**

```bash
sudo /tmp/deploy_immich_pg.sh
```

**Follow the prompts carefully for&#x20;**_**each**_**&#x20;node:** All IP addresses you provide to the script must be the **WireGuard IP addresses** of your nodes, not their physical LAN IPs.

* **"Enter this node's IP address"**: The IP of the current Raspberry Pi you're setting up.
* **"Enter this node's unique name"**: A distinct name like `burgermaster-0`, `burgermaster-1`, etc.
* **"Enter this node's unique ID"**: A distinct number like `51`, `52`, `53`, `54`, `55`.
* **"Enter this node's repmgr priority"**:
  * For your **initial primary**, use the highest priority (e.g., `150`).
  * For **subsequent replicas**, use lower, unique priorities (e.g., `100`, `90`, `80`, `70`, `60`).
* **"Enter the IP address of the&#x20;**_**initial primary**_**&#x20;host"**:
  * **For the FIRST node (your intended initial primary):** Enter its _own_ IP address.
  * **For SUBSEQUENT nodes (replicas):** Enter the IP address of the node that is _already established_ as the primary.
* **"Enter ALL partner node IPs, comma-separated"**: This should be a comma-separated list of the IPs of _all_ 5 nodes in your cluster (e.g., `192.168.0.51,192.168.0.52,192.168.0.53,192.168.0.54,192.168.0.55`). This must be the **same input on all nodes**.
* **"Enter the REPMGR user password"**: A strong password for `repmgr`. This must be the **same on all nodes**.
* **"Enter the PostgreSQL 'postgres' user password"**: A strong password for the `postgres` superuser. This must be the **same on all nodes**.

#### **Repmgr Cluster Initialization (Crucial Post-Deployment Steps)**

After running the script on a node, you **must** perform additional `docker exec` commands to initialize the `repmgr` cluster:

**A. On your designated initial Primary Node (`10.0.0.1` in the example):**

1. **Wait** a few minutes for the container to fully start PostgreSQL and `repmgr` internally.
2.  **Register the primary:**&#x42;ash

    ```bash
    sudo docker exec immich_postgres repmgr primary register -f /opt/bitnami/postgresql/conf/repmgr/repmgr.conf
    ```

    You should see output confirming successful registration.

**B. On each Subsequent Replica Node (`10.0.0.2`, `10.0.0.3`, etc.):**

1. **Wait** a few minutes for the container to fully start PostgreSQL.
2.  **Clone from the primary and register:** _(Replace `<PRIMARY_WIREGUARD_IP>` with the actual WireGuard IP address of your active primary node, e.g., `10.0.0.1`)_&#x42;ash

    ```bash
    sudo docker exec immich_postgres repmgr standby clone -h <PRIMARY_WIREGUARD_IP> -d immichdb -U repmgr -F --force
    sudo docker exec immich_postgres repmgr standby register -f /opt/bitnami/postgresql/conf/repmgr/repmgr.conf
    ```

    This process will stop the PostgreSQL instance on the replica, wipe its data, clone from the primary, and then register itself with the `repmgr` cluster. This is crucial for replication to start.

#### **Verify Repmgr Cluster Status**

From any node, after completing the initial setup for all nodes:

```bash
sudo docker exec immich_postgres repmgr cluster show
```

You should see a table listing all your nodes (primary and standbys) with their correct roles and health status, all using their WireGuard IP addresses for communication.



