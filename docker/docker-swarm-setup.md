---
icon: sitemap
---

# Docker Swarm Setup

**Create the script file:** Use `nano` to create a new file named `docker_setup.sh` in the `/tmp` directory:

```bash
nano /tmp/docker_swarm_setup.sh
```

**Paste the script content:** Copy your provided script content and paste it into the `nano` editor.

```bash
#!/bin/bash

# Define ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
BLUE='\033[0;34m'
RESET='\033[0m'

# Function to print colored messages
print_message() {
  local color="$1"
  local message="$2"
  echo -e "${color}${message}${RESET}"
}

# Function for input with prompt
get_input() {
  local prompt_color="$1"
  local prompt_text="$2"
  local var_name="$3"
  read -r -p "$(print_message "${prompt_color}" "${prompt_text}")" "$var_name"
}

# Function to check if a package is installed on Debian/Ubuntu-based systems
is_package_installed() {
  local package_name="$1"
  dpkg-query -W -f='${Status}' "$package_name" 2>/dev/null | grep -c "ok installed"
}

# Function to check if Docker service is running
is_docker_running() {
  systemctl is-active --quiet docker
}

# Function to get Docker Swarm status of the current node
get_swarm_status() {
  docker info --format '{{.Swarm.LocalNodeState}}' 2>/dev/null
}

# Function to validate an IP address format
validate_ip() {
  local ip="$1"
  if [[ "$ip" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
    IFS='.' read -r -a octets <<< "$ip"
    for octet in "${octets[@]}"; do
      [[ "$octet" -ge 0 && "$octet" -le 255 ]] || return 1
    done
    return 0
  else
    return 1
  fi
}

# Main menu function
main_menu() {
  while true; do
    clear # Clear screen for cleaner menu
    print_message "${BLUE}" "=== Docker Swarm Management Menu ==="
    print_message "${YELLOW}" "1) Initialize Docker Swarm (Create New From Scratch)"
    print_message "${YELLOW}" "2) Join Docker Swarm (Swarm Already Created - Has at least 1 Manager)"
    print_message "${YELLOW}" "3) Leave Docker Swarm (Leave current swarm, even if sole member)"
    print_message "${RED}" "0) Exit"
    print_message "${BLUE}" "===================================="

    get_input "${GREEN}" "Enter your choice: " choice

    case "$choice" in
      1) init_swarm ;;
      2) join_swarm ;;
      3) leave_swarm ;;
      0) print_message "${GREEN}" "Exiting script. Goodbye!"; exit 0 ;;
      *) print_message "${RED}" "Invalid choice. Please enter 0, 1, 2, or 3." ; sleep 2 ;;
    esac
    sleep 1 # Small delay before clearing for next menu loop
  done
}

# Option 1: Initialize Docker Swarm
init_swarm() {
  local swarm_status=$(get_swarm_status)
  if [ "$swarm_status" = "active" ] || [ "$swarm_status" = "pending" ]; then
    print_message "${YELLOW}" "This node is already part of a Swarm ($swarm_status). Cannot initialize a new one."
    print_message "${YELLOW}" "To reinitialize, you must first leave the existing Swarm (Option 3)."
    get_input "${BLUE}" "Press Enter to continue..." dummy # Wait for user
    return
  fi

  local manager_ip
  while true; do
    get_input "${YELLOW}" "Enter Current Node Internal IP Address (e.g., 192.168.1.10): " manager_ip
    if validate_ip "$manager_ip"; then
      break
    else
      print_message "${RED}" "Invalid IP address format. Please try again."
    fi
  done

  print_message "${YELLOW}" "Attempting to initialize a new Swarm using $manager_ip..."
  if docker swarm init --advertise-addr "$manager_ip"; then
    print_message "${GREEN}" "Docker Swarm initialized successfully!"
    print_message "${BLUE}" "--- Swarm Join Tokens ---"
    print_message "${YELLOW}" "To add a WORKER to this swarm, run the following command on the worker node:"
    docker swarm join-token worker # Show worker token
    print_message "${YELLOW}" "To add another MANAGER to this swarm, run the following command on the manager node:"
    docker_swarm_join_token_manager_output=$(docker swarm join-token manager)
    echo "$docker_swarm_join_token_manager_output"
    print_message "${BLUE}" "--------------------------"
    print_message "${YELLOW}" "Store these tokens securely. They are needed to add more nodes."
  else
    print_message "${RED}" "Failed to initialize Docker Swarm. Check Docker logs for details."
  fi
  get_input "${BLUE}" "Press Enter to continue..." dummy
}

# Option 2: Join Docker Swarm
join_swarm() {
  local swarm_status=$(get_swarm_status)
  if [ "$swarm_status" = "active" ] || [ "$swarm_status" = "pending" ]; then
    print_message "${YELLOW}" "This node is already part of a Swarm ($swarm_status). Cannot join a new one."
    print_message "${YELLOW}" "To join a different Swarm, you must first leave the existing Swarm (Option 3)."
    get_input "${BLUE}" "Press Enter to continue..." dummy
    return
  fi

  local manager_ip
  while true; do
    get_input "${YELLOW}" "Enter Manager Node Internal IP Address (e.g., 192.168.1.10): " manager_ip
    if validate_ip "$manager_ip"; then
      break
    else
      print_message "${RED}" "Invalid IP address format. Please try again."
    fi
  done

  local join_token
  get_input "${YELLOW}" "Enter Swarm Join Token (e.g., SWMTKN-1-xxxxxxxxxxxxxxxxxxxxxxxxx): " join_token
  # Basic validation for swarm token format
  if [[ ! "$join_token" =~ ^SWMTKN-[0-9]-[a-zA-Z0-9]+-[a-zA-Z0-9]+$ ]]; then
    print_message "${RED}" "Invalid Swarm Join Token format. Please ensure it starts with SWMTKN-X-..."
    print_message "${YELLOW}" "Example: SWMTKN-1-3x00m8l6d6w9k0o4q2z2c1t1v1m9g6h3b7f5r4s2p0u8a4e3a3f5j7t5c9j1e7k8a7j0o0t0q1i0l0d0c3i9a9b2r3k2i4d8c9h9o2s1m7e5f1y1n2h8g7e9i4v5k2x3o2v9r4s1p3u9p6v3d7e5d8a9c7b6c5j0t2c1l9s8q0t8u9g9s2k4m0s5e1n9c6f2a7x1l3w9t8f8g3z8q5j9t7m9k2o2s1q2w2r3e2t3y2u2i2o2p2l2k2j2h2g2f2d2s2a2z2x2c2v2b2n2m2"
    get_input "${BLUE}" "Press Enter to continue..." dummy
    return
  fi

  print_message "${YELLOW}" "Attempting to join Swarm at $manager_ip:2377..."
  if docker swarm join --token "$join_token" "$manager_ip":2377; then
    print_message "${GREEN}" "Successfully joined Docker Swarm!"
  else
    print_message "${RED}" "Failed to join Docker Swarm. Check Docker logs for details."
  fi
  get_input "${BLUE}" "Press Enter to continue..." dummy
}

# Option 3: Leave Docker Swarm
leave_swarm() {
  local swarm_status=$(get_swarm_status)
  if [ "$swarm_status" = "inactive" ] || [ "$swarm_status" = "error" ] || [ -z "$swarm_status" ]; then
    print_message "${YELLOW}" "This node is not part of a Swarm. No need to leave."
    get_input "${BLUE}" "Press Enter to continue..." dummy
    return
  fi

  print_message "${YELLOW}" "Attempting to leave current Swarm."
  print_message "${RED}" "WARNING: If this is the only manager, leaving will destroy the Swarm!"
  get_input "${RED}" "Are you absolutely sure you want to leave the swarm? (type 'yes' to confirm): " confirm_leave

  if [[ "$confirm_leave" == "yes" ]]; then # Exact 'yes' match for force
    if docker swarm leave --force; then
      print_message "${GREEN}" "Successfully left Docker Swarm!"
    else
      print_message "${RED}" "Failed to leave Docker Swarm. Check Docker logs for details."
    fi
  else
    print_message "${YELLOW}" "Swarm leave operation cancelled."
  fi
  get_input "${BLUE}" "Press Enter to continue..." dummy
}

# --- Main script execution ---
print_message "${BLUE}" "Checking Docker installation and service status..."

if ! is_package_installed docker-ce; then
  print_message "${RED}" "Error: 'docker-ce' package not found. Please install Docker first."
  print_message "${YELLOW}" "Refer to Docker's official documentation for installation: https://docs.docker.com/engine/install/"
  exit 1
fi

if ! is_docker_running; then
  print_message "${RED}" "Error: Docker service is not running. Please start Docker."
  print_message "${YELLOW}" "You can try: 'sudo systemctl start docker' or 'sudo systemctl enable --now docker'"
  exit 1
fi

print_message "${GREEN}" "Docker is installed and running."
main_menu
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Add execute permissions:**&#x42;ash

```bash
chmod +x /tmp/docker_swarm_setup.sh
```

**Execute the script with `sudo`:**

```bash
sudo /tmp/docker_swarm_setup.sh
```

**Script Options Explained:**

The script will present you with a main menu with the following options:

* **1) Initialize Docker Swarm (Create New From Scratch)**
  * **Purpose:** Use this option on the **first node** you want to make a Docker Swarm manager. This node will become the initial manager of your new Swarm cluster.
  * **Input Required:**
    * **Current Node Internal IP Address:** Enter the internal IP address of the machine you are currently running the script on (e.g., `192.168.1.10`). This IP will be advertised for other nodes to join.
  * **Output:** If successful, it will display the `docker swarm join-token` commands for both **worker nodes** and **manager nodes**. **Copy these tokens immediately** as they are crucial for adding other nodes to your Swarm. Store them securely!
  * **Note:** If the node is already part of a Swarm, it will prevent initialization. You must leave the existing Swarm first.
* **2) Join Docker Swarm (Swarm Already Created - Has at least 1 Manager)**
  * **Purpose:** Use this option on any other node you want to add to an existing Docker Swarm. You can add nodes as either workers or additional managers.
  * **Input Required:**
    * **Manager Node Internal IP Address:** Enter the internal IP address of an existing Swarm manager node.
    * **Swarm Join Token:** Paste the join token you obtained from the manager node (from option 1, or by running `docker swarm join-token worker` or `docker swarm join-token manager` on an existing manager).
  * **Validation:** The script performs a basic check on the token format (starts with `SWMTKN-X-...`).
  * **Note:** If the node is already part of a Swarm, it will prevent joining a new one. You must leave the existing Swarm first.
* **3) Leave Docker Swarm (Leave current swarm, even if sole member)**
  * **Purpose:** Use this option to make a node leave its current Docker Swarm.
  * **Confirmation:** The script will ask for confirmation, especially warning you if the node is the _only_ manager, as leaving will destroy the entire Swarm.
  * **Important:** This command uses `--force`, which means the node will leave even if it can't communicate with other managers. Use with caution, especially on manager nodes.
* **0) Exit**
  * Exits the script.

**Important Notes for Docker Swarm:**

* **Internal IP Addresses:** Always use the **internal IP addresses** of your nodes for Swarm commands (`--advertise-addr` and `join` commands). These are typically non-routable IPs within your local network (e.g., 192.168.x.x, 10.x.x.x).
* **Firewall:** Ensure no firewalls (like `ufw`) are blocking communication between your Swarm nodes on the necessary ports (2377/TCP, 7946/TCP/UDP, 4789/UDP).
* **Manager Node Quorum:** For a robust Swarm, aim for an odd number of manager nodes (e.g., 1, 3, or 5). If you only have one manager and it fails or leaves, the Swarm will cease to function.
* **Security:** Swarm join tokens grant significant power. Keep them secure and regenerate them if compromised.
* **Docker Compose for Swarm:** Once your Swarm is set up, you'll deploy your applications (like Immich) using `docker stack deploy` with a `docker-compose.yml` (or `stack.yml`) file. This is different from `docker-compose up`. Docker Compose files for Swarm require a `deploy` section for services.

#### **Cleaning Up the Script (Temporary File)**

Once your Docker and Docker Compose are set up as desired, the `docker_setup.sh` file in `/tmp` is no longer needed.

```bash
rm /tmp/docker_swarm_setup.sh
```
