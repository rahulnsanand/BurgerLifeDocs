---
icon: rotate
---

# SyncThing Configuration

**Create the script file:**

```bash
nano /tmp/syncthing_setup.sh
```

**Paste the script content:** Copy the entire script provided in your prompt and paste it into the `nano` editor.

```bash
#!/bin/bash

# Define ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
RESET='\033[0m'

syncthing_user="root"

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

# Function to uninstall Syncthing
uninstall_syncthing() {
  if is_package_installed syncthing; then
    print_message "${RED}" "Uninstalling Syncthing..."
    sudo apt-get remove --purge -y syncthing
    sudo apt-get autoremove -y
    sudo apt-get clean

    print_message "${RED}" "....................................................."
    read -p "REMOVE ALL CONFIGURATIONS? (y/n): " choice
    print_message "${RED}" "....................................................."

    if [[ $choice == "y" ]]; then
        sudo rm -rf /home/$syncthing_user/.local/state/syncthing
    fi

    print_message "${GREEN}" "Syncthing uninstallation complete."
  else
    print_message "${GREEN}" "Syncthing is not installed."
  fi
}

# Function to install Syncthing
install_syncthing() {
  if is_package_installed syncthing; then
    sudo systemctl stop syncthing@$syncthing_user.service
    uninstall_syncthing
  fi

  print_message "${YELLOW}" "Installing curl..."
  sudo apt-get update
  sudo apt-get install -y curl

  print_message "${YELLOW}" "Adding Syncthing repository..."
  curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
  echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list > /dev/null

  print_message "${YELLOW}" "Updating package list..."
  sudo apt-get update

  print_message "${YELLOW}" "Installing Syncthing..."
  sudo apt-get install -y syncthing
  print_message "${GREEN}" "Syncthing installation complete."

  # Modify the Syncthing configuration for current user
  syncthing_config1="/root/.local/state/syncthing/config.xml" # Correct path for root user
  syncthing_config2="/root/.config/syncthing/config.xml" # Correct path for root user

  # Start Syncthing for the user to create config.xml if it doesn't exist
  print_message "${YELLOW}" "Starting Syncthing for user $syncthing_user to initialize configuration..."
  sudo systemctl enable syncthing@$syncthing_user.service
  sudo systemctl start syncthing@$syncthing_user.service
  sleep 15

  if [ -f "$syncthing_config1" ]; then
    # Use XMLStarlet or sed to change the address
    print_message "${YELLOW}" "Modifying Syncthing config for user $syncthing_user..."

    # Check if xmlstarlet is installed
    print_message "${YELLOW}" "Installing xmlstarlet for XML modifications..."
    sudo apt-get install -y xmlstarlet

    xmlstarlet ed --inplace \
      -u "/configuration/gui/address" \
      -v "0.0.0.0:8384" "$syncthing_config1"

    print_message "${GREEN}" "Configuration updated for user $syncthing_user."

    # Restart Syncthing
    sudo systemctl restart syncthing@$syncthing_user.service
  else
    if [ -f "$syncthing_config2" ]; then
      # Use XMLStarlet or sed to change the address
      print_message "${YELLOW}" "Modifying Syncthing config for user $syncthing_user..."
  
      # Check if xmlstarlet is installed
      print_message "${YELLOW}" "Installing xmlstarlet for XML modifications..."
      sudo apt-get install -y xmlstarlet
  
      xmlstarlet ed --inplace \
        -u "/configuration/gui/address" \
        -v "0.0.0.0:8384" "$syncthing_config2"
  
      print_message "${GREEN}" "Configuration updated for user $syncthing_user."
  
      # Restart Syncthing
      sudo systemctl restart syncthing@$syncthing_user.service
    else
      print_message "${RED}" "No configuration found for user $syncthing_user."
    fi
  fi
}

# Install Syncthing
install_syncthing
```

**Add execute permissions:**&#x42;ash

```bash
chmod +x /tmp/syncthing_setup.sh
```

**Execute the script:**&#x42;ash

```bash
sudo /tmp/syncthing_setup.sh
```

The script will print colored messages indicating its progress.

* It will first check for existing Syncthing installations and offer to uninstall them.
* It will install necessary dependencies (curl, xmlstarlet).
* It will add the Syncthing repository and install Syncthing.
* It will start Syncthing (as `root`) to generate the initial configuration.
* It will then modify the `config.xml` to set the GUI address to `0.0.0.0:8384`.
* Finally, it will restart the Syncthing service.

**ON MASTER NODE: Disable Syncthing from starting on boot:**

```bash
sudo systemctl stop syncthing@root.service
```

