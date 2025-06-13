---
description: >-
  This page helps in guiding how to setup SSH Server so we are never locked out
  of our servers remotely.
---

# SSH Setup

#### 1. **Update Your System**

First, ensure your package lists and installed packages are up to date:

```bash
sudo apt update
sudo apt upgrade
```

#### 2. **Install the OpenSSH Server Package**

Install the SSH server package:

```bash
sudo apt install openssh-server
```

#### 3. **Check SSH Service Status**

Verify that the SSH service is running:

```bash
sudo systemctl status ssh
```

* If it’s running, you’ll see “active (running)”.
*   If not, start it with:Code Example

    ```bash
    sudo systemctl start ssh
    ```

#### 4. **Enable SSH to Start on Boot**

Make sure SSH starts automatically after a reboot:

```bash
sudo systemctl enable ssh
```

#### 5. **Configure SSH (Optional but Recommended)**

Edit the SSH configuration file for security and customization:

```bash
sudo nano /etc/ssh/sshd_config
```

**Common settings to review:**

* **Port 22** (change for security, e.g., `Port 2222`)
* **PermitRootLogin yes** (not recommended in Production)

After making changes, save and exit (`Ctrl+X`, `Y`, `ENTER`).

#### 6. **Restart SSH Service to Apply Changes**

```bash
sudo systemctl restart ssh
```
