---
icon: swap-arrows
---

# Nginx Proxy Manager

**Create the `npm_docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/npm_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

**Access the MariaDB/MySQL Monitor:**

You need to log in as a user with sufficient privileges to create databases. This is typically the `root` user or a user you've created with `CREATE` privileges.

**As root (most common for administrative tasks):**

```bash
sudo mysql -u root -p
```

You'll be prompted to enter the `root` password for your MariaDB installation

Once you're inside the `mysql` monitor (the prompt will change to `MariaDB [(none)]>` or `mysql>`), use the `CREATE DATABASE` command:

```sql
CREATE DATABASE npm_db;
```

**ATTENTION:**

* **`DB_MYSQL_HOST`**: **CHANGE `"10.10.10.10"` to the WireGuard IP of one of your MariaDB Galera Cluster nodes** (e.g., `10.0.0.1`).
* **`DB_MYSQL_PASSWORD`**: **CHANGE \`**&#x79;our\_actual\_burger\_passwor&#x64;**\` to the actual password for your `burger` MariaDB user.**

```yaml
version: '3.8'
services:
  nginx:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx
    restart: on-failure
    ports:
      - '80:80'    # HTTP traffic for proxied sites
      - '443:443'  # HTTPS traffic for proxied sites
      - '81:81'    # Nginx Proxy Manager Admin UI
    environment:
      # MariaDB/MySQL connection parameters:
      DB_MYSQL_HOST: "10.21.22.5" # !!! CHANGE THIS TO YOUR MARIADB CLUSTER NODE'S WIREGUARD IP !!!
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "burger"
      DB_MYSQL_PASSWORD: "your_actual_burger_password" # !!! CHANGE THIS TO YOUR ACTUAL PASSWORD !!!
      DB_MYSQL_NAME: "npm_db" # The database created in Part 1
      # Other optional environment variables can go here (refer to NPM docs)
    volumes:
      - /opt/nginx/data:/data
      - /opt/nginx/letsencrypt:/etc/letsencrypt
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p nginx -f /tmp/npm_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p nginx -f /tmp/npm_docker-compose.yml create
```

You should see output indicating that the `nginx` container is being created and started.

**Verify the container is running:**

```bash
docker ps -a
```

You should see `nginx` listed with `Status: Up ...`.

**Open your web browser:** On your local computer, navigate to:

```
http://your_ip_address:81
```

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/npm_docker-compose.yml
```
