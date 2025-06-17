---
icon: database
---

# Setup MariaDB

**Create the script file:**&#x20;

```
nano /tmp/install_mariadb.sh
```

**Paste the script content:** Copy the entire script you provided and paste it into the `nano` editor:

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

# Function to uninstall MariaDB Server
uninstall_mariadb() {
  if is_package_installed mariadb-server; then
    print_message "${RED}" "Stopping and Uninstalling MariaDB Server..."
    sudo systemctl stop mariadb # Ensure service is stopped
    sudo apt-get remove --purge -y mariadb-server mariadb-client mariadb-common # Include client/common for a cleaner removal
    sudo apt-get autoremove -y
    sudo apt-get clean
    print_message ${GREEN} "MariaDB Server uninstallation complete."
  else
    print_message ${GREEN} "MariaDB Server is not installed."
  fi
}

# Remove the existing galera.cnf file if it exists
# This is useful if running this script after a previous Galera setup
conf_file="/etc/mysql/conf.d/galera.cnf"
if [ -f "$conf_file" ]; then
  sudo rm "$conf_file"
  print_message ${YELLOW} "Removed existing $conf_file"
fi

# Function to install MariaDB Server
install_mariadb() {
  if is_package_installed mariadb-server; then
    print_message "${YELLOW}" "Existing MariaDB Server installation detected. Attempting to uninstall first..."
    uninstall_mariadb
    sleep 2 # Give some time for services to stop and files to be removed
  fi

  print_message "${YELLOW}" "Updating package lists..."
  sudo apt-get update

  print_message "${YELLOW}" "Installing MariaDB Server..."
  # Install specific packages for server and client tools
  sudo apt-get install -y mariadb-server mariadb-client
  print_message ${GREEN} "MariaDB Server installation complete."
}

# --- Script Execution Starts Here ---

# Call the installation function
install_mariadb

# Define the MariaDB configuration file path
CONFIG_FILE="/etc/mysql/mariadb.conf.d/50-server.cnf"

# Check if the config file exists before attempting to modify
if [ ! -f "$CONFIG_FILE" ]; then
  print_message "${RED}" "Error: MariaDB server configuration file ($CONFIG_FILE) not found."
  print_message "${RED}" "MariaDB might not have installed correctly. Aborting configuration changes."
  exit 1
fi

print_message "${YELLOW}" "Backing up original configuration file..."
sudo cp "$CONFIG_FILE" "${CONFIG_FILE}.bak"

print_message "${YELLOW}" "Changing bind-address to 0.0.0.0 in $CONFIG_FILE..."
# Using sed to find and replace the line containing bind-address
# Ensures it handles commented lines or different spacing
sudo sed -i -E 's/^(#\s*)?bind-address\s*=.*/bind-address = 0.0.0.0/' "$CONFIG_FILE"
# The -E enables extended regex, ^(#\s*)? makes it optionally match commented lines

print_message "${YELLOW}" "Restarting MariaDB to apply changes..."
sudo systemctl restart mariadb

# Check MariaDB service status
if sudo systemctl is-active --quiet mariadb; then
  print_message "${GREEN}" "MariaDB Server installed and configured successfully. It is now listening on 0.0.0.0."
  print_message "${GREEN}" "Remember to secure your MariaDB installation, e.g., using 'sudo mysql_secure_installation'."
else
  print_message "${RED}" "Failed to start MariaDB Server after configuration changes."
  print_message "${RED}" "Check logs for details: journalctl -xeu mariadb"
fi
```

#### **Make the Script Executable**

```bash
chmod +x /tmp/install_mariadb.sh
```

**Execute the script:**&#x42;ash

```bash
sudo /tmp/install_mariadb.sh
```

The script will print colored messages indicating its progress.

* It will first check for existing MariaDB installations and offer to uninstall them.
* It will remove any pre-existing `galera.cnf` file.
* It will update package lists and install MariaDB Server and Client.
* It will back up your `50-server.cnf` file.
* It will modify the `bind-address` to `0.0.0.0`.
* Finally, it will restart the MariaDB service and report its success or failure.

#### **Post-Installation & Security Steps**

After the script completes, MariaDB Server should be running and listening on all network interfaces.

1.  **Verify MariaDB Status:**

    Bash

    ```bash
    sudo systemctl status mariadb
    ```

    Look for `Active: active (running)`.
2.  **Verify Listening Port:**

    Bash

    ```bash
    sudo ss -tuln | grep 3306
    ```

    You should see an entry like `tcp LISTEN 0 80 0.0.0.0:3306 0.0.0.0:*`

**Remove the script:**&#x42;ash

```bash
rm /tmp/install_mariadb.sh
```

