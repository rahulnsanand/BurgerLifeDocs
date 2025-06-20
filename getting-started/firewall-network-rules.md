---
description: Below are all the rules needed to make WireGuard VPN work
icon: fire
---

# Firewall/Network Rules

**Create the script file:** Use `nano` to create a new file named `docker_setup.sh` in the `/tmp` directory:

```bash
nano /tmp/ufw_setup.sh
```

**Paste the script content:** Copy your provided script content and paste it into the `nano` editor.

```bash
#!/bin/bash

# Script: Automated UFW Configuration for Raspbian/Debian Systems
# Author: Your Name/Handle (optional)
# Date: June 15, 2025
# Description: This script automates the installation, uninstallation, and
#              configuration of UFW (Uncomplicated Firewall) on Raspbian
#              and Debian-based systems. It sets up default rules, allows
#              specified common ports globally, and allows additional
#              application and cluster ports from defined private subnets.
#              It's designed for servers in a local network, including
#              those part of a Keepalived/GlusterFS cluster.

# --- Configuration Variables ---
UFW_LOG_FILE="/var/log/ufw_script.log" # Log file for script output
PRIVATE_SUBNETS=("192.168.0.0/16" "10.0.0.0/8" "172.16.0.0/12") # Private network ranges for restricted access

# Define ports to be allowed from ANY IP (public-facing services)
declare -A PUBLIC_OPEN_PORTS=(
    [22]="Allow SSH/SFTP (Consider restricting to specific IPs if possible)"
    [80]="Allow HTTP (Nginx/Web Traffic)"
    [443]="Allow HTTPS (Nginx/SSL Traffic)"
    [51820]="Allow VPN (WireGuard)"
    [51821]="Allow VPN Dashboard"
)

# Define ports to be allowed ONLY from specified PRIVATE_SUBNETS
declare -A PRIVATE_NETWORK_PORTS=(
    # --- General & Management ---
    [800]="Allow HTTP Nginx Traffic (Specific instances, e.g., NPM admin)"
    [9000]="Allow Portainer Docker Container Web UI"
    [9443]="Allow Portainer API"
    [61208]="Allow Glances Monitoring HTTP"
    [61209]="Allow Glances Monitoring HTTPS" # If Glances serves HTTPS

    # --- Database Services ---
    [3306]="Allow MariaDB Server Default"
    [4567]="Allow MariaDB Galera Cluster (Default SYNC)"
    [4444]="Allow MariaDB Galera Cluster (Default IST)"
    [4568]="Allow MariaDB Galera Cluster (Default SST)"
    [5432]="Allow PostgreSQL Server"    
    [8008]="Patroni REST API"
    [2379]="etcd default client/peer ports"
    [2280]="etcd default client/peer ports"

    # --- Keepalived/VRRP ---
    [5400]="Allow Keepalived (Unicast)" # For non-multicast VRRP
    [112]="Allow Keepalived (VRRP - IP Protocol)" # This is an IP Protocol, not a TCP/UDP port
    [113]="Allow Keepalived (VRRP - IP Protocol)" # Not a standard port, often confused with auth service

    # --- File Sharing & IoT ---
    [139]="Allow Samba Share (SMB/CIFS)"
    [445]="Allow Samba Share (SMB/CIFS)"
    [67]="Allow DHCP Server/Client (UDP)" # Only if this machine is a DHCP server/client needing inbound DHCP

    # --- AdGuard Home / DNS ---
    [3000]="Allow AdGuard Home Web UI"
    [53]="Allow AdGuard Home/DNS (TCP/UDP for DNS queries)"

    # --- Media / Home Lab Apps ---
    [32400]="Allow Plex Media Server (Main Port)"
    [8989]="Allow Sonarr/Hostapd/Wifi Management (e.g., RaspAP Web UI)"
    [7878]="Allow Radarr"
    [9696]="Allow Prowlarr"
    [8686]="Allow Lidarr"
    [8787]="Allow Readarr"
    [6767]="Allow Bazarr"
    [5055]="Allow Overseerr"
    [5454]="Allow Notifiarr"
    [8096]="Allow Jellyfin HTTP"
    [8920]="Allow Jellyfin HTTPS"
    [7359]="Allow Jellyfin Local Network Discovery"

    # --- Cloud/Productivity Suite (Nextcloud, Authentik, etc.) ---
    [808]="Allow Nextcloud AIO Configurator (Commonly used during initial setup)"
    [3478]="Allow Nextcloud Talk (TURN server related)"
    [11000]="Allow Nextcloud (Typical internal port if proxied)"
    [11008]="Allow Authentik HTTP (Internal)"
    [11009]="Allow Authentik HTTPS (Internal)"
    [11002]="Allow Bookstack"
    [11003]="Allow Vaultwarden (HTTP)"
    [11004]="Allow Monica"
    [11006]="Allow n8n"
    [11007]="Allow Uptime Kuma"
    [11013]="Allow Vaultwarden (Websocket)" # If different from main HTTP/HTTPS
    [11014]="Allow Vaultwarden (Attachments)" # If different
    [11015]="Allow Homepage (Dashboard)"
    [11016]="Allow Speedtest Tracker"
    [11017]="Allow Remotely (Remote Desktop)"
    [11019]="Allow AudioBook Shelf"
    [11020]="Allow Mealie"
    [11021]="Allow InfluxDB"
    [11022]="Allow Grafana"
    [11023]="Allow Firefox (Perhaps a specific containerized instance?)"
    [11024]="Allow Immich"
    [11025]="Allow VPN Speedtest (Internal tool?)"
    [11026]="Allow Firefox (Another instance?)"
    [11027]="Allow File Browser"
    [11028]="Allow OnlyOffice Document Server (HTTP)"
    [11029]="Allow OnlyOffice Document Server (HTTPS)"
    [11030]="Allow Bazarr (Another instance?)"
    [11031]="Allow Rahul's Portfolio (Internal Web Server)"
    [6246]="Allow Maintainerr"
    [6719]="Allow BurgerLife API"
    [5000]="Allow BurgerLife Webhook"

    # --- VPN Client-Specific (e.g., Gluetun) ---
    [8000]="Allow Gluetun VPN Client API"
    [8888]="Allow Gluetun VPN Client HTTP Proxy"
    [8889]="Allow Gluetun VPN Client TCP Proxy"

    # --- Plex Network Discovery & Companion ---
    [1900]="Allow Plex/Jellyfin DLNA Server (UDP)" # Both Plex and Jellyfin use this
    [5353]="Allow Plex Bonjour/Avahi network discovery (UDP)"
    [8324]="Allow Plex for Roku via Plex Companion (TCP)"
    [32410]="Allow Plex GDM network discovery (UDP)"
    [32412]="Allow Plex GDM network discovery (UDP)"
    [32413]="Allow Plex GDM network discovery (UDP)"
    [32414]="Allow Plex GDM network discovery (UDP)"
    [32469]="Allow Plex DLNA Server (UDP)" # Another DLNA port

    # --- Syncthing ---
    [22000]="Allow Syncthing Sync Protocol (TCP/UDP)"
    [21027]="Allow Syncthing Local Discovery (UDP)"
    [8384]="Allow Syncthing Web GUI (HTTP)"
)

# GlusterFS Specific Ports (Allowed ONLY from PRIVATE_SUBNETS)
GLUSTER_PORTS=(
    24007 # Gluster Daemon/Management
    24008 # Gluster Daemon/Management
)
GLUSTER_TCP_PORT_RANGES=(
    "24009:24024" # Gluster Brick/RDMA TCP
    "38465:38467" # Gluster Brick/RDMA TCP
)
GLUSTER_UDP_PORT_RANGES=(
    "24009:24024" # Gluster Brick/RDMA UDP
    "38465:38467" # Gluster Brick/RDMA UDP
)

# --- ANSI Color Codes ---
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
MAGENTA='\033[0;35m'
NC='\033[0m' # No Color (RESET)

# --- Logging and Messaging Functions ---

# Function to print messages with color and log to file
log_and_print() {
    local color="$1"
    local message="$2"
    echo -e "${color}[$(date +'%Y-%m-%d %H:%M:%S')] ${message}${NC}" | tee -a "$UFW_LOG_FILE"
}

# Function to print header sections
print_header() {
    local title="$1"
    log_and_print "${BLUE}" "\n--- ${title} ---"
}

# --- Core Functions ---

# Check for sudo permissions at the start
check_sudo() {
    if [[ "$EUID" -ne 0 ]]; then
        log_and_print "${RED}" "ERROR: This script must be run with sudo or as root."
        exit 1
    fi
}

# Function to check if a package is installed
is_package_installed() {
    local package_name="$1"
    dpkg-query -l "$package_name" &> /dev/null
    return $? # Returns 0 if installed, non-zero otherwise
}

# Function to install UFW
install_ufw() {
    print_header "UFW Installation"
    if is_package_installed ufw; then
        log_and_print "${YELLOW}" "UFW is already installed. Attempting uninstallation and reinstallation..."
        uninstall_ufw_silent # Uninstall silently if already present
    fi

    log_and_print "${YELLOW}" "Installing UFW package..."
    if sudo apt-get update && sudo apt-get install -y ufw; then
        log_and_print "${GREEN}" "UFW installation complete."
    else
        log_and_print "${RED}" "ERROR: UFW installation failed. Aborting."
        exit 1
    fi
}

# Function to uninstall UFW (verbose)
uninstall_ufw() {
    print_header "UFW Uninstallation"
    if ! is_package_installed ufw; then
        log_and_print "${YELLOW}" "UFW is not installed. Nothing to uninstall."
        return 0
    fi

    log_and_print "${YELLOW}" "Disabling UFW before uninstalling..."
    sudo ufw disable &>> "$UFW_LOG_FILE" || true # Disable, ignore error if already disabled
    log_and_print "${YELLOW}" "Uninstalling UFW package..."
    if sudo apt-get remove --purge -y ufw &>> "$UFW_LOG_FILE"; then
        log_and_print "${GREEN}" "UFW package uninstalled successfully."
        log_and_print "${YELLOW}" "Cleaning up unused packages and cache..."
        sudo apt-get autoremove -y &>> "$UFW_LOG_FILE"
        sudo apt-get clean &>> "$UFW_LOG_FILE"
        log_and_print "${GREEN}" "UFW uninstallation complete."
    else
        log_and_print "${RED}" "ERROR: UFW uninstallation failed."
        exit 1
    fi
}

# Function to uninstall UFW (silent, used internally)
uninstall_ufw_silent() {
    if is_package_installed ufw; then
        sudo ufw disable &>> "$UFW_LOG_FILE" || true
        sudo apt-get remove --purge -y ufw &>> "$UFW_LOG_FILE"
        sudo apt-get autoremove -y &>> "$UFW_LOG_FILE"
        sudo apt-get clean &>> "$UFW_LOG_FILE"
    fi
}

# Function to configure UFW rules
configure_ufw() {
    print_header "UFW Configuration"
    log_and_print "${YELLOW}" "Disabling UFW and resetting rules before configuration..."
    sudo ufw disable &>> "$UFW_LOG_FILE" || true # Ignore error if already disabled
    sudo ufw reset --force &>> "$UFW_LOG_FILE" # Use --force for non-interactive reset
    log_and_print "${GREEN}" "UFW rules reset."

    log_and_print "${YELLOW}" "Setting default policies: deny incoming, allow outgoing."
    sudo ufw default deny incoming &>> "$UFW_LOG_FILE"
    sudo ufw default allow outgoing &>> "$UFW_LOG_FILE"
    log_and_print "${GREEN}" "Default policies set."

    print_header "Adding Public-Facing Port Rules (from any IP)"
    for port in "${!PUBLIC_OPEN_PORTS[@]}"; do
        if [[ "$port" =~ ^[0-9]+$ ]]; then # Basic numeric port validation
            log_and_print "${CYAN}" "Allowing public TCP/UDP port ${port} (${PUBLIC_OPEN_PORTS[$port]})..."
            sudo ufw allow "$port" comment "${PUBLIC_OPEN_PORTS[$port]}" &>> "$UFW_LOG_FILE"
        elif [[ "$port" == "112" || "$port" == "113" ]]; then # Handle IP protocols for Keepalived
             log_and_print "${CYAN}" "Allowing public IP protocol ${port} (${PUBLIC_OPEN_PORTS[$port]})..."
             # UFW doesn't directly allow IP protocols by number in the same way as TCP/UDP ports.
             # You might need specific 'ufw route' rules or iptables for true IP protocol allowance.
             # For VRRP, typically it's handled by ensuring interfaces are up and not blocked from each other.
             # This script will attempt to add a rule, but be aware of its limitations for IP protocols.
             sudo ufw allow proto "$port" from any to any comment "${PUBLIC_OPEN_PORTS[$port]}" &>> "$UFW_LOG_FILE"
        else
            log_and_print "${YELLOW}" "Skipping invalid public port definition: ${port}"
        fi
    done
    log_and_print "${GREEN}" "Public-facing port rules added."

    print_header "Adding Private Network Port Rules (from defined subnets)"
    for subnet in "${PRIVATE_SUBNETS[@]}"; do
        log_and_print "${YELLOW}" "Configuring rules for subnet: ${subnet}"
        for port in "${!PRIVATE_NETWORK_PORTS[@]}"; do
             if [[ "$port" =~ ^[0-9]+$ ]]; then # Basic numeric port validation
                log_and_print "${CYAN}" "Allowing TCP/UDP port ${port} from ${subnet} (${PRIVATE_NETWORK_PORTS[$port]})..."
                sudo ufw allow from "$subnet" to any port "$port" comment "${PRIVATE_NETWORK_PORTS[$port]}" &>> "$UFW_LOG_FILE"
            elif [[ "$port" == "112" || "$port" == "113" ]]; then
                 log_and_print "${CYAN}" "Allowing IP protocol ${port} from ${subnet} (${PRIVATE_NETWORK_PORTS[$port]})..."
                 sudo ufw allow proto "$port" from "$subnet" to any comment "${PRIVATE_NETWORK_PORTS[$port]}" &>> "$UFW_LOG_FILE"
            else
                log_and_print "${YELLOW}" "Skipping invalid private port definition: ${port}"
            fi
        done
        log_and_print "${GREEN}" "Rules for subnet ${subnet} applied."
    done
    log_and_print "${GREEN}" "Private network port rules added."

    print_header "Adding GlusterFS Specific Rules (from defined subnets)"
    for subnet in "${PRIVATE_SUBNETS[@]}"; do
        log_and_print "${YELLOW}" "Configuring GlusterFS rules for subnet: ${subnet}"
        for port in "${GLUSTER_PORTS[@]}"; do
            log_and_print "${CYAN}" "Allowing GlusterFS port ${port} from ${subnet} (TCP/UDP)..."
            sudo ufw allow from "$subnet" to any port "$port" comment "Allow Gluster Daemon/Management" &>> "$UFW_LOG_FILE"
        done
        for port_range in "${GLUSTER_TCP_PORT_RANGES[@]}"; do
            log_and_print "${CYAN}" "Allowing GlusterFS TCP port range ${port_range} from ${subnet}..."
            sudo ufw allow from "$subnet" to any proto tcp port "$port_range" comment "Allow Gluster Brick/RDMA TCP" &>> "$UFW_LOG_FILE"
        done
        for port_range in "${GLUSTER_UDP_PORT_RANGES[@]}"; do
            log_and_print "${CYAN}" "Allowing GlusterFS UDP port range ${port_range} from ${subnet}..."
            sudo ufw allow from "$subnet" to any proto udp port "$port_range" comment "Allow Gluster Brick/RDMA UDP" &>> "$UFW_LOG_FILE"
        done
        log_and_print "${GREEN}" "GlusterFS rules for subnet ${subnet} applied."
    done
    log_and_print "${GREEN}" "GlusterFS specific rules added."

    log_and_print "${YELLOW}" "Enabling UFW..."
    if sudo ufw --force enable &>> "$UFW_LOG_FILE"; then
        log_and_print "${GREEN}" "UFW enabled successfully! Current UFW status:"
        sudo ufw status verbose | tee -a "$UFW_LOG_FILE" # Display status and log it
    else
        log_and_print "${RED}" "ERROR: Failed to enable UFW."
        exit 1
    fi
    log_and_print "${RED}" "If you were using SFTP, you might need to reconnect now as firewall rules have changed."
}

# Function to show current UFW status
show_ufw_status() {
    print_header "UFW Current Status"
    if ! is_package_installed ufw; then
        log_and_print "${YELLOW}" "UFW is not installed."
        return
    fi
    sudo ufw status verbose | tee -a "$UFW_LOG_FILE"
    if [[ $? -ne 0 ]]; then
        log_and_print "${RED}" "Failed to retrieve UFW status. Ensure UFW is installed and enabled."
    fi
}

# --- Main Script Logic ---

# Ensure script is run with sudo
check_sudo

# Set trap for cleanup on script exit (success or failure)
trap 'log_and_print "${BLUE}" "Script finished. Check $UFW_LOG_FILE for details." EXIT'

print_header "Starting UFW Configuration Script"
log_and_print "${CYAN}" "This script will install/reinstall UFW and configure firewall rules."

# --- User Options ---
echo -e "\n${YELLOW}Choose an action:${NC}"
echo -e "  ${GREEN}1) Install and Configure UFW${NC}"
echo -e "  ${RED}2) Uninstall UFW${NC}"
echo -e "  ${BLUE}3) Show UFW Status${NC}"
echo -e "  ${MAGENTA}4) Exit${NC}"
echo -n "${YELLOW}Enter your choice (1-4):${NC}"
read -r CHOICE

case "$CHOICE" in
    1)
        install_ufw
        configure_ufw
        ;;
    2)
        uninstall_ufw
        ;;
    3)
        show_ufw_status
        ;;
    4)
        log_and_print "${GREEN}" "Exiting script."
        exit 0
        ;;
    *)
        log_and_print "${RED}" "Invalid choice. Exiting."
        exit 1
        ;;
esac

print_header "UFW Script Execution Complete"
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Add execute permissions:**&#x42;ash

```bash
chmod +x /tmp/ufw_setup.sh
```

**Execute the script with `sudo`:**

```bash
sudo /tmp/ufw_setup.sh
```

#### **Cleaning Up the Script (Temporary File)**

Once your Docker and Docker Compose are set up as desired, the `ufw_setup.sh` file in `/tmp` is no longer needed.

```bash
rm /tmp/ufw_setup.sh
```
