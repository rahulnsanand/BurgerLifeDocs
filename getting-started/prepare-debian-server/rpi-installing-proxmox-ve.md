---
description: >-
  This page helps with setting up Raspberry Pi 5 OS to work with SD Card and
  then move onto NVMe drive as the primary boot option and effectively get rid
  of the SD Card
---

# RPI Installing Proxmox VE

### Preparing your Raspberry Pi for Proxmox <a href="#preparing-your-raspberry-pi-for-proxmox" id="preparing-your-raspberry-pi-for-proxmox"></a>

**1.** Our first task before installing Proxmox onto the Raspberry Pi is to update the package list cache and upgrade any out-of-date packages.

You can perform both tasks by using the following two commands within the terminal.

```bash
sudo apt update
sudo apt upgrade -y
```

**2.** Your next step is to ensure that curl is installed on your Pi. We will be using curl to grab the GPG key for the Proxmox ports repository that we will be relying on.

You can install this package by using the following command within the terminal.

```bash
sudo apt install curl -y
```

**3.** Before proceeding with this tutorial, you must set up your [Raspberry Pi to use a static IP address](https://pimylifeup.com/raspberry-pi-static-ip-address/).

The best way to do this is using DHCP reservation in your router. However, we have a guide that shows you how to do this through your Raspberry Pi if you don’t have access to your router.

### Modifying your Hosts File for Proxmox <a href="#modifying-your-hosts-file-for-proxmox" id="modifying-your-hosts-file-for-proxmox"></a>

**4.** After setting up a static IP address, we must now edit the hosts file. Proxmox expects your hostname to point to the IP address of your Raspberry Pi.

You can begin editing the hosts file on the Raspberry Pi [using the nano text editor](https://pimylifeup.com/nano-text-editor/) by running the following command.

```bash
sudo nano /etc/hosts
```

**5.** With the hosts file open, you should see a line similar to the one below. Our hostname is set to the default “`raspberrypi`“, but yours might differ.

```bash
127.0.0.1            raspberrypi
```

After finding this line, you will want to replace “`127.0.0.1`” with the local IP address of your Raspberry Pi.

In our case, our Pi’s IP is “`192.168.0.32`” so we changed the line to look like the following.

```bash
192.168.0.32            raspberrypi
```

**6.** After making these changes, save and quit by pressing <kbd>CTRL</kbd> + <kbd>X</kbd>, <kbd>Y</kbd>, and then the <kbd>ENTER</kbd> key.

**7.** To verify that our changes are working, we can [use the hostname command](https://pimylifeup.com/hostname-command/) to output the IP address of our Raspberry Pi.

```bash
hostname --ip-address
```

If you have configured everything properly, the static IP of your Pi should be returned.

```
192.168.0.32
```

### Set a Password for the root User <a href="#set-a-password-for-the-root-user" id="set-a-password-for-the-root-user"></a>

**8.** The next thing we must do to use Proxmox on our Raspberry Pi is set a password for the “`root`” user. If we don’t set a password for this user we will be unable to login through the Proxmox web interface.

You can set a password for the root user using the command below within the terminal.

```bash
sudo passwd root
```

**9.** To set a password, you must enter it twice, as shown below. We recommend that you set a strong password for this.

```bash
New password:
Retype new password:
```

### Adding the Proxmox Port Repository to your Raspberry Pi <a href="#adding-the-proxmox-port-repository-to-your-raspberry-pi" id="adding-the-proxmox-port-repository-to-your-raspberry-pi"></a>

**10.** We are finally at the stage where we can begin adding the [Proxmox Ports repository](https://github.com/jiangcuo/Proxmox-Port) to our Raspberry Pi. This repository is managed by a third-party but allows us to install versions compiled for the Raspberry Pi’s hardware.

Our first step in this process is to add the GPG key for the repository. This key helps verify that the packages legitimately come from the Proxmox ports repository.

```bash
curl -L https://mirrors.apqa.cn/proxmox/debian/pveport.gpg | sudo tee /usr/share/keyrings/pveport.gpg >/dev/null
```

**11.** With the GPG key added, we can now add the repository itself to our sources list.

Use the command below to save the repository link into a file called “`pveport.list`“.

```bash
echo "deb [arch=arm64 signed-by=/usr/share/keyrings/pveport.gpg] https://mirrors.apqa.cn/proxmox/debian/pve bookworm port" | sudo tee  /etc/apt/sources.list.d/pveport.list
```

**12.** Since we made changes to the sources list, we must update the package list cache again by running the following command.

If we don’t do this, the package manager will be unaware of the repository we just added.

```bash
sudo apt update
```

### Preparing your Pi’s Network for Proxmox <a href="#preparing-your-pi-s-network-for-proxmox" id="preparing-your-pi-s-network-for-proxmox"></a>

These next couple of sections will ensure that your network will continue to function once Proxmox has been installed. Without these steps you will lose network connection later.

**Installing ifupdown2**

**13.** With everything in place, we can finally begin installing Proxmox on the Raspberry Pi.

The first package that must be installed on our system is “`ifupdown2`“. Proxmox uses this package to handle the network.

```bash
sudo apt install ifupdown2
```

Please note, if you run into an error that says the following, there are a couple of extra

If you see the following error message when installing “`ifupdown2`” then you will need to follow steps “`a`” and “`b`” below. Otherwise, skip straight to **step 14**.

```bash
error: Another instance of this program is already running.
```

**a.** To work around the “`already running`” error, we will need to do some workaround steps to get ifupdown2 running.

This step involves deleting a temporary file that the ifupdown2 package is searching for during installation

```bash
sudo rm /tmp/.ifupdown2-first-install
```

**b.** With the temporary file removed, we need to re-run the install process for ifupdown2 again by using the following command.

This will ensure that the final setup steps are properly executed.

```bash
sudo apt install ifupdown2
```

### **Modifying Your Network Interfaces**

**14.** Our next step is to make some adjustments to the interfaces file. If we don’t make these changes, Proxmox will break when your Raspberry Pi restarts.

With these changes we will be setting our “`eth0`” interface to manual and create a new network interface that will bridge your connections and be used by your virtual machines.

You can begin to write this file by using the following command.

```bash
sudo nano /etc/network/interfaces
```

**15.** Within this file, you will want to add the following lines to the bottom of the file. There is a chance some of these interfaces are automatically specified, delete them and replace them with the values we have specified here

While filling out this file you must replace two values.

* `<IPADDRESS>`: Change this placeholder with the IP address of your Raspberry Pi. This must be the local IP that it is accessible on.
* `<GATEWAY>`**:** Replace this placeholder with the IP address of your gateway. Typically this will be the IP of your router.

```bash
auto lo
  iface lo inet loopback

auto eth0
  iface eth0 inet manual

auto vmbr0
iface vmbr0 inet manual
        address <IPADDRESS>
        gateway <GATEWAY>
        netmask 255.255.255.0
        bridge-ports eth0
        bridge-stp off
        bridge-fd 0
```

**16.** Once you have filled out this file, save and quit by pressing <kbd>CTRL</kbd> + <kbd>X</kbd>, <kbd>Y</kbd>, and then <kbd>ENTER</kbd>.

**17.** To ensure these network changes are working, restart your Raspberry Pi by running the following command in the terminal.

```bash
sudo reboot
```

### Installing Proxmox to the Raspberry Pi from the Repository <a href="#installing-proxmox-to-the-raspberry-pi-from-the-repository-1" id="installing-proxmox-to-the-raspberry-pi-from-the-repository-1"></a>

**18.** Finally, you can install Proxmox VE and a few required packages by typing the following command in the terminal.

This installation process can take a few minutes, so be prepared to wait a few minutes. Additionally, there are a few prompts you will have to go through during installation.

```bash
sudo apt install proxmox-ve postfix open-iscsi pve-edk2-firmware-aarch64 -y
```

**19.** During installation, you will be asked to configure Postfix. You can navigate this menu using the <kbd>ARROW</kbd> keys to move and <kbd>ENTER</kbd> to select.

If you are unsure what you are doing here, select the “`Local only`” option.

<figure><img src="https://pimylifeup.com/wp-content/uploads/2024/01/Raspberry-Pi-Proxmox-03-Set-Postfix-Choice-to-Local.jpg" alt="Choose Postfix Setup Choice" height="386" width="728"><figcaption></figcaption></figure>

**20.** Next, you will be asked to set the mail name for Postfix to utilize (**1.**).

Again, if you don’t know what you are doing here, leave the default name and press <kbd>ENTER</kbd> to continue (**2.**).

<figure><img src="https://pimylifeup.com/wp-content/uploads/2024/01/Raspberry-Pi-Proxmox-04-Set-System-mail-name.jpg" alt="Set System Mail name" height="386" width="728"><figcaption></figcaption></figure>

### Accessing the Proxmox Web Interface <a href="#accessing-the-proxmox-web-interface" id="accessing-the-proxmox-web-interface"></a>

**21.** Once Proxmox finishes installing to your Raspberry Pi, it should be safe to access its web interface.

All you need to access the web interface is the IP address of your Raspberry Pi.

```bash
hostname -I
```

**22.** With your IP address in hand, you will want to go to the following address in your favorite web browser. Please note if you can’t connect, try restarting your Raspberry Pi and waiting a few minutes.

Ensure that you replace “`<IPADDRESS>`” with the IP of your Raspberry Pi.

```
https://<IPADDRESS>:8006
```

**23.** With the Proxmox web interface now open in your web browser, you must log in to your account.

Start by typing in the username as “`root`” and then use the password you set earlier for this user (**1.**).

Once you have filled in your information, <kbd>click</kbd> the “`Login`” button (**2.**).

<figure><img src="https://pimylifeup.com/wp-content/uploads/2024/01/Raspberry-Pi-Proxmox-01-Login-to-Proxmox-VE.jpg" alt="Login to Proxmox VE on the Raspberry Pi" height="459" width="728"><figcaption></figcaption></figure>

**24.** You should now have access to the Proxmox web interface.

You can begin to set up new VMs through this interface as well as configure Proxmox VE itself.

<figure><img src="https://pimylifeup.com/wp-content/uploads/2024/01/Raspberry-Pi-Proxmox-02-Access-Web-Interface.jpg" alt="Proxmox Dashboard" height="459" width="728"><figcaption></figcaption></figure>
