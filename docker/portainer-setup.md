---
icon: grid-horizontal
---

# Portainer Setup

**Create the `portainer_docker-compose.yml` file:** Use `nano` to create the file in the `/tmp` directory:

```bash
nano /tmp/portainer_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3.8' # Using 3.8 for consistency, though '3' is fine too
services:
  portainer:
    image: portainer/portainer-ce:latest # Using :latest tag for explicit clarity
    container_name: portainer
    restart: always
    ports:
      - "9000:9000" # HTTP port (often redirects to 9443)
      - "9443:9443" # HTTPS UI port
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Essential for Docker management
      - /opt/portainer:/data # Persistent data storage for Portainer
    environment:
      - TZ=Asia/Kolkata # Example for setting timezone
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p portainer -f /tmp/portainer_docker-compose.yml up -d
```

You should see output indicating that the `portainer` container is being created and started.

**Wait a few seconds and then restart Portainer (as requested):** This step is unusual for a typical `docker-compose up -d` as it starts the container. However, if you've encountered issues or have a specific reason for this, we'll include it.

```bash
sleep 10
docker restart portainer
```

You should see `portainer` as output, indicating a successful restart.

> SETUP NEW PORTAINER INSTANCE RIGHT AFTER RESTARTING

**Verify the container is running:**

```bash
docker ps -a
```

You should see `portainer` listed with `Status: Up ...`.

**Open your web browser:** Navigate to:

```
http://your_ip_address:9000
```

(Replace `your_ip_address` with the actual IP address of your Raspberry Pi.)

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/portainer_docker-compose.yml
```

You now have Portainer up and running, providing a powerful GUI for managing your Docker setup!





