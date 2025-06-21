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
    print_message "${RED}" "Stopping and Uninstalling Keepalived..."
    sudo systemctl stop keepalived || true # Use || true to prevent script exit if service not running
    sudo apt-get remove --purge keepalived -y
    sudo rm -rf /etc/keepalived
    print_message "${GREEN}" "Keepalived and its configuration have been uninstalled."
  else
    print_message "${YELLOW}" "Keepalived is not installed."
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
read -p "Enter current node's physical/WireGuard IP address (e.g., 10.21.22.15): " current_node_ip
echo ""
ip a # Show IP configuration for user reference
echo ""

read -p "Enter the network interface name (e.g., wg0, eth0): " interface_name
echo ""

# New: Role-based input
while true; do
  read -p "Enter the role for this node (Master, Manager, Observer): " node_role
  node_role=$(echo "$node_role" | tr '[:upper:]' '[:lower:]') # Convert to lowercase for consistency
  case "$node_role" in
    master|manager|observer)
      break
      ;;
    *)
      print_message "${RED}" "Invalid role. Please enter 'Master', 'Manager', or 'Observer'."
      ;;
  esac
done
echo ""

# Variables for configuration
vrrp_state=""
current_node_priority=""
additional_config="" # For nopreempt or track_script

if [ "$node_role" == "observer" ]; then
    print_message "${YELLOW}" "Observer node selected. This node will not participate in VIP election."
    print_message "${YELLOW}" "Only basic Keepalived setup for VRRP advertisements is needed."
    read -s -p "Enter the VRRP authentication password (e.g., MyAuthPass): " auth_pass # -s for silent input
    echo "" # New line after silent input
    # Observer nodes don't need a state or priority in the traditional sense,
    # and they don't hold the VIP. Their primary purpose is often just to see VRRP traffic
    # or potentially run custom scripts based on other node states (though not directly via vrrp_instance).
    # For simplicity in this script, an Observer will just be a basic BACKUP with low priority
    # that won't try to acquire the VIP. We'll set a very low priority and state BACKUP
    # without a virtual_ipaddress block.
    vrrp_state="BACKUP"
    current_node_priority="1" # Very low priority
    virtual_ip_with_cidr="" # Observers don't get the VIP
else
    # Master or Manager roles
    read -p "Enter the VIP WITH CIDR (e.g., 10.21.22.5/24): " virtual_ip_with_cidr
    echo ""

    read -s -p "Enter the VRRP authentication password (e.g., MyAuthPass): " auth_pass # -s for silent input
    echo "" # New line after silent input

    # Specific configurations based on Master/Manager
    if [ "$node_role" == "master" ]; then
        vrrp_state="MASTER"
        # Standard Master priority - adjust as needed, e.g., 200
        read -p "Enter Master node's priority (e.g., 200): " current_node_priority
        # Master needs a notify_master script to kick off failback
        # Make sure this path exists and the script is executable
        additional_config="    notify_master \"/etc/keepalived/scripts/notify_master.sh\"
            notify_backup \"/etc/keepalived/scripts/notify_backup.sh\"
            notify_fault \"/etc/keepalived/scripts/notify_fault.sh\""
    elif [ "$node_role" == "manager" ]; then
        vrrp_state="BACKUP"
        # Manager priority should be lower than Master, but higher than other backups if any
        read -p "Enter Manager node's priority (e.g., 150): " current_node_priority
        # Manager needs nopreempt to hold VIP during failover until Master tells it to release
        additional_config="    nopreempt
            notify_master \"/etc/keepalived/scripts/notify_master.sh\"
            notify_backup \"/etc/keepalived/scripts/notify_backup.sh\""
        # You might also want notify_backup and notify_fault for comprehensive logging/actions
        # additional_config+="\\n    notify_backup \"/etc/keepalived/scripts/log_keepalived_state.sh BACKUP\""
        # additional_config+="\\n    notify_fault \"/etc/keepalived/scripts/log_keepalived_state.sh FAULT\""
    fi
fi

# Get peer IPs for Master/Manager (Observers don't need to be in unicast_peer usually, as they don't participate in VIP election)
if [ "$node_role" == "master" ] || [ "$node_role" == "manager" ]; then
    read -p "Enter comma-separated IP addresses of OTHER Keepalived peers (e.g., 10.10.0.52,10.10.0.53): " peer_ips_comma_separated
    # Format peer IPs for the config file (one per line, indented)
    peer_ips_formatted=$(echo "$peer_ips_comma_separated" | tr ',' '\n' | sed 's/^/        /')
else
    # Observers typically don't unicast to other peers for VRRP for VIP.
    # They usually listen for multicast or are configured with no peers to just see advertisements.
    # For unicast setups, they might need the unicast_peer block if they want to monitor specific nodes.
    # For now, let's keep it simple for observers - no explicit unicast_peer unless needed.
    peer_ips_formatted=""
fi


# Define the Keepalived configuration
new_config=$(cat <<EOF
vrrp_instance VI_1 {
    state $vrrp_state
    interface $interface_name
    virtual_router_id 55 # Unique ID for this VRRP instance (common across all nodes)
    priority $current_node_priority
    advert_int 1
    unicast_src_ip $current_node_ip
    # Only include unicast_peer if there are peers defined
$(if [ -n "$peer_ips_formatted" ]; then echo "    unicast_peer {"; echo "$peer_ips_formatted"; echo "    }"; fi)

    authentication {
        auth_type PASS
        auth_pass $auth_pass
    }

    # Only include virtual_ipaddress for Master/Manager (nodes that handle the VIP)
$(if [ -n "$virtual_ip_with_cidr" ]; then echo "    virtual_ipaddress {"; echo "        $virtual_ip_with_cidr"; echo "    }"; fi)

$(if [ -n "$additional_config" ]; then echo "$additional_config"; fi)
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

#### To Ensure KeepAlived Service waits till wg0 (WireGuard) interface is up and running before starting up:

**Create an Override for the Service:** **DO NOT directly edit the file in `/lib/systemd/system/`** as it might be overwritten by package updates. Instead, create an override directory:

```bash
sudo mkdir -p /etc/systemd/system/keepalived.service.d/
sudo nano /etc/systemd/system/keepalived.service.d/override.conf
```

**Add Dependencies:** In the `override.conf` file, add the following:

```ini
[Unit]
After=wg-quick@wg0.service
Requires=wg-quick@wg0.service
```

* **`After=wg-quick@wg0.service`**: This tells Systemd that `keepalived.service` should only _start after_ `wg-quick@wg0.service` has finished starting.
* **`Requires=wg-quick@wg0.service`**: This creates a stronger dependency. If `wg-quick@wg0.service` fails to start, `keepalived.service` will also fail to start. If `wg-quick@wg0.service` stops, `keepalived.service` will also be stopped.

**Reload Systemd and Restart Keepalived:** After making changes to Systemd unit files, you must reload the Systemd daemon:

```bash
sudo systemctl daemon-reload
sudo systemctl restart keepalived
```

**Verify:** Check the status of Keepalived to see if it's running and if its dependencies are met:

```bash
systemctl status keepalived
```

You should see something like: `Loaded: loaded (/lib/systemd/system/keepalived.service; ...; /etc/systemd/system/keepalived.service.d/override.conf)` And it should show "active (running)" only after `wg0` is up.

#### **Cleaning Up the Script (Temporary File)**

Once your Keepalived setup is confirmed and working, you can remove the temporary script file.

```bash
rm /tmp/deploy_keepalived.sh
```
