---
icon: cloud
---

# NextCloud

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/nextcloud_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3.8'
services:
  nextcloud-aio-mastercontainer:
    image: nextcloud/all-in-one:latest
    container_name: nextcloud-aio-mastercontainer
    init: true
    restart: unless-stopped
    ports:
      - '808:8080'
    environment:
      APACHE_PORT: 11000
      APACHE_IP_BINDING: 0.0.0.0
      NEXTCLOUD_MOUNT: "/mnt/primary/nextcloud/"
      NEXTCLOUD_DATADIR: "/mnt/primary/nextcloud/ncdata"
      NEXTCLOUD_MEMORY_LIMIT: 1024M
      #AIO_COMMUNITY_CONTAINERS: "facerecognition"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /opt/Documents/nextcloud_backup:/opt/Documents/nextcloud_backup
      - /mnt/backup:/mnt/backup
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config
volumes:
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer

```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p nextcloud -f /tmp/nextcloud_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p nextcloud -f /tmp/nextcloud_docker-compose.yml create
```

You should see output indicating that the `nginx` container is being created and started.

**Verify the container is running:**

```bash
docker ps -a
```

**Open your web browser:** On your local computer, navigate to:

```css
https://your_server_ip_address:808
```

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/nextcloud_docker-compose.yml
```
