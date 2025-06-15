---
icon: swap-arrows
---

# Nginx Proxy Manager

**Create the `npm_docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/npm_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

**ATTENTION:**

* **`DB_MYSQL_HOST`**: **CHANGE `"10.10.10.10"` to the WireGuard IP of one of your MariaDB Galera Cluster nodes** (e.g., `10.0.0.1`).
* **`DB_MYSQL_PASSWORD`**: **CHANGE `"Rasdasdasda06"` to the actual password for your `burger` MariaDB user.**

```yaml
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx
    restart: unless-stopped
    ports:
      - '80:80'    # HTTP traffic for proxied sites
      - '443:443'  # HTTPS traffic for proxied sites
      - '81:81'    # Nginx Proxy Manager Admin UI
    environment:
      # MariaDB/MySQL connection parameters:
      DB_MYSQL_HOST: "10.10.10.5" # !!! CHANGE THIS TO YOUR MARIADB CLUSTER NODE'S WIREGUARD IP !!!
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

**Navigate to the `/tmp` directory:**

```bash
cd /tmp
```

*   **Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

    ```bash
    docker-compose -f npm_docker-compose.yml up -d
    ```

    You should see output indicating that the `nginx` container is being created and started.
*   **Verify the container is running:**

    ```bash
    docker ps -a
    ```

    You should see `nginx` listed with `Status: Up ...`.

**Open your web browser:** On your local computer, navigate to:

```
http://your_ip_address:81
```
