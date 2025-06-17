---
description: >-
  You have SSH access between Manager and Master (and vice-versa) using SSH keys
  for passwordless authentication. This is crucial for scripts to execute
  commands remotely.
icon: square-terminal
---

# Setup Passwordless SSH

#### Root User Creation (If not already done)

We will be using root user to run all the automation orchestration scripts due to overall access. Make sure this root user has a strong password and is capable of passwordless SSH between the devices.

```bash
sudo passwd root
```

```bash
su
cd ~
```

#### **Temporarily allow Root to login Remotely**

Edit the SSH configuration file for security and customization:

```bash
sudo nano /etc/ssh/sshd_config
```

* **PermitRootLogin yes** (not recommended in Production)

After making changes, save and exit (`Ctrl+X`, `Y`, `ENTER`).

#### **Restart SSH Service to Apply Changes**

```bash
sudo systemctl restart ssh
```

#### On Manager:&#x20;

```bash
ssh-keygen
```

* Save in default location
* Do Not Create Passphrase

```bash
ssh-copy-id root@10.21.22.15
```

#### On Master:&#x20;

```bash
ssh-keygen
```

* Save in default location
* Do Not Create Passphrase

```bash
ssh-copy-id root@10.21.22.16
```

#### Test the Passwordless SSH

```bash
ssh root@10.21.22.15
ssh root@10.21.22.16
```
