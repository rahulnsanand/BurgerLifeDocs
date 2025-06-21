---
icon: arrow-rotate-left
---

# Borg Backup PENDING

**Create the `deploy_adguard.sh` file:** Use `nano` to create a new file:

```bash
nano /tmp/deploy_borg.sh
```

**Paste the refined script content:** Copy the following content and paste it into the `nano` editor. This version will prompt you for all the necessary dynamic values.

{% code overflow="wrap" %}
```bash
#!/bin/bash

# Define ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
BLUE='\033[0;34m'
RESET='\033[0m'

# --- Configuration Variables (User Customizable) ---
# These variables can be set once or prompted from the user.
# For a truly dynamic script, we'll prompt the user for these.

# Global variables for paths (will be set interactively)
BACKUP_SCRIPT_DIR="/opt/Documents/BurgerLifeAutomation/shell_scripts/backup_scripts"
BACKUP_SCRIPT_NAME="backup_opt.sh"
BACKUP_SCRIPT_PATH="" # Will be constructed later
SERVICE_NAME="borg_backup_opt" # Dynamic service name
SERVICE_FILE="" # Will be constructed later
TIMER_FILE="" # Will be constructed later

# Borg specific variables (will be set interactively)
BORG_REPO_BASE_DIR="/mnt/backup" # Base directory for Borg repositories
BORG_REPO_NAME="opt_borg"
BORG_REPO_PATH="" # Will be constructed later
BACKUP_SOURCE_PATH="/opt" # Default backup source
BORG_PASSPHRASE="" # Will be prompted securely
KEEP_DAILY_BACKUPS=14 # Default retention policy

# Webhook notification (optional, can be prompted)
WEBHOOK_URL="http://10.21.22.5:6719/notify/script_notification"

# --- Helper Functions ---

# Function to print colored messages
print_message() {
  local color="$1"
  local message="$2"
  echo -e "${color}${message}${RESET}"
}

# Function to check if a command exists
command_exists() {
  command -v "$1" &> /dev/null
}

# Function to check if a package is installed on Raspbian/Debian OS
is_package_installed() {
  local package_name="$1"
  dpkg-query -l "$package_name" &> /dev/null
}

# Function to prompt for user input with a default value
get_user_input() {
  local prompt_message="$1"
  local default_value="$2"
  # Print the colored prompt using print_message
  print_message "${BLUE}${prompt_message} [Default: ${default_value}]:${RESET}"
  # Read the user input on a new line
  read user_input
  echo "${user_input:-$default_value}"
}

# Function to prompt for sensitive user input (e.g., passphrase)
get_sensitive_input() {
  local prompt_message="$1"
  # Print the colored prompt using print_message
  print_message "${BLUE}${prompt_message}: ${RESET}"
  # Read the user input silently on a new line
  read -s sensitive_input
  echo # Add a newline after sensitive input for better formatting
  echo "$sensitive_input"
}

# Function to send a notification via webhook
send_webhook() {
  if [ -z "$WEBHOOK_URL" ]; then
    print_message "${YELLOW}" "Webhook URL not configured. Skipping notification."
    return
  fi

  local message="$1"
  local status="$2"

  print_message "${BLUE}" "Sending webhook notification (Status: ${status})..."
  if command_exists curl; then
    curl -s -X POST -H "Content-Type: application/json" \
         -d '{
         "sender": "server_script",
         "recipient": "is_admin",
         "message": "'"${message}"'",
         "instance": "server",
         "action": "notify",
         "status": "'"${status}"'",
         "tvdb": -1,
         "tmdb": -1
         }' "$WEBHOOK_URL"         
    if [ $? -eq 0 ]; then
      print_message "${GREEN}" "Webhook notification sent successfully."
    else
      print_message "${RED}" "Failed to send webhook notification."
    fi
  else
    print_message "${RED}" "Curl is not installed. Cannot send webhook notifications."
  fi
}

# --- Core Setup Functions ---

## Initial Setup and Configuration

# This section gathers necessary information from the user, making the script highly interactive.

interactive_setup() {
  print_message "${BLUE}" "--- Borg Backup Interactive Setup ---"

  # Prompt for Borg Repository details
  BORG_REPO_BASE_DIR=$(get_user_input "Enter the base directory for Borg repositories" "$BORG_REPO_BASE_DIR")
  BORG_REPO_NAME=$(get_user_input "Enter the name for this Borg repository" "$BORG_REPO_NAME")
  BORG_REPO_PATH="${BORG_REPO_BASE_DIR}/${BORG_REPO_NAME}"

  # Prompt for backup source path
  BACKUP_SOURCE_PATH=$(get_user_input "Enter the directory you want to back up" "$BACKUP_SOURCE_PATH")
  if [ ! -d "$BACKUP_SOURCE_PATH" ]; then
    print_message "${RED}" "Warning: The specified backup source directory '${BACKUP_SOURCE_PATH}' does not exist."
    echo -en "${YELLOW}Do you want to proceed anyway? (y/N): ${RESET}"
    read proceed_anyway
    if [[ ! "$proceed_anyway" =~ ^[Yy]$ ]]; then
      print_message "${RED}" "Setup aborted. Please provide a valid backup source."
      exit 1
    fi
  fi

  # Prompt for Borg Passphrase securely
  while true; do
    BORG_PASSPHRASE=$(get_sensitive_input "Enter the Borg passphrase (will not be echoed)")
    if [ -n "$BORG_PASSPHRASE" ]; then
      break
    else
      print_message "${RED}" "Passphrase cannot be empty. Please try again."
    fi
  done

  # Prompt for retention policy
  KEEP_DAILY_BACKUPS=$(get_user_input "Enter the number of daily backups to keep" "$KEEP_DAILY_BACKUPS")
  if ! [[ "$KEEP_DAILY_BACKUPS" =~ ^[0-9]+$ ]] || [ "$KEEP_DAILY_BACKUPS" -lt 1 ]; then
    print_message "${RED}" "Invalid input for daily backups. Using default: ${KEEP_DAILY_BACKUPS}."
    KEEP_DAILY_BACKUPS=14
  fi

  # Prompt for backup script details
  BACKUP_SCRIPT_DIR=$(get_user_input "Enter the directory for the backup script" "$BACKUP_SCRIPT_DIR")
  BACKUP_SCRIPT_NAME=$(get_user_input "Enter the name for the backup script" "$BACKUP_SCRIPT_NAME")
  BACKUP_SCRIPT_PATH="${BACKUP_SCRIPT_DIR}/${BACKUP_SCRIPT_NAME}"

  # Prompt for service name
  SERVICE_NAME=$(get_user_input "Enter the desired name for the systemd service (e.g., borg_backup_opt)" "$SERVICE_NAME")
  SERVICE_FILE="/etc/systemd/system/${SERVICE_NAME}.service"
  TIMER_FILE="/etc/systemd/system/${SERVICE_NAME}.timer"

  # Prompt for webhook URL (optional)
  WEBHOOK_URL=$(get_user_input "Enter the Webhook URL for notifications (leave blank to disable)" "$WEBHOOK_URL")

  print_message "${GREEN}" "Configuration complete!"
  print_message "${BLUE}" "--- Review Configuration ---"
  echo "Borg Repository: ${BORG_REPO_PATH}"
  echo "Backup Source: ${BACKUP_SOURCE_PATH}"
  echo "Backup Script Path: ${BACKUP_SCRIPT_PATH}"
  echo "Systemd Service Name: ${SERVICE_NAME}"
  echo "Retention Policy (Daily): ${KEEP_DAILY_BACKUPS}"
  echo "Webhook URL: ${WEBHOOK_URL:-Disabled}"
  print_message "${BLUE}" "----------------------------"

  echo -en "${YELLOW}Proceed with this configuration? (Y/n): ${RESET}"
  read confirm_config
  if [[ ! "$confirm_config" =~ ^[Yy]$ && -n "$confirm_config" ]]; then
    print_message "${RED}" "Setup cancelled by user."
    exit 0
  fi
}

## BorgBackup Installation and Repository Management
# ---
# This section handles the installation and uninstallation of BorgBackup, as well as the initialization of the repository.

uninstall_borgbackup() {
  if is_package_installed borgbackup; then
    print_message "${RED}" "Uninstalling borgbackup..."
    sudo apt-get remove --purge -y borgbackup || { print_message "${RED}" "Error uninstalling borgbackup."; return 1; }
    sudo apt-get autoremove -y
    sudo apt-get clean
    print_message "${GREEN}" "BorgBackup uninstallation complete."
  else
    print_message "${GREEN}" "BorgBackup is not installed."
  fi
  return 0
}

install_borgbackup() {
  print_message "${YELLOW}" "Checking for BorgBackup installation..."
  if is_package_installed borgbackup; then
    print_message "${GREEN}" "BorgBackup is already installed."
    echo -en "${YELLOW}Do you want to reinstall BorgBackup? (y/N): ${RESET}"
    read reinstall_choice
    if [[ "$reinstall_choice" =~ ^[Yy]$ ]]; then
      uninstall_borgbackup || return 1
      print_message "${YELLOW}" "Installing borgbackup..."
      sudo apt-get update || { print_message "${RED}" "Error updating package lists."; return 1; }
      sudo apt-get install -y borgbackup || { print_message "${RED}" "Error installing borgbackup."; return 1; }
      print_message "${GREEN}" "BorgBackup reinstallation complete."
    fi
  else
    print_message "${YELLOW}" "BorgBackup not found. Installing now..."
    sudo apt-get update || { print_message "${RED}" "Error updating package lists."; return 1; }
    sudo apt-get install -y borgbackup || { print_message "${RED}" "Error installing borgbackup."; return 1; }
    print_message "${GREEN}" "BorgBackup installation complete."
  fi

  # Initialize the Borg Repository if it doesn't exist
  print_message "${YELLOW}" "Checking Borg Repository at ${BORG_REPO_PATH}..."
  if [ ! -d "$BORG_REPO_PATH" ]; then
    print_message "${YELLOW}" "Creating Borg Repository directory: ${BORG_REPO_PATH}..."
    sudo mkdir -p "$BORG_REPO_PATH" || { print_message "${RED}" "Error creating repository directory."; return 1; }
    sudo chmod 700 "$BORG_REPO_PATH" # Ensure proper permissions

    print_message "${YELLOW}" "Initializing the Borg Repository. You will be prompted for the passphrase."
    export BORG_PASSPHRASE # Temporarily export for borg init
    borg init --encryption=repokey "$BORG_REPO_PATH"
    unset BORG_PASSPHRASE # Unset for security
    if [ $? -eq 0 ]; then
      print_message "${GREEN}" "Borg Repository initialized successfully."
    else
      print_message "${RED}" "Error initializing Borg Repository. Please check passphrase and permissions."
      return 1
    fi
  else
    print_message "${GREEN}" "Borg Repository already exists at ${BORG_REPO_PATH}. Skipping initialization."
  fi
  return 0
}

## Dynamic Backup Script Generation
# ---
# This function creates the actual backup script based on user-defined configurations.

create_dynamic_backup_script() {
  print_message "${YELLOW}" "Generating dynamic backup script: ${BACKUP_SCRIPT_PATH}..."

  sudo mkdir -p "$BACKUP_SCRIPT_DIR" || { print_message "${RED}" "Error creating script directory."; return 1; }

  # Write the backup script content
  sudo bash -c "cat <<EOF > ${BACKUP_SCRIPT_PATH}
#!/bin/bash

# --- Script Generated by Borg Backup Setup Script ---
# This script performs a BorgBackup for the configured source.

# Define variables from setup script
BACKUP_REPO=\"${BORG_REPO_PATH}\"
BACKUP_SOURCE=\"${BACKUP_SOURCE_PATH}\"
TIMESTAMP=\$(date '+%Y-%m-%d_%H-%M-%S')
BACKUP_NAME=\"\$(basename \${BACKUP_SOURCE})_\${TIMESTAMP}\"
BORG_PASSPHRASE=\"${BORG_PASSPHRASE}\"
KEEP_DAILY_BACKUPS=${KEEP_DAILY_BACKUPS}
WEBHOOK_URL=\"${WEBHOOK_URL}\"

# Function to send a notification via webhook
send_webhook() {
  if [ -z \"\$WEBHOOK_URL\" ]; then
    echo \"Webhook URL not configured. Skipping notification.\"
    return
  fi

  local message=\"\$1\"
  local status=\"\$2\"

  echo \"Sending webhook notification (Status: \${status})...\"
  if command -v curl &> /dev/null; then
    curl -s -X POST -H \"Content-Type: application/json\" \\
         -d '{
         \"sender\": \"server_script\",
         \"recipient\": \"is_admin\",
         \"message\": \"'\${message}'\",
         \"instance\": \"server\",
         \"action\": \"notify\",
         \"status\": \"'\${status}'\",
         \"tvdb\": -1,
         \"tmdb\": -1
         }' \"\$WEBHOOK_URL\" &> /dev/null
    if [ \$? -eq 0 ]; then
      echo \"Webhook notification sent successfully.\"
    else
      echo \"Failed to send webhook notification.\"
    fi
  else
    echo \"Curl is not installed. Cannot send webhook notifications.\"
  fi
}

# Export the passphrase for Borg
export BORG_PASSPHRASE

# Create the backup using Borg
echo \"Starting Borg backup for \${BACKUP_SOURCE} to \${BACKUP_REPO}::\${BACKUP_NAME}...\"
borg create --verbose --stats --progress \\
    --exclude '/opt/swapfile16G' \\
    --exclude '/opt/vmlinuz' \\
    --exclude '/opt/vmlinuz.old' \\
    \"\${BACKUP_REPO}::\${BACKUP_NAME}\" \"\${BACKUP_SOURCE}\"

BACKUP_EXIT_CODE=\$?

if [ \$BACKUP_EXIT_CODE -ne 0 ]; then
  send_webhook \"Error: Borg backup of \${BACKUP_SOURCE} failed (\${TIMESTAMP}).\" \"error\"
  echo \"Borg backup failed with exit code \$BACKUP_EXIT_CODE.\"
  unset BORG_PASSPHRASE # Unset the passphrase
  exit \$BACKUP_EXIT_CODE
fi

echo \"Borg backup successful. Pruning old backups...\"

# Prune old backups based on retention policy
borg prune --verbose --list \"\${BACKUP_REPO}\" --keep-daily=\${KEEP_DAILY_BACKUPS}
PRUNE_EXIT_CODE=\$?

if [ \$PRUNE_EXIT_CODE -ne 0 ]; then
  send_webhook \"Warning: Borg prune for \${BACKUP_SOURCE} failed (\${TIMESTAMP}).\" \"warning\"
  echo \"Borg prune failed with exit code \$PRUNE_EXIT_CODE.\"
else
  echo \"Borg prune successful.\"
fi

# Unset the passphrase after the backup is done for security reasons
unset BORG_PASSPHRASE

# If everything is successful (or prune had minor issues), send a success notification
if [ \$BACKUP_EXIT_CODE -eq 0 ]; then
  send_webhook \"Success: Borg backup of \${BACKUP_SOURCE} completed (\${TIMESTAMP}).\" \"success\"
fi

# Exit script with the backup exit code
exit \$BACKUP_EXIT_CODE
EOF"

  sudo chmod +x "$BACKUP_SCRIPT_PATH" || { print_message "${RED}" "Error setting execute permissions on script."; return 1; }
  print_message "${GREEN}" "Dynamic backup script created: ${BACKUP_SCRIPT_PATH}"
  return 0
}

## Systemd Service and Timer Management
# ---
# These functions create, remove, enable, and disable systemd units, allowing for automated backups.

create_systemd_service() {
  print_message "${YELLOW}" "Creating systemd service for backup script: ${SERVICE_FILE}..."

  sudo bash -c "cat <<EOF > ${SERVICE_FILE}
[Unit]
Description=Borg Backup Service for ${BACKUP_SOURCE_PATH}
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=${BACKUP_SCRIPT_PATH}
User=root
Group=root
EOF"

  sudo systemctl daemon-reload || { print_message "${RED}" "Error reloading systemd daemon."; return 1; }
  print_message "${GREEN}" "Systemd service created: ${SERVICE_FILE}"
  return 0
}

create_systemd_timer() {
  print_message "${YELLOW}" "Creating systemd timer for backup service: ${TIMER_FILE}..."

  # Offer custom timer schedules
  print_message "${BLUE}" "--- Configure Backup Schedule ---"
  echo "Choose a scheduling option:"
  echo "1) Daily at a specific time"
  echo "2) Multiple times a day (e.g., 3am and 5pm)"
  echo "3) Weekly on specific days"
  echo "4) Custom OnCalendar expression (advanced)"
  echo -en "${BLUE}Enter your choice (1-4): ${RESET}"
  read schedule_choice

  TIMER_SCHEDULE=""
  case "$schedule_choice" in
    1)
      echo -en "${BLUE}Enter the daily backup time (e.g., 03:00 for 3 AM): ${RESET}"
      read daily_time
      if [[ ! "$daily_time" =~ ^([01]?[0-9]|2[0-3]):[0-5][0-9]$ ]]; then
        print_message "${RED}" "Invalid time format. Using default: 03:00:00"
        daily_time="03:00"
      fi
      TIMER_SCHEDULE="OnCalendar=*-*-* ${daily_time}:00"
      ;;
    2)
      echo -en "${BLUE}Enter first backup time (e.g., 03:00): ${RESET}"
      read time1
      echo -en "${BLUE}Enter second backup time (e.g., 17:00):: ${RESET}"
      read time2
      if [[ ! "$time1" =~ ^([01]?[0-9]|2[0-3]):[0-5][0-9]$ || ! "$time2" =~ ^([01]?[0-9]|2[0-3]):[0-5][0-9]$ ]]; then
        print_message "${RED}" "Invalid time format. Using default: 03:00:00 and 17:00:00"
        TIMER_SCHEDULE="OnCalendar=*-*-* 03:00:00\nOnCalendar=*-*-* 17:00:00"
      else
        TIMER_SCHEDULE="OnCalendar=*-*-* ${time1}:00\nOnCalendar=*-*-* ${time2}:00"
      fi
      ;;
    3)
      echo -en "${BLUE}Enter days of the week (e.g., Mon,Wed,Fri): ${RESET}" 
      read days_of_week
      echo -en "${BLUE}Enter time for weekly backups (e.g., 04:00): ${RESET}"       
      read weekly_time
      if [[ -z "$days_of_week" || ! "$weekly_time" =~ ^([01]?[0-9]|2[0-3]):[0-5][0-9]$ ]]; then
        print_message "${RED}" "Invalid input. Using default: Mon,Wed,Fri at 04:00:00"
        TIMER_SCHEDULE="OnCalendar=Mon,Wed,Fri *-*-* 04:00:00"
      else
        TIMER_SCHEDULE="OnCalendar=${days_of_week} *-*-* ${weekly_time}:00"
      fi
      ;;
    4)
      echo -en "${BLUE}Enter custom OnCalendar expression (e.g., annually, weekly): ${RESET}"
      read custom_calendar
      if [ -z "$custom_calendar" ]; then
        print_message "${RED}" "No custom expression entered. Using default: daily at 03:00:00 and 17:00:00"
        TIMER_SCHEDULE="OnCalendar=*-*-* 03:00:00\nOnCalendar=*-*-* 17:00:00"
      else
        TIMER_SCHEDULE="OnCalendar=${custom_calendar}"
      fi
      ;;
    *)
      print_message "${RED}" "Invalid choice. Using default: daily at 03:00:00 and 17:00:00"
      TIMER_SCHEDULE="OnCalendar=*-*-* 03:00:00\nOnCalendar=*-*-* 17:00:00"
      ;;
  esac

  sudo bash -c "cat <<EOF > ${TIMER_FILE}
[Unit]
Description=Timer to run Borg Backup Service for ${BACKUP_SOURCE_PATH}

[Timer]
${TIMER_SCHEDULE}
Persistent=true

[Install]
WantedBy=timers.target
EOF"

  sudo systemctl daemon-reload || { print_message "${RED}" "Error reloading systemd daemon."; return 1; }
  print_message "${GREEN}" "Systemd timer created: ${TIMER_FILE}"
  return 0
}

enable_and_start_service() {
  print_message "${YELLOW}" "Enabling and starting systemd timer for ${SERVICE_NAME}.timer..."

  sudo systemctl enable "${SERVICE_NAME}.timer" || { print_message "${RED}" "Error enabling timer."; return 1; }
  sudo systemctl start "${SERVICE_NAME}.timer" || { print_message "${RED}" "Error starting timer."; return 1; }
  sudo systemctl status "${SERVICE_NAME}.timer" --no-pager # Show status
  print_message "${GREEN}" "Systemd timer enabled and started."
  return 0
}

remove_systemd_service_and_timer() {
  print_message "${YELLOW}" "Stopping and disabling systemd timer and service for ${SERVICE_NAME}..."

  # Stop and disable the timer and service
  sudo systemctl stop "${SERVICE_NAME}.timer" 2>/dev/null
  sudo systemctl disable "${SERVICE_NAME}.timer" 2>/dev/null
  sudo systemctl stop "${SERVICE_NAME}.service" 2>/dev/null
  sudo systemctl disable "${SERVICE_NAME}.service" 2>/dev/null

  # Remove the service and timer files
  if [ -f "$SERVICE_FILE" ]; then
    print_message "${YELLOW}" "Removing service file: ${SERVICE_FILE}..."
    sudo rm -f "$SERVICE_FILE" || { print_message "${RED}" "Error removing service file."; }
    print_message "${GREEN}" "Service file removed."
  else
    print_message "${GREEN}" "Service file does not exist. No action needed."
  fi

  if [ -f "$TIMER_FILE" ]; then
    print_message "${YELLOW}" "Removing timer file: ${TIMER_FILE}..."
    sudo rm -f "$TIMER_FILE" || { print_message "${RED}" "Error removing timer file."; }
    print_message "${GREEN}" "Timer file removed."
  else
    print_message "${GREEN}" "Timer file does not exist. No action needed."
  fi

  sudo systemctl daemon-reload || { print_message "${RED}" "Error reloading systemd daemon."; }
  print_message "${GREEN}" "Systemd timer and service removal complete."
  return 0
}

## Main Control Flow
# ---
# The main function provides a menu-driven interface for the user to perform various actions.

main_menu() {
  while true; do
    print_message "${BLUE}" "\n--- Borg Backup Setup & Management ---"
    echo "1) Setup New Borg Backup (Install, Configure, Schedule)"
    echo "2) Uninstall BorgBackup"
    echo "3) Remove Current Backup Service & Timer"
    echo "4) Run Backup Script Manually (for testing)"
    echo "5) View Systemd Timer Status"
    echo "6) Exit"
    print_message "${BLUE}" "--------------------------------------"

    read -rp "Enter your choice: " choice

    case "$choice" in
      1)
        interactive_setup || continue
        install_borgbackup || continue
        create_dynamic_backup_script || continue
        remove_systemd_service_and_timer # Clean up existing if any
        create_systemd_service || continue
        create_systemd_timer || continue
        enable_and_start_service || continue
        print_message "${GREEN}" "Borg Backup setup complete and scheduled!"
        ;;
      2)
        uninstall_borgbackup
        ;;
      3)
        # Before removal, we need to know what service/timer to remove.
        # This assumes the last configured service or prompts for it.
        if [ -z "$SERVICE_NAME" ]; then
          print_message "${YELLOW}" "No active service name known. Please enter the service name to remove."
          SERVICE_NAME=$(get_user_input "Enter the service name (e.g., borg_backup_opt)" "borg_backup_opt")
          SERVICE_FILE="/etc/systemd/system/${SERVICE_NAME}.service"
          TIMER_FILE="/etc/systemd/system/${SERVICE_NAME}.timer"
        fi
        remove_systemd_service_and_timer
        ;;
      4)
        if [ -f "$BACKUP_SCRIPT_PATH" ]; then
          print_message "${YELLOW}" "Running backup script manually..."
          sudo "$BACKUP_SCRIPT_PATH"
          if [ $? -eq 0 ]; then
            print_message "${GREEN}" "Manual backup completed successfully."
          else
            print_message "${RED}" "Manual backup failed. Check the script output for errors."
          fi
        else
          print_message "${RED}" "Backup script not found at ${BACKUP_SCRIPT_PATH}. Please run option 1 first."
        fi
        ;;
      5)
        if [ -n "$SERVICE_NAME" ]; then
          print_message "${BLUE}" "Viewing status for ${SERVICE_NAME}.timer..."
          sudo systemctl status "${SERVICE_NAME}.timer" --no-pager
          print_message "${BLUE}" "Viewing status for ${SERVICE_NAME}.service..."
          sudo systemctl status "${SERVICE_NAME}.service" --no-pager
        else
          print_message "${YELLOW}" "No service name configured yet. Please run setup first or provide the service name."
          echo -en "${BLUE}Enter the service name to check status for (e.g., borg_backup_opt): ${RESET}"
          read check_service_name
          if [ -n "$check_service_name" ]; then
            sudo systemctl status "${check_service_name}.timer" --no-pager
            sudo systemctl status "${check_service_name}.service" --no-pager
          else
            print_message "${RED}" "No service name provided."
          fi
        fi
        ;;
      6)
        print_message "${GREEN}" "Exiting. Goodbye!"
        exit 0
        ;;
      *)
        print_message "${RED}" "Invalid choice. Please enter a number between 1 and 6."
        ;;
    esac
  done
}

# Run the main menu
main_menu
```
{% endcode %}

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