```bash
sudo systemctl disable syncthing@root.service
```

```bash
sudo systemctl start syncthing@root.service
```

**Open your web browser:** On your computer, open a web browser and navigate to:

```tsconfig
http://<your_node_address>:8384
```

* **Initial Setup:** The first time you access the UI, Syncthing might ask you to:
  * Set an admin username and password (highly recommended for security).
  * Opt-out of anonymous usage reporting.
  * You can now proceed to add folders and devices to synchronize.

#### **Configure Sync Protocol Listen Addresses (on ALL Syncthing Nodes)**

This tells your Syncthing instance to only bind its synchronization listener to the WireGuard interface.

1. **Go to Actions > Settings** (the gear icon on the top right, then "Settings").
2. Click on the **"Connections" tab**.
3. In the **"Sync Protocol Listen Addresses"** field, replace `default` with your WireGuard assigned IP address and port `22000` (Syncthing's default sync port).
   *   **Example:** If `node1`'s WireGuard IP is `10.0.0.1`, you would enter:

       ```tsconfig
       tcp://10.21.22.10:22000
       ```
   * **Crucially, uncheck the following options:**
     * **Enable NAT traversal:** This would allow it to try connecting via UPnP/NAT-PMP, which you want to avoid if you're forcing WireGuard.
     * **Global Discovery:** This allows Syncthing to find other devices via the public discovery servers. You want to disable this if you're only using private WireGuard IPs.
     * **Enable Relaying:** This allows Syncthing to use public relay servers if direct connections fail. Disable this for the same reason as Global Discovery.
     * **Local Discovery:** While this is often fine on a secure LAN, if your WireGuard network is strictly separate from your physical LAN and you want _only_ WireGuard traffic, you can uncheck this too. For most scenarios, leaving it checked is fine, as it will try to find devices on _any_ directly connected network, including your `wg0` interface's subnet. However, for absolute isolation, uncheck it.

#### Configure Endpoints for Each Peer (on ALL Syncthing Nodes)

This ensures that when your Syncthing instance tries to connect to a peer, it _only_ tries to connect via that peer's WireGuard IP.

1. **Add all your Syncthing nodes as devices to each other.** (Go to "Devices" tab -> "Add Remote Device" -> paste device ID).
   * **Important:** If you haven't already, ensure all 3 nodes have Syncthing running and have exchanged Device IDs. Each node needs to add the other two.
2. **Edit Each Remote Device:**
   * On `node1`'s GUI, click on the **"Devices"** tab.
   * Locate the entry for `node2` and click the **"Edit"** button (pencil icon).
   * Go to the **"Advanced" tab** in the "Edit Device" window.
   * In the **"Addresses"** field, replace `dynamic` with the WireGuard IP address of `node2` and Syncthing's sync port `22000`.
     *   **Example (on `node1`'s GUI, for `node2`):**

         ```tsconfig
         tcp://10.0.0.2:22000
         ```
     *   **Example (on `node1`'s GUI, for `node3`):**

         ```tsconfig
         tcp://10.0.0.3:22000
         ```
   * **Click "Save".**
3. Repeat this for every peer on every node.

#### How do I increase the inotify limit to get my filesystem watcher to work?

Linux typically restricts the number of watches per user (usually 8192). If you have many directories, you will need to adjust that number.

On many Linux distributions you can run the following to fix it:

```bash
echo "fs.inotify.max_user_watches=204800" | sudo tee -a /etc/sysctl.conf
```

On Arch Linux and potentially others it is preferred to write this line into a separate file, i.e. you should run:

```bash
echo "fs.inotify.max_user_watches=204800" | sudo tee -a /etc/sysctl.d/90-override.conf
```

This only takes effect after a reboot. To adjust the limit immediately, run:

```bash
echo 204800 | sudo tee /proc/sys/fs/inotify/max_user_watches
```

#### **Managing the Syncthing Service**

After the script runs, Syncthing will be installed and configured to run as a `systemd` service for the `root` user.

*   **Check Syncthing Status:**

    ```bash
    sudo systemctl status syncthing@root.service
    ```
*   **Stop Syncthing:**

    ```bash
    sudo systemctl stop syncthing@root.service
    ```
*   **Start Syncthing:**

    ```bash
    sudo systemctl start syncthing@root.service
    ```
*   **Restart Syncthing (after configuration changes):**

    ```bash
    sudo systemctl restart syncthing@root.service
    ```
*   **Disable Syncthing from starting on boot:**

    ```bash
    sudo systemctl disable syncthing@root.service
    ```

**Remove the script:**&#x42;ash

```bash
rm /tmp/syncthing_setup.sh
```
