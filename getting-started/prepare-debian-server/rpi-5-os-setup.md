---
description: >-
  This page helps with setting up Raspberry Pi 5 OS to work with SD Card and
  then move onto NVMe drive as the primary boot option and effectively get rid
  of the SD Card
---

# RPi 5 - OS Setup

#### **Phase 1: Prepare the SD Card with Debian Headless OS**

This phase involves writing the base operating system to your SD card.

1. **Download Raspberry Pi Imager:**
   * On your computer (Windows, macOS, or Linux), download and install the official Raspberry Pi Imager from [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/).
2. **Insert SD Card:**
   * Insert your 64GB SD card into your computer's SD card reader.
3. **Write Debian Headless OS using Imager:**
   * Open Raspberry Pi Imager.
   * Click **"CHOOSE OS"**:
     * Select "Raspberry Pi OS (other)"
     * Then choose **"Raspberry Pi OS Lite (64-bit)"**
   * Click **"CHOOSE STORAGE"**:
     * Select your 64GB SD card. **Double-check that you are selecting the correct drive!**
   * **Configure Headless Settings (VERY IMPORTANT for Remote Login):**
     * Click the **gear icon (⚙️)** in the bottom right corner (or press `Ctrl+Shift+X` on Windows/Linux).
     * **Set username and password:** Create your desired username (e.g., `burger`) and a strong password. **Remember these!**
     * **Click "Save."**
   * Click **"WRITE"**:
     * Confirm the write operation. This will erase all data on the SD card.
     * Wait for the imaging process to complete and verify.

***

#### **Phase 2: Initial Boot from SD Card and Remote Access**

This phase gets your Pi up and running, accessible via SSH.

1. **Mount SD Card & NVMe:**
   * Safely eject the SD card from your computer.
   * Insert the prepared SD card into the Raspberry Pi 5's SD card slot.
   * **Ensure your NVMe drive is correctly installed in its HAT** and the HAT is securely connected to the Raspberry Pi 5.
2. **Power On and Connect to Network:**
   * Connect an Ethernet cable from your RPi 5 to your router/switch (recommended for initial setup).
   * Connect the power supply to your RPi 5.
   * Turn on the RPi. The green activity light on the Pi should flicker during boot.
3. **Find Raspberry Pi's IP Address:**
   * **Check your router's connected devices list** (most reliable).
4. **Remote Login (SSH):**
   * Open a terminal on your computer.
   *   Connect via SSH using the username and IP address you identified:

       ```bash
       ssh burger@<raspberry_pi_ip_address>
       ```
   * Accept the fingerprint if prompted.
   * Enter the password you set during imaging.

***

#### **Phase 3: Configure Boot Order & Prepare NVMe for Cloning**

Here, you'll ensure the Pi prioritizes booting from the SD card and then clean up the NVMe drive.

1. **Check Current Drive Status:**
   *   Once logged into the Pi, run `lsblk` to confirm your devices:

       ```bash
       lsblk
       ```
   *   **Expected Output (similar to yours, but `/` on `mmcblk0p2`):**

       ```bash
       NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
       mmcblk0     179:0    0 59.5G  0 disk
       ├─mmcblk0p1 179:1    0  512M  0 part /boot/firmware
       └─mmcblk0p2 179:2    0   59G  0 part /
       nvme0n1     259:0    0 476.9G  0 disk
       ├─nvme0n1p1 259:1    0  512M  0 part # Potentially mounted if previous OS was there
       └─nvme0n1p2 259:2    0 476.4G  0 part # Potentially mounted if previous OS was there
       ```
   * **Crucial Check:** Verify that `/` (your root filesystem) is mounted on `mmcblk0p2`. This confirms you are currently running from the SD card.
2. **Configure Boot Order (Prioritize SD Card):**
   * This is essential to ensure your Pi continues to boot from the SD card after you've cloned the OS to the NVMe, before you decide to permanently switch.
   *   Run `raspi-config`:

       ```bash
       sudo raspi-config
       ```
   * Navigate using arrow keys and Enter:
     * **Advanced Options**
     * **Boot Order**
     * Select **"SD Card Boot (prioritise SD card)"**
     * Confirm your choice and exit `raspi-config`.
   *   **Reboot the Raspberry Pi:**

       ```bash
       sudo reboot
       ```
   * **Remote Login again** after the reboot.
3. **Verify Boot Order and Unmount NVMe Partitions:**
   * After logging back in, run `lsblk` again to confirm `/` is still on `mmcblk0p2`.
   * **Identify NVMe Partitions:** Note the partitions on `/dev/nvme0n1` (e.g., `nvme0n1p1`, `nvme0n1p2`).
   *   **Unmount ALL NVMe Partitions:** This is critical before cloning.

       ```bash
       sudo umount /dev/nvme0n1p?
       ```
