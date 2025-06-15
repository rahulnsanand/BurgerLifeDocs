---
icon: docker
---

# Docker Setup

**Create the script file:** Use `nano` to create a new file named `docker_setup.sh` in the `/tmp` directory:

```bash
nano /tmp/docker_setup.sh
```

**Paste the script content:** Copy your provided script content and paste it into the `nano` editor.

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

# Function to install Docker
install_docker() {
  # No need to check if docker-ce is installed directly, the curl script handles it.
  # If you want to uninstall before reinstall, call uninstall_docker() explicitly here.

  print_message "${YELLOW}" "Installing Docker Engine..."
  # The official convenience script handles prerequisites and adds docker-ce, docker-ce-cli, containerd.io
  curl -fsSL https://get.docker.com | sh || { print_message "${RED}" "Docker Engine installation failed!"; exit 1; }

  # Add all members of the 'users' group to the 'docker' group
  print_message ${YELLOW} "Adding all members of the 'users' group to the 'docker' group..."
  for user in $(getent group users | cut -d: -f4 | tr ',' ' '); do
    sudo usermod -aG docker "$user"
  done
  print_message ${GREEN} "All members of the 'users' group added to the 'docker' group."

  print_message ${GREEN} "Docker Engine installation complete."
}

# Function to install Docker Compose
install_docker_compose() {
  # No need to check if docker-compose is installed directly, the curl command will overwrite.
  # If you want to uninstall before reinstall, call uninstall_docker_compose() explicitly here.

  print_message "${YELLOW}" "Installing Docker Compose v2.20.2 from GitHub..."
  # Use sudo curl to ensure proper permissions in /usr/local/bin
  sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose || { print_message "${RED}" "Docker Compose download failed!"; exit 1; }

  print_message ${YELLOW} "Making Docker Compose executable..."
  sudo chmod +x /usr/local/bin/docker-compose || { print_message "${RED}" "Failed to make Docker Compose executable!"; exit 1; }

  print_message ${YELLOW} "Verifying Docker Compose Installation..."
  docker-compose --version || { print_message "${RED}" "Docker Compose verification failed!"; exit 1; }
  print_message ${GREEN} "Docker Compose installation complete."
}

# Function to uninstall Docker Compose
uninstall_docker_compose() {
  print_message "${RED}" "Attempting to uninstall Docker Compose..."
  # Try to remove the curled version first
  if [ -f "/usr/local/bin/docker-compose" ]; then
    sudo rm "/usr/local/bin/docker-compose"
    print_message ${GREEN} "Removed /usr/local/bin/docker-compose."
  fi
  # Then try to remove apt-installed version if it exists
  if is_package_installed docker-compose; then
    sudo apt-get remove --purge -y docker-compose
    print_message ${GREEN} "Removed apt-installed docker-compose."
  fi
  sudo apt-get autoremove -y
  sudo apt-get clean
  print_message ${GREEN} "Docker Compose uninstallation complete."
}

# Function to uninstall Docker Engine
uninstall_docker() {
  print_message "${RED}" "Uninstalling Docker Engine..."
  # Unhold packages first before removing
  sudo apt-mark unhold docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin &>/dev/null
  sudo apt-get remove --purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin || true # Use || true to prevent script exit on non-existent packages
  # Remove Docker group memberships (optional, but good for a clean uninstall)
  for user in $(getent group docker | cut -d: -f4 | tr ',' ' '); do
    sudo deluser "$user" docker &>/dev/null
  done
  sudo rm -rf /var/lib/docker # Remove Docker data (CAUTION!)
  sudo rm -rf /etc/docker # Remove Docker config
  sudo apt-get autoremove -y
  sudo apt-get clean
  print_message ${GREEN} "Docker Engine uninstallation complete."
}

# --- Main Script Logic ---

# Ask if the user wants to install or uninstall Docker
read -p "$(print_message ${YELLOW} 'Do you want to: (a) Install Docker Engine only, (b) Install Docker Compose only, (c) Install Both, (u) Uninstall Docker & Compose, (s) Skip setup? (a/b/c/u/s): ')" choice

# Convert choice to lowercase for case-insensitivity
choice=$(echo "$choice" | tr '[:upper:]' '[:lower:]')

case "$choice" in
  a)
    install_docker
    sudo apt-mark hold docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ;;
  b)
    install_docker_compose
    # If only compose is installed, no need to hold engine packages, but holding compose plugin is still valid
    sudo apt-mark hold docker-compose-plugin &>/dev/null # This hold applies to the plugin version if it were installed via apt
    ;;
  c)
    install_docker
    install_docker_compose
    sudo apt-mark hold docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ;;
  u)
    uninstall_docker_compose # Uninstall compose first
    uninstall_docker       # Then uninstall engine
    ;;
  s)
    print_message ${GREEN} "Docker setup skipped."
    ;;
  *)
    print_message ${RED} "Invalid choice. Docker setup skipped."
    ;;
esac

print_message "${YELLOW}" "Please remember to log out and log back in for Docker group changes to take effect if you installed Docker Engine."
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Add execute permissions:**&#x42;ash

```bash
chmod +x /tmp/docker_setup.sh
```

**Execute the script with `sudo`:**

```bash
sudo /tmp/docker_setup.sh
```

**Follow the prompts:** The script will ask you: `Do you want to: (a) Install Docker Engine only, (b) Install Docker Compose only, (c) Install Both, (u) Uninstall Docker & Compose, (s) Skip setup? (a/b/c/u/s):` Enter your desired choice (`a`, `b`, `c`, `u`, or `s`) and press `Enter`.

**Observe the output:** The script will print messages in different colors, indicating its progress (installation steps, package removals, etc.).

**Log out and log back in:** This is crucial for your user to be able to run Docker commands without `sudo`.

**Verify Docker Engine (if installed):**

```bash
docker --version
```

Test with a simple container (should run without `sudo`):

```bash
docker run hello-world
```

**Execute the `apt-mark hold` command for each Docker package:**

```bash
sudo apt-mark hold docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Explanation of packages:**

* `docker-ce`: The Docker Community Edition engine itself.
* `docker-ce-cli`: The Docker command-line client.
* `containerd.io`: The container runtime that Docker uses.
* `docker-buildx-plugin`: A plugin for extended build capabilities (part of newer Docker installations).
* `docker-compose-plugin`: The new Docker Compose V2 plugin (if installed via `apt`). If you installed Docker Compose by curling it to `/usr/local/bin/docker-compose`, this specific package might not be installed, but it's good to include it just in case or for future consistency.

#### **How to Unhold Packages (when you want to update them later)**

When you decide to manually update Docker (e.g., to a newer major version or a security patch), you will first need to "unhold" them.

To unhold:

Bash

```bash
sudo apt-mark unhold docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

After unholding, you can then perform your `sudo apt update && sudo apt upgrade` to get the latest versions. Remember to `apt-mark hold` them again if you want to prevent future automatic updates.

#### **Cleaning Up the Script (Temporary File)**

Once your Docker and Docker Compose are set up as desired, the `docker_setup.sh` file in `/tmp` is no longer needed.

```bash
rm /tmp/docker_setup.sh
```
