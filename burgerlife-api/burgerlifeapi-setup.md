---
icon: webhook
---

# BurgerLifeAPI Setup

**Create the `npm_docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/deploy_webhook.sh
```

**Paste the Script content:** Copy the following content and paste it into the `nano` editor.

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

# --- Configuration Variables ---
GITHUB_USERNAME="rahulnsanand"
REPO_NAME="BurgerLifeWebHook" # Extracted from the URL for clarity
DESTINATION_DIR="/opt/$REPO_NAME"
LOG_DIR="$DESTINATION_DIR/logs"
venv_dir="/opt/${REPO_NAME}PythonEnv" # Changed to match repo name for clarity
SERVICE_NAME="burgerlife_webhook.service" # Using the name from the script directly

# Prompt securely for the GitHub Personal Access Token
print_message "${YELLOW}" "Please enter your GitHub Personal Access Token for $GITHUB_USERNAME/$REPO_NAME:"
read -s ACCESS_TOKEN # -s makes input silent (secure)

# Construct the repository URL with the token
REPO_URL="https://$GITHUB_USERNAME:$ACCESS_TOKEN@github.com/$GITHUB_USERNAME/$REPO_NAME"

# Function to check and create the destination directory
check_create_destination_dir() {
  if [ -d "$DESTINATION_DIR" ]; then
    print_message "${YELLOW}" "Repository directory $DESTINATION_DIR already exists. Deleting it and virtual environment..."
    sudo rm -r "$DESTINATION_DIR" || { print_message "${RED}" "Failed to remove existing $DESTINATION_DIR. Aborting."; exit 1; }
    if [ -d "$venv_dir" ]; then
      sudo rm -r "$venv_dir" || { print_message "${RED}" "Failed to remove existing $venv_dir. Aborting."; exit 1; }
    fi
  fi
  print_message "${YELLOW}" "Creating destination directory: $DESTINATION_DIR"
  sudo mkdir -p "$DESTINATION_DIR" || {
    print_message "${RED}" "Failed to create destination directory: $DESTINATION_DIR. Aborting."
    exit 1
  }
  # Ensure permissions allow cloning into it
  sudo chown "$USER":"$USER" "$DESTINATION_DIR" || { print_message "${RED}" "Failed to set ownership for $DESTINATION_DIR. Aborting."; exit 1; }
}

# Function to clone the repository
clone_repository() {
  print_message "${YELLOW}" "Cloning repository from GitHub..."
  git clone "$REPO_URL" "$DESTINATION_DIR" && {
    print_message "${GREEN}" "Repository cloned successfully."
  } || {
    print_message "${RED}" "Failed to clone repository. Check your PAT and network."
    exit 1
  }
}

# Function to make shell scripts executable
make_shell_scripts_executable() {
  print_message "${YELLOW}" "Making shell scripts executable in $DESTINATION_DIR..."
  find "$DESTINATION_DIR" -type f -name "*.sh" -exec chmod +x {} \;
}

# Function to remove a systemd service if it exists
remove_service() {
  print_message "${YELLOW}" "Checking for and removing existing service: $1..."
  if systemctl -q is-active "$1" || systemctl -q is-enabled "$1"; then
      sudo systemctl stop "$1"
      sudo systemctl disable "$1"
      sudo rm "/etc/systemd/system/$1"
      sudo systemctl daemon-reload
      print_message "${GREEN}" "Service $1 removed."
  else
      print_message "${YELLOW}" "Service $1 not found or not active/enabled."
  fi
}

# --- Script Execution Starts Here ---

# Stop the service if it's running (using the correct service name)
remove_service "$SERVICE_NAME"

# Prepare directories and clone
check_create_destination_dir
clone_repository

# Create logs directory
print_message "${YELLOW}" "Creating logs directory: $LOG_DIR"
sudo mkdir -p "$LOG_DIR" || {
  print_message "${RED}" "Failed to create logs directory: $LOG_DIR. Aborting."
  exit 1
}
# Ensure permissions allow writing to logs
sudo chown "$USER":"$USER" "$LOG_DIR" || { print_message "${RED}" "Failed to set ownership for $LOG_DIR. Aborting."; exit 1; }

