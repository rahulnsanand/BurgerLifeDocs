---
description: >-
  This page helps with setting up Raspberry Pi 5 OS to work with SD Card and
  then move onto NVMe drive as the primary boot option and effectively get rid
  of the SD Card
---

# 'burger' User Creation

**Create the New User**

First, you need to create the user account. Replace `burger` with the desired username for your new user.

```bash
adduser burger
```

You'll be prompted to enter and confirm a password for the new user. Choose a strong, secure password. After that, you'll be asked for some optional information like full name, room number, etc. You can just press Enter to skip these if you don't need them. Finally, it will ask "Is the information correct? \[Y/n]". Type `Y` and press Enter.

**2. Add the User to the `sudo` Group**

To grant sudo privileges, you need to add the new user to the `sudo` group. Members of this group are allowed to execute commands with `sudo`.

```bash
usermod -aG sudo burger
```

* `usermod`: This command modifies user account properties.
* `-aG`: This option means "append to supplementary groups."
* `sudo`: This is the name of the group that grants sudo privileges.
* `burger`: The username you just created.

**3. Verify Sudo Privileges (Optional but Recommended)**

It's a good idea to verify that the new user can use `sudo`.

First, switch to the new user account:

```bash
su - burger
```

Now, try to execute a command with `sudo`, for example, to update the package lists:

```bash
sudo apt update
```

You'll be prompted to enter the password for `burger`. If everything is set up correctly, the `apt update` command should run without issues.

**4. Log Out and Log Back In (Important)**

For the group changes to take full effect, it's best to log out of your current session and then log back in as the new user.