#### **Make the Script Executable**

```bash
chmod +x /tmp/deploy_borg.sh
```

**Execute the script with `sudo`:**

```bash
sudo /tmp/deploy_borg.sh
```

***

**List archives in the repository:** This command will show you all the backups (archives) available in your repository.

```bash
sudo borg list /mnt/backup/opt_borg
```

***



**Perform the restore:** You can restore to the original location (`/opt`) or to a temporary location first for verification. **Restoring to a temporary location is highly recommended for safety.**

**a) Perform a Dry Run (Recommended First Step):** This allows you to verify the contents before applying them to your live `/opt`.

```bash
sudo borg extract --dry-run --list /mnt/backup/opt_borg::opt_backup_07_Jun_2025_05_PM
```

* `/path/to/my/borg_repo::myhost-2025-06-17T10:00:00`: This is the full path to your repository and the specific archive name.
* `/tmp/opt_restore_test`: This is the target directory where Borg will extract the contents of the `/opt` folder from the backup. Borg will recreate the `/opt` directory structure inside `/tmp/opt_restore_test` (e.g., `/tmp/opt_restore_test/opt/...`).

**Perform the ACTUAL RESTORE:**

```bash
sudo borg extract --list --strip-components 1 /mnt/backup/opt_borg::opt_backup_07_Jun_2025_05_PM
```

**--strip-components 1** Ensures we are ignoring the first level of the backup (i.e: /opt)

***
