# Debian OS Setup

### Installing Debian on Your Server

#### Prerequisites

* Access to your server's console
* Debian installation ISO
* Backup of existing data, if applicable

#### Preparation Steps

1. **Download Debian ISO:**\
   The latest netinst ISO at the time of writing this blog is here in the below link: [https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.11.0-amd64-netinst.iso](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.11.0-amd64-netinst.iso)
2. **Download Rufus (Portable/Installer):**\
   Rufus is a bootable drive write that can be downloaded from below link:\
   [https://rufus.ie/en/](https://rufus.ie/en/)
3. Write the ISO to a USB stick (at least 4GB).
4. #### **Boot from USB**
   * Insert the USB drive into your server.
   * Reboot the server and enter the BIOS/UEFI setup (for MSI motherboards it's usually by pressing `Del` during startup).
   * Set the USB drive as the primary boot device.
   * Save and exit BIOS/UEFI.

#### **Debian 12 Installation Process**

**a. Start the Installer**

* Select "Install" or "Graphical Install" from the Debian boot menu.

**b. Language, Location, and Keyboard**

* Choose your preferred language, location, and keyboard layout.

**c. Configure Network**

* The installer will attempt to configure your network via DHCP.
* If DHCP fails or you need a static IP, configure it manually.

**d. Set Hostname and Domain**

* Enter a hostname for your server (e.g., `burgermaster`).
* Enter a domain name if applicable (can be left blank for home use).

**e. Set Up Users and Passwords**

* Set a root password (or leave blank to disable direct root login).
* Create a regular user account and password.

**f. Partition Disks**

* **If replacing an existing OS:**
  * Select "Guided - use entire disk" to overwrite the current OS and data.
  * If you want to manually partition, choose "Manual" and delete existing partitions before creating new ones.
* **If dual-booting:**
  * Choose "Manual" and resize or add partitions as needed.

**g. Select Software**

* Choose the software you want to install (e.g., "SSH server", "standard system utilities").
* Unselect the desktop environment and GNOME options.

**h. Install GRUB Bootloader**

* Install GRUB to the primary disk.
* Confirm when prompted.

**i. Finish Installation**

* Remove the USB drive when prompted.
* Reboot the server.

If you're unable to do basic commands like 'reboot' or 'sudo' try the below steps

*   **Check `/root/.bashrc`:**

    Bash

    ```bash
    nano /root/.bashrc
    ```

    Look for `PATH=` lines. If found, modify them as above.
*   **If still not found or you want to ensure it's always set for root:** You can _add_ the `export PATH` line to the end of `/root/.bashrc` (or `/root/.profile` if you prefer it for login shells) if it's not explicitly set elsewhere:

    Bash

    ```bash
    export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games"
    ```

    (Ensure you don't duplicate `PATH` settings if they already exist, just modify the existing one.)

**After making the permanent change:**

1. **Save the file** in `nano` (Ctrl+O, Enter, Ctrl+X).
2. **Either log out and log back in** as root (best way to ensure new `PATH` is loaded).
3.  **Or, source the configuration file** to apply changes immediately to your current session without logging out:Bash

    ```bash
    source /root/.profile
    ```

    Then run `echo $PATH` again to confirm it's correct.

#### Disable GRUB

Log into the server through SSH/Local Terminal - log into super user (ROOT)

```bash
su 
nano /etc/default/grub
```

Look for these lines and adjust as follows:

* **GRUB\_TIMEOUT=0**\
  This makes GRUB boot immediately into the default OS.
* **GRUB\_TIMEOUT\_STYLE=hidden**\
  This hides the menu unless you press a key.

```bash
update-grub
```

```bash
reboot
```









You're now ready to use Debian on your server! For advanced configurations, consult the official Debian documentation.
