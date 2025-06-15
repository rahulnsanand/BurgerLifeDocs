---
icon: sitemap
---

# Setup Galera Cluster

**Create the script file:**&#x20;

```bash
nano /tmp/setup_galera_node.sh
```

**Paste the script content:**

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

# Function to check if a package is installed on Raspbian OS
is_package_installed() {
  local package_name="$1"
  dpkg-query -l "$package_name" &> /dev/null
}

# Function to install MariaDB Server (including client tools and galera provider)
install_mariadb() {
  if ! is_package_installed mariadb-server; then
    print_message "${YELLOW}" "MariaDB Server is not installed. Installing now..."
    sudo apt-get update
    sudo apt-get install -y mariadb-server mariadb-client galera-4
    print_message ${GREEN} "MariaDB Server installation complete."
  else
    print_message ${GREEN} "MariaDB Server is already installed."
  fi
}

# Prompt the user for inputs
read -p "Enter current node IP address (e.g., 192.168.0.51): " node_ip
read -p "Enter current node name (e.g., n1): " node_name
read -p "Enter ALL cluster IP addresses, comma-separated (e.g., 192.168.0.51,192.168.0.52,192.168.0.53): " cluster_ips

# Construct the wsrep_cluster_address
wsrep_cluster_address="gcomm://$cluster_ips"

# Install MariaDB (if not already installed, this will also update apt lists)
install_mariadb

# Remove the existing galera.cnf file if it exists
conf_file="/etc/mysql/conf.d/galera.cnf"
if [ -f "$conf_file" ]; then
  sudo rm "$conf_file"
  print_message ${YELLOW} "Removed existing $conf_file"
fi

# Create the new galera.cnf file with the provided content
print_message "${YELLOW}" "Creating $conf_file with Galera Cluster configuration..."
cat <<EOF | sudo tee "$conf_file" > /dev/null
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="$wsrep_cluster_address"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="$node_ip"
wsrep_node_name="$node_name"
EOF

# Verify that the file has been created
if [ -f "$conf_file" ]; then
  print_message ${GREEN} "Created $conf_file with the provided content."
else
  print_message ${RED} "Failed to create $conf_file."
  exit 1 # Exit if config file creation fails
fi

# IMPORTANT: Secure installation BEFORE starting the cluster for the first time.
print_message "${YELLOW}" "Running mysql_secure_installation. Follow the prompts!"
print_message "${YELLOW}" "This is best done AFTER initial installation but BEFORE joining a cluster with data."
sudo mysql_secure_installation

# Stop the mariadb service (ensure it uses 'mariadb' not 'mysqld')
print_message "${YELLOW}" "Stopping mariadb service (pre-cluster start configuration)..."
sudo systemctl stop mariadb

print_message "${RED}" "---------------------------------------------------------"
print_message "${RED}" "MariaDB service is now configured and stopped. To start the Galera Cluster:"
print_message "${RED}" "---------------------------------------------------------"
print_message "${RED}" "ON THE FIRST NODE ONLY (e.g., n1):"
print_message "${RED}" "  sudo galera_new_cluster"
print_message "${RED}" "---------------------------------------------------------"
print_message "${RED}" "ON SUBSEQUENT NODES (e.g., n2, n3):"
print_message "${RED}" "  sudo systemctl start mariadb"
print_message "${RED}" "---------------------------------------------------------"
print_message "${RED}" "Verify cluster status with: sudo mysql -e 'SHOW STATUS LIKE \"wsrep_cluster_size\";'"
print_message "${RED}" "---------------------------------------------------------"
```

**Save the file:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Add execute permissions:**&#x42;ash

```bash
chmod +x /tmp/setup_galera_node.sh
```

**Execute the script:**

```bash
sudo /tmp/setup_galera_node.sh
```

**Follow the prompts:**

* **Enter current node IP address:** Provide the specific IP address of the current Raspberry Pi.
* **Enter current node name:** Provide a unique name for this node (e.g., `n1`, `n2`, `n3`).
* **Enter ALL cluster IP addresses, comma-separated:** Provide a comma-separated list of ALL WireGuard (or local LAN) IP addresses of all nodes that will be in the cluster. This list should be identical on every node.

**Complete `mysql_secure_installation`:** The script will initiate `mysql_secure_installation`. Follow the on-screen prompts carefully to:

* Enter current password for root (enter for none): \<ENTER DESIRED ROOT PASSWORD>
* Switch to unix\_socket authentication \[Y/n] n
* Change the root password? \[Y/n] n
* Remove anonymous users? \[Y/n] Y
* Disallow root login remotely? \[Y/n] Y
* Remove test database and access to it? \[Y/n] Y
* Reload privilege tables now? \[Y/n] Y

**Observe Script Output:** The script will complete by stopping the `mariadb` service and providing clear instructions on how to start the cluster, distinguishing between the first node and subsequent nodes.

#### **Forming the Galera Cluster**

Once you have run the script on all your intended nodes, follow these steps to bring the cluster online:

1.  **On the FIRST Node (e.g., `n1`):** This node will initiate the cluster. Run this command _only_ on the node you designate as the first one:

    ```bash
    sudo galera_new_cluster
    ```

    Wait for this node to start up fully.
2.  **On SUBSEQUENT Nodes (e.g., `n2`, `n3`):** For every other node you configured, simply start the MariaDB service:

    ```bash
    sudo systemctl start mariadb
    ```

    These nodes will automatically discover and join the existing cluster initiated by the first node.
3.  **Verify Cluster Status (on any node):** After all nodes have started, you can check the cluster size and status from any node:

    ```bash
    sudo mysql -e 'SHOW STATUS LIKE "wsrep_cluster_size";'
    ```

    The `Value` column should show the total number of active nodes in your cluster (e.g., `3`).

#### **Cleaning Up the Script (Temporary File)**

Once your Galera Cluster is successfully installed and running, you can remove the temporary script file from each node.

1.  **Remove the script (on each node):**

    ```bash
    rm /tmp/setup_galera_node.sh
    ```

{% hint style="info" %}
IF NODE GOES DOWN AND BACK UP AND IS INACCESSIBLE DIRECTLY PERFORM BELOW STEPS TO FORCE RECREATE AND JOIN CLUSTER
{% endhint %}

```bash
sudo systemctl stop mysql
sudo rm /var/lib/mysql/grastate.dat
sudo systemctl start mysql
```
