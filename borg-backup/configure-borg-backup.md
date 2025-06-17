---
icon: sitemap
---

# Configure Borg Backup

#### **Install Borg**&#x20;

**Repeat this step on `node1`, `node2`, and `node3`.**

1.  **Update package lists:**

    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```
2.  **Install WireGuard tools:**

    ```bash
    sudo apt install borgbackup -y
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

&#x20;**Perform a Dry Run (Recommended First Step):** This allows you to verify the contents before applying them to your live `/opt`.

```bash
sudo borg extract --list --strip-components 1 /mnt/backup/opt_borg::opt_backup_07_Jun_2025_05_PM
```

* `/path/to/my/borg_repo::myhost-2025-06-17T10:00:00`: This is the full path to your repository and the specific archive name.
* `/tmp/opt_restore_test`: This is the target directory where Borg will extract the contents of the `/opt` folder from the backup. Borg will recreate the `/opt` directory structure inside `/tmp/opt_restore_test` (e.g., `/tmp/opt_restore_test/opt/...`).

***

#### **Step 4: Enable and Start WireGuard on All Nodes**

**Repeat this step on `node1`, `node2`, and `node3`.**

1.  **Bring up the WireGuard interface:**

    ```bash
    sudo wg-quick up wg0
    ```
2.  **Enable WireGuard to start on boot:**

    ```bash
    sudo systemctl enable wg-quick@wg0
    ```
3.  **To Restart WireGuard Service:**\


    ```bash
    sudo systemctl restart wg-quick@wg0
    ```
4.  **Check WireGuard status:**

    ```bash
    sudo wg
    ```

    You should see your interface details and the peers. If the connection is successful, you'll see a `latest handshake` timestamp and `transfer` data.

***

#### **Step 5: Test the Mesh Network**

From any node, you should now be able to ping the _WireGuard IP addresses_ of the other nodes.

If you can successfully ping between the `10.0.0.x` addresses, your WireGuard mesh network is active!

***

#### **Optional: Enable IP Forwarding (if nodes need to route traffic for each other)**

If you want your Raspberry Pis to act as routers for other devices on their respective LANs to reach the WireGuard network, or if you plan to introduce more complex routing, you'll need to enable IP forwarding.

**Repeat this on all nodes:**

1.  **Edit `sysctl.conf`:**

    ```bash
    sudo nano /etc/sysctl.conf
    ```
2.  **Uncomment (remove `#`) or add this line:**

    ```bash
    net.ipv4.ip_forward = 1
    ```
3. **Save and exit.**
4.  **Apply changes immediately:**

    ```bash
    sudo sysctl -p
    ```