4.  **Completely Wipe NVMe Drive (Crucial - ERASES ALL DATA!):**

    * This ensures a clean slate and avoids issues during cloning.
    * **WARNING: This command will permanently delete ALL data on your NVMe drive (`/dev/nvme0n1`). Double-check the device name!** \<!-- end list -->

    ```bash
    sudo wipefs --all --force /dev/nvme0n1p?
    sudo wipefs --all --force /dev/nvme0n1
    sudo dd if=/dev/zero of=/dev/nvme0n1 bs=1024 count=1
    ```

    * The `wipefs` command removes filesystem and partition table signatures.
    * The `dd` command writes zeros to the first block, ensuring the old partition table is completely gone.

***

#### **Phase 4: Clone OS from SD Card to NVMe**

Now, you'll use `rpi-clone` to copy your SD card OS to the NVMe.

1.  **Install `git` (if not already installed):**

    ```bash
    sudo apt update
    sudo apt install git -y
    ```
2.  **Download `rpi-clone`:**

    ```bash
    git clone https://github.com/geerlingguy/rpi-clone.git
    cd rpi-clone
    ```
3.  **Install `rpi-clone` scripts:**

    ```bash
    sudo cp rpi-clone rpi-clone-setup /usr/local/sbin/
    ```
4. **Clone to the NVMe drive:**
   * **Identify your NVMe device again with `lsblk` if unsure, it should still be `/dev/nvme0n1`.**
   *   Run the cloning command:

       ```bash
       sudo rpi-clone /dev/nvme0n1
       ```
   * `rpi-clone` will guide you through the process. It will automatically:
     * Create appropriate partitions on the NVMe.
     * Copy files.
     * Update `fstab` and `cmdline.txt` on the NVMe clone to point to its own partitions (using PARTUUIDs).
     * **Crucially, it will prompt you to confirm the destination drive multiple times. Read carefully and ensure it's `/dev/nvme0n1`!**
   * Wait for `rpi-clone` to complete. This can take some time.

***

#### **Phase 5: Boot from NVMe (Optional - for Faster Performance)**

After successfully cloning, you can switch to booting directly from the NVMe.

1. **Verify Clone (Optional but Recommended):**
   * After `rpi-clone` finishes, it's a good idea to perform a quick check, though `rpi-clone` is very reliable.
   * Run `lsblk` and you should see new partitions on `/dev/nvme0n1` (e.g., `nvme0n1p1` and `nvme0n1p2`) which are not mounted.
2.  **Open the EEPROM configuration for editing:**

    ```bash
    sudo rpi-eeprom-config --edit
    ```

    This command will open the bootloader configuration file in the `nano` text editor.
3.  **Modify the `BOOT_ORDER`:**

    * Locate the line that begins with `BOOT_ORDER=`.
    * **Change it to:**

    ```bash
    BOOT_ORDER=0xf416
    ```

    This setting prioritizes booting from the **SD card (`1`)** first, then the **NVMe/USB (`6`)**, then USB mass storage (`4`), and finally repeats the sequence (`f`) if none are found. (Remember, the order is read right-to-left in the hex value).
4. **Add `PCIE_PROBE=1` (if necessary):**
   1. If you are using a **non-HAT+ NVMe adapter** (i.e., not an official Raspberry Pi HAT+), you might need to explicitly enable PCIe device probing for the NVMe to be detected as a boot option.
5. **Add this line to the file:**

```bash
PCIE_PROBE=1
```

If you're using an official HAT+, this line might not be strictly necessary, but it doesn't hurt to include it to ensure compatibility.

6. **Save and Exit:**
   1. Press `Ctrl` + `O` (to Write Out the changes).
   2. Press `Enter` to confirm the filename.
   3. Press `Ctrl` + `X` (to eXit the editor).
7.  Enabling PCIe Gen 3.0 Speeds for NVMe on Raspberry Pi 5

    **Access `config.txt`:** Open a terminal on your Raspberry Pi 5 (via SSH or directly). You'll need root privileges to edit this file.

    Bash

    ```
    sudo nano /boot/firmware/config.txt
    ```
8.  **Add the PCIe Gen 3.0 Parameter:** Scroll to the end of the `config.txt` file (or find a suitable place, like under other `dtparam` lines). Add the following line:

    ```
    dtparam=pciex1_gen=3
    ```

    It should look something like this in the file (other lines will be present):

    ```
    # ... other existing configurations ...

    # Enable PCIe Gen 3.0 for NVMe (unsupported, test for stability)
    dtparam=pciex1_gen=3
    ```
9. **Save and Exit:**
   * Press `Ctrl` + `O` (to Write Out) to save the file.
   * Press `Enter` to confirm the filename.
   * Press `Ctrl` + `X` (to eXit) the `nano` editor.
10. **Remove the SD Card:**
    1. Once the Pi is completely off (no lights, or stable red light), remove the SD card.
    2. **This is essential to force the Pi to try booting from the NVMe.**
11. **Reboot:**
    1. The changes will take effect after a reboot:

```bash
sudo reboot
```

9. **Final Verification:**
   1. Run `lsblk` again. You should now see `/` (root) mounted on `nvme0n1p2` and `/boot/firmware` mounted on `nvme0n1p1`.
   2. Your Raspberry Pi 5 is now running its OS directly from the faster NVMe drive!
