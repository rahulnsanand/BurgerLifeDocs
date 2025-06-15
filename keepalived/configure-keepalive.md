---
icon: sitemap
---

# Configure KeepAlive

**Create the `deploy_keepalived.sh` file:** Use `nano` to create a new file:

```bash
nano /tmp/deploy_keepalived.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

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

# Function to check if a package is installed
is_package_installed() {
  local package_name="$1"
  dpkg-query -l "$package_name" &> /dev/null
}

# Function to uninstall Keepalived
uninstall_keepalived() {
  if is_package_installed keepalived; then
    print_message ${RED} "Stopping and Uninstalling Keepalived..."
    sudo systemctl stop keepalived || true # Use || true to prevent script exit if service not running
    sudo apt-get remove --purge keepalived -y
    sudo rm -rf /etc/keepalived
    print_message ${GREEN}  "Keepalived and its configuration have been uninstalled."
  else
    print_message ${YELLOW} "Keepalived is not installed."
  fi
}

# Function to install Keepalived
install_keepalived() {
  print_message "${YELLOW}" "Installing Keepalived..."
  sudo apt-get update && sudo apt-get install keepalived -y
  print_message "${GREEN}" "Keepalived installation complete."
}

# Function to start Keepalived
start_keepalived() {
  print_message "${YELLOW}" "Enabling and Starting Keepalived..."
  sudo systemctl enable --now keepalived.service
  sudo systemctl restart --now keepalived.service
  print_message "${GREEN}" "Keepalived Started"
  sudo systemctl status keepalived --no-pager
}

# --- Script Execution Starts Here ---

# Install Keepalived (will uninstall previous version if found)
install_keepalived

# --- Dynamic Input Prompts ---
print_message "${YELLOW}" "--- Keepalived Configuration Inputs ---"
read -p "Enter current node's physical/WireGuard IP address (e.g., 192.168.0.51 or 10.0.0.1): " current_node_ip
read -p "Enter the network interface name (e.g., eth0, enp2s0, wg0): " interface_name
read -p "Enter the VRRP state for this node (MASTER or BACKUP): " vrrp_state
read -p "Enter this node's priority (e.g., 150 for MASTER, 100 for BACKUP, 50 for another BACKUP): " current_node_priority
read -p "Enter the Virtual IP (VIP) with CIDR (e.g., 192.168.0.50/24 or 10.10.10.10/24): " virtual_ip_with_cidr
read -p "Enter comma-separated IP addresses of OTHER Keepalived peers (e.g., 192.168.0.52,192.168.0.53): " peer_ips_comma_separated
read -s -p "Enter the VRRP authentication password (e.g., MyAuthPass): " auth_pass # -s for silent input
echo "" # New line after silent input

# Format peer IPs for the config file (one per line, indented)
peer_ips_formatted=$(echo "$peer_ips_comma_separated" | tr ',' '\n' | sed 's/^/        /')

# Define the Keepalived configuration
new_config=$(cat <<EOF
vrrp_instance VI_1 {
state $vrrp_state
interface $interface_name
virtual_router_id 55 # Unique ID for this VRRP instance (common across all nodes)
priority $current_node_priority
advert_int 1
unicast_src_ip $current_node_ip
unicast_peer {
$peer_ips_formatted
}

authentication {
    auth_type PASS
    auth_pass $auth_pass
}

virtual_ipaddress {
    $virtual_ip_with_cidr
}
}
EOF
)

# Define the path to the configuration file
config_file="/etc/keepalived/keepalived.conf"

# Remove existing config file
if [ -e "$config_file" ]; then
    print_message "${YELLOW}" "Removing existing $config_file..."
    sudo rm "$config_file"
fi

# Create new config file
print_message "${YELLOW}" "Creating new $config_file with the specified configuration..."
echo "$new_config" | sudo tee "$config_file" > /dev/null
print_message "${GREEN}" "Keepalived configuration created successfully."

# Start Keepalived service
start_keepalived

print_message "${GREEN}" "--- Keepalived setup complete. ---"
print_message "${YELLOW}" "Verify the VIP using: ip a | grep '$virtual_ip_with_cidr'"
print_message "${YELLOW}" "Check Keepalived logs: journalctl -xeu keepalived"

# Unset auth_pass for security
unset auth_pass
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /tmp/deploy_keepalived.sh
```

**Execute the script with `sudo`:**

```bash
sudo /tmp/deploy_keepalived.sh
```

**Follow the prompts carefully:**

* **Current node's physical/WireGuard IP address:** Enter the IP address of the Pi you are currently configuring.
* **Network interface name:** Enter the exact name of the network interface (e.g., `eth0`, `enp2s0`, `wg0`) that Keepalived will use for VRRP communication. This is crucial!
* **VRRP state for this node (MASTER or BACKUP):**
  * For the primary node, enter `MASTER`.
  * For other nodes, enter `BACKUP`.
* **This node's priority:** A number. The highest priority (e.g., `150`) will become MASTER. Other backups should have lower, unique priorities (e.g., `100`, `50`).
* **Virtual IP (VIP) with CIDR:** This is the floating IP your services will use (e.g., `192.168.0.50/24` or `10.0.0.100/24`). This must be the **same** on all nodes.
* **Comma-separated IP addresses of OTHER Keepalived peers:** Enter the IP addresses of all _other_ nodes in the VRRP group, separated by commas. Do _not_ include the current node's IP here.
* **VRRP authentication password:** Enter a strong password for VRRP communication. This must be the **same** on all nodes.

**Check VIP Assignment:** On the node you configured as `MASTER`, run:

```bash
ip a | grep "your_virtual_ip_address"
```

You should see the VIP assigned to the specified interface. On BACKUP nodes, you should _not_ initially see the VIP unless the MASTER fails.

#### **Cleaning Up the Script (Temporary File)**

Once your Keepalived setup is confirmed and working, you can remove the temporary script file.

```bash
rm /tmp/deploy_keepalived.sh
```

