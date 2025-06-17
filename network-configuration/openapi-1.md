---
icon: network-wired
---

# Cloudflared Configuration

#### Visit CloudFlare Zero Trust Dashboard

{% embed url="https://one.dash.cloudflare.com/" %}

1. **Under Networks > Tunnels > Create a Tunnel**
2. **Give your Tunnel a name (eg. 'asherslife-tunnel')**
3. **Once Tunnel is created, you'll be asked to set it up on your server, choose 'Docker' to containerize the tunnel deployment on your server.**

**Create the `portainer_docker-compose.yml` file:** Use `nano` to create the file in the `/tmp` directory:

```bash
nano /tmp/setup_cloudflared.sh
```

**Paste the below Script content:** Copy the following content and paste it into the `nano` editor

<pre class="language-bash"><code class="lang-bash">#!/bin/bash

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

print_message "${YELLOW}" "--- Cloudflared Tunnel Container Setup ---"

# --- Input for Token ---
read -s -p "Enter your Cloudflare Tunnel Token (starts with 'ey...'): " CF_TUNNEL_TOKEN
echo "" # New line after silent input

if [[ -z "$CF_TUNNEL_TOKEN" ]]; then
    print_message "${RED}" "Error: Cloudflare Tunnel Token cannot be empty."
    exit 1
fi

# --- Container Name ---
read -p "Enter a desired container name (e.g., cloudflared-tunnel, my-app-tunnel): " CONTAINER_NAME
if [[ -z "$CONTAINER_NAME" ]]; then
    CONTAINER_NAME="cloudflared-tunnel"
    print_message "${YELLOW}" "No container name entered. Defaulting to '$CONTAINER_NAME'."
fi

# --- Restart Policy ---
# RESTART_POLICY_PROMPT=$(cat &#x3C;&#x3C;EOF
# Choose a restart policy:
#   [1] no         (Do not automatically restart the container.)
#   [2] on-failure (Restart only if the container exits with a non-zero exit code.)
#   [3] unless-stopped (Restart unless the container is explicitly stopped or Docker daemon is stopped.)
#   [4] always     (Always restart the container, even if it's stopped manually.)
# Enter choice (1-4, default: 3):
# EOF
# )
# read -p "$RESTART_POLICY_PROMPT" RESTART_POLICY_CHOICE

RESTART_POLICY="on-failure" # Default
# case "$RESTART_POLICY_CHOICE" in
#     1) RESTART_POLICY="no" ;;
#     2) RESTART_POLICY="on-failure" ;;
#     3) RESTART_POLICY="unless-stopped" ;;
<strong>#     4) RESTART_POLICY="always" ;;
</strong>#     *) print_message "${YELLOW}" "Invalid choice. Defaulting to 'unless-stopped'." ;;
# esac

DOCKER_COMPOSE_FILE_PATH="/tmp/docker-compose-${CONTAINER_NAME}.yml"

rm "$DOCKER_COMPOSE_FILE_PATH"

DOCKER_COMPOSE_CONTENT=$(cat &#x3C;&#x3C;EOF
version: '3.8'
services:
  ${CONTAINER_NAME}:
    image: cloudflare/cloudflared:latest
    container_name: ${CONTAINER_NAME}
    restart: ${RESTART_POLICY}
    command: tunnel --no-autoupdate run --token ${CF_TUNNEL_TOKEN}
EOF
)

print_message "${GREEN}" "Writing Docker Compose file to: $DOCKER_COMPOSE_FILE_PATH"
echo "$DOCKER_COMPOSE_CONTENT" > "$DOCKER_COMPOSE_FILE_PATH"

print_message "${BLUE}" "\nTo deploy with Docker Compose:"
print_message "${GREEN}" "  sudo docker-compose -p ${CONTAINER_NAME} -f ${DOCKER_COMPOSE_FILE_PATH} up -d"
print_message "${YELLOW}" "To stop and remove:"
print_message "${GREEN}" "  sudo docker-compose -p ${CONTAINER_NAME} -f ${DOCKER_COMPOSE_FILE_PATH} down"

# --- Generate Docker Run Command ---
print_message "${BLUE}" "\n--- Generating Docker Run Command ---"
DOCKER_RUN_COMMAND="sudo docker run -d \\"
DOCKER_RUN_COMMAND+=" --name ${CONTAINER_NAME} \\"
DOCKER_RUN_COMMAND+=" --restart ${RESTART_POLICY} \\"
DOCKER_RUN_COMMAND+=" cloudflare/cloudflared:latest \\"
DOCKER_RUN_COMMAND+=" tunnel --no-autoupdate run --token ${CF_TUNNEL_TOKEN}"

print_message "${GREEN}" "Your generated Docker Run command:"
print_message "${GREEN}" "$DOCKER_RUN_COMMAND"

print_message "${YELLOW}" "\n--- Next Steps ---"
print_message "${YELLOW}" "Choose either the Docker Compose or Docker Run method to deploy your tunnel."
print_message "${YELLOW}" "Remember to manage your tunnel's DNS records via the Cloudflare dashboard."
print_message "${YELLOW}" "For more advanced configurations (like ingress rules, multiple tunnels), consult the Cloudflared documentation."

# Unset sensitive variables
unset CF_TUNNEL_TOKEN
</code></pre>

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /tmp/setup_cloudflared.sh
```

**Execute the script:**&#x42;ash

```bash
sudo /tmp/setup_cloudflared.sh
```

**Remove the script:**&#x42;ash

```bash
rm /tmp/setup_cloudflared.sh
```
