---
icon: user
---

# User burger Creation

**Connect to MariaDB as `root`:**&#x20;

```bash
sudo mysql -u root -p
```

When prompted, enter the MariaDB `root` password.

**Create the user and grant privileges:**

```sql
DROP USER IF EXISTS 'burger'@'%';
DROP USER IF EXISTS 'burger'@'localhost';
FLUSH PRIVILEGES;
CREATE USER 'burger'@'localhost' IDENTIFIED BY 'ENTER_PASSWORD';
CREATE USER 'burger'@'%' IDENTIFIED BY 'ENTER_PASSWORD';

GRANT ALL PRIVILEGES ON *.* TO 'burger'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'burger'@'%' WITH GRANT OPTION;

FLUSH PRIVILEGES;
```

**Important:**

* **`'burger'@'%'`**: This means the user `burger` can connect from _any_ host (`%`). If you want to restrict connections to specific IPs (e.g., only from your WireGuard network's `10.0.0.0/24` subnet), you would use `'burger'@'10.0.0.%'` or `'burger'@'10.0.0.1'` for specific nodes. For "all db artifacts" and flexibility, `'%'` is usually implied for such a task.
* **`'your_strong_password_here'`**: **CHANGE THIS IMMEDIATELY!** Use a unique, strong password.
* **`WITH GRANT OPTION`**: This is powerful. It means `burger` can create new users and grant them privileges. If `burger` doesn't need to do this, omit this part.
*   **Verify the new user (Optional):** You can try logging in as the new user from the same Raspberry Pi:

    Bash

    ```bash
    mysql -u burger -p
    ```

    Enter the password you just set for `burger`. If you get the `MariaDB [(none)]>` prompt, the user was created successfully. You can then `EXIT;`.
