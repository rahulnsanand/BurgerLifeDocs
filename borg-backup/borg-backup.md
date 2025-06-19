---
icon: arrow-rotate-left
---

# Borg Backup

#### **Install Borg**&#x20;

1.  **Update package lists:**

    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```
2.  **Install BorgBackup tool:**

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

* `/path/to/my/borg_repo::myhost-2025-06-17T10:00:00`: This is the full path to your repository and the specific archive name.
* `/tmp/opt_restore_test`: This is the target directory where Borg will extract the contents of the `/opt` folder from the backup. Borg will recreate the `/opt` directory structure inside `/tmp/opt_restore_test` (e.g., `/tmp/opt_restore_test/opt/...`).

**Perform the ACTUAL RESTORE:**

```bash
sudo borg extract --list --strip-components 1 /mnt/backup/opt_borg::opt_backup_07_Jun_2025_05_PM
```

**--strip-components 1** Ensures we are ignoring the first level of the backup (i.e: /opt)

***