make_shell_scripts_executable

# Create and configure virtual environment
print_message "${YELLOW}" "Creating and configuring Python virtual environment..."
python3 -m venv "$venv_dir" || { print_message "${RED}" "Failed to create virtual environment. Aborting."; exit 1; }

source "$venv_dir/bin/activate" || { print_message "${RED}" "Failed to activate virtual environment. Aborting."; exit 1; }

print_message "${YELLOW}" "Installing Python dependencies from requirements.txt..."
pip install -r "$DESTINATION_DIR/requirements.txt" || { print_message "${RED}" "Failed to install Python dependencies. Aborting."; exit 1; }

deactivate

print_message ${GREEN} "Virtual environment created and configured successfully."

print_message ${YELLOW} "Configuring Webhook Systemd Service..."

# Create and configure the systemd service
cat <<EOF | sudo tee "/etc/systemd/system/$SERVICE_NAME" > /dev/null
# Configuration
[Unit]
Description=BurgerLifeWebHook Service
After=network.target

[Service]
User=root
WorkingDirectory=/opt/BurgerLifeWebHook
Environment="PATH=/opt/BurgerLifePythonEnv/bin"
ExecStart=/opt/BurgerLifePythonEnv/bin/python /opt/BurgerLifeWebHook/main.py
Restart=always

[Install]
WantedBy=multi-user.target
EOF

print_message ${YELLOW} "Reloading Systemd Daemon and Enabling/Starting the Service..."
# Reload systemd and enable/start the service
sudo systemctl daemon-reload
sudo systemctl enable "$SERVICE_NAME"
sudo systemctl restart "$SERVICE_NAME"

# Check service status
if sudo systemctl is-active --quiet "$SERVICE_NAME"; then
  print_message "${GREEN}" "WebHook service '$SERVICE_NAME' has been configured and is running."
  print_message "${GREEN}" "Webhook Is Now Running!"
  print_message "${YELLOW}" "Check service logs with: journalctl -xeu $SERVICE_NAME"
  print_message "${YELLOW}" "Or check specific log files at: /opt/BurgerLifeWebHook/logs/service.log and /opt/BurgerLifeWebHook/logs/error.log"
else
  print_message "${RED}" "Failed to start WebHook service '$SERVICE_NAME'."
  print_message "${RED}" "Check logs for details: journalctl -xeu $SERVICE_NAME"
  print_message "${RED}" "Or specific log files at: /opt/BurgerLifeWebHook/logs/service.log and /opt/BurgerLifeWebHook/logs/error.log"
fi

# Unset the ACCESS_TOKEN variable for security
unset ACCESS_TOKEN
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /tmp/deploy_webhook.sh
```

#### **Run the Script**

```bash
sudo /tmp/deploy_webhook.sh
```

**Provide your GitHub Personal Access Token (PAT):** The script will prompt you:

```bash
Please enter your GitHub Personal Access Token for rahulnsanand/BurgerLifeWebHook:
```

#### **Verify the Webhook Service**

1.  **Check the service status:**

    Bash

    ```bash
    sudo systemctl status burgerlife_webhook.service
    ```

    Look for `Active: active (running)` in green.
2.  **Check the service logs:** To see the recent output from your Python application:

    Bash

    ```bash
    journalctl -xeu burgerlife_webhook.service
    ```

    Or check the dedicated log files defined in the service:

    Bash

    ```bash
    sudo cat /opt/BurgerLifeWebHook/logs/service.log
    sudo cat /opt/BurgerLifeWebHook/logs/error.log
    ```

#### **Cleaning Up the Script (Temporary File)**

Once your webhook service is successfully deployed and running, the `deploy_webhook.sh` file in `/tmp` is no longer needed.

```
rm /tmp/deploy_webhook.sh
```
