---
icon: folders
---

# File Browser

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/filebrowser_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    ports:
      - "11027:80"
    volumes:
      - /:/srv
      - /opt/filebrowser/filebrowser.db:/database.db
    user: "0:0" # Root user and group
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p filebrowser -f /tmp/filebrowser_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p filebrowser -f /tmp/filebrowser_docker-compose.yml create
```

You should see output indicating that the `nginx` container is being created and started.

**Verify the container is running:**

```bash
docker ps -a
```

**Open your web browser:** On your local computer, navigate to:

```bash
http://your_pi_ip_address:11013
```

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/filebrowser_docker-compose.yml
```
