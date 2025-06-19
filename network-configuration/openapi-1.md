---
icon: network-wired
---

# Adguard Configuration

**Create the `deploy_adguard.sh` file:** Use `nano` to create a new file:

```bash
nano /tmp/deploy_adguard.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

```bash
#!/bin/bash

# Define ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
RESET='\033[0m'

# --- Functions ---

# Function to print colored messages
print_message() {
  local color="$1"
  local message="$2"
  echo -e "${color}${message}${RESET}"
}

# Function to check if AdGuard Home is already installed
is_adguard_installed() {
  if systemctl is-active --quiet AdGuardHome; then
    return 0 # True, AdGuard Home service is active
  elif [ -f "/opt/AdGuardHome/AdGuardHome" ]; then
    return 0 # True, binary exists
  else
    return 1 # False
  fi
}

# Function to uninstall AdGuard Home
uninstall_adguard() {
  if is_adguard_installed; then
    print_message ${YELLOW} "AdGuard Home detected. Attempting to uninstall..."
    if sudo /opt/AdGuardHome/AdGuardHome -s uninstall; then
      print_message ${GREEN} "AdGuard Home service uninstalled successfully."
    else
      print_message ${YELLOW} "Could not uninstall AdGuard Home service. It might not have been installed as a service, or there was an issue."
    fi

    if sudo rm -rf /opt/AdGuardHome; then
      print_message ${GREEN} "Removed /opt/AdGuardHome directory."
    else
      print_message ${RED} "Failed to remove /opt/AdGuardHome. Manual intervention might be required."
      return 1 # Indicate failure
    fi
  else
    print_message ${YELLOW} "AdGuard Home not found. Nothing to uninstall."
  fi
  return 0 # Indicate success
}

# Function to determine the correct AdGuard Home architecture
get_architecture() {
  case "$(dpkg --print-architecture)" in
    amd64)
      echo "linux_amd64"
      ;;
    arm64|aarch64)
      echo "linux_arm64"
      ;;
    armhf|arm)
      echo "linux_armv7"
      ;;
    *)
      print_message ${RED} "Unsupported architecture: $(dpkg --print-architecture). Please check AdGuard Home documentation for compatible architectures."
      exit 1
      ;;
  esac
}

# --- Main Script Logic ---

print_message ${GREEN} "Starting AdGuard Home installation script..."
print_message ${GREEN} "This script will install or reinstall AdGuard Home on your Debian server."
echo ""

# Uninstall any previous installation
uninstall_adguard || exit 1 # Exit if uninstall fails

echo ""
print_message ${YELLOW} "Ensuring necessary directories exist..."
sudo mkdir -p /opt/AdGuardHome /opt/Downloads || { print_message ${RED} "Failed to create directories. Exiting."; exit 1; }

ARCH=$(get_architecture)
DOWNLOAD_URL="https://static.adtidy.org/adguardhome/release/AdGuardHome_${ARCH}.tar.gz"
DOWNLOAD_PATH="/opt/Downloads/AdGuardHome_${ARCH}.tar.gz"

print_message ${YELLOW} "Downloading AdGuard Home for ${ARCH} from ${DOWNLOAD_URL}..."
if ! sudo wget "${DOWNLOAD_URL}" -O "${DOWNLOAD_PATH}"; then
  print_message ${RED} "Failed to download AdGuard Home. Please check your internet connection or the download URL."
  exit 1
fi

print_message ${YELLOW} "Extracting AdGuard Home to /opt/AdGuardHome..."
if ! sudo tar xvf "${DOWNLOAD_PATH}" -C /opt/; then
  print_message ${RED} "Failed to extract AdGuard Home. The downloaded file might be corrupt or incomplete."
  sudo rm -f "${DOWNLOAD_PATH}" # Clean up incomplete download
  exit 1
fi

print_message ${YELLOW} "Cleaning up downloaded archive..."
sudo rm -f "${DOWNLOAD_PATH}" || print_message ${YELLOW} "Could not remove downloaded archive: ${DOWNLOAD_PATH}. Manual cleanup might be needed."

print_message ${YELLOW} "Installing AdGuard Home as a system service..."
if ! sudo /opt/AdGuardHome/AdGuardHome -s install; then
  print_message ${RED} "Failed to install AdGuard Home as a service. Please check the logs for errors."
  print_message ${YELLOW} "AdGuard Home might still be present in /opt/AdGuardHome, but not as a service."
  exit 1
fi

print_message ${GREEN} "AdGuard Home is now installed and running!"
print_message ${GREEN} "You can access the AdGuard Home web interface by navigating to http://YOUR_SERVER_IP:3000 in your browser."
print_message ${GREEN} "Remember to configure your DNS settings to use AdGuard Home."
echo ""
print_message ${GREEN} "Installation complete."

exit 0
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /tmp/deploy_adguard.sh
```

**Execute the script with `sudo`:**

```bash
sudo /tmp/deploy_adguard.sh
```

#### **Cleaning Up the Script (Temporary File)**

Once your Keepalived setup is confirmed and working, you can remove the temporary script file.

```bash
rm /tmp/deploy_adguard.sh
```
