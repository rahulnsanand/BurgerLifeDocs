# BurgerLife Automation Script

Let's take a look at how we can clone and setup the BurgerLife Automation Script using git on our Servers. This automation script includes installation, update and uninstallation logic in shell script for almost all the applications we will be configuring.

#### 1. Create a Documents folder if not already created.

```bash
mkdir /opt/Documents/
cd /opt/Documents
```

#### 2. Clone the Automation Script repository using git.

```bash
git clone https://github.com/rahulnsanand/BurgerLifeAutomation.git
```

#### 3. Make Script Executable

```bash
sudo chmod +x /opt/Documents/init.sh
```

#### 4. Create Symbolic Link

```bash
sudo ln -s /opt/Documents/init.sh /usr/local/bin/burgerscript
```

#### 5. Trigger Script

```bash
burgerscript
```

#### &#x20;
