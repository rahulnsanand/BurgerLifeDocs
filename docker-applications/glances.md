---
icon: chart-pie-simple
---

# Glances

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/glances_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
services:
  glances:
    image: nicolargo/glances:latest
    container_name: glances
    pid: host
    ports:
      - 61208-61209:61208-61209
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /run/user/1000/podman/podman.sock:/run/user/1000/podman/podman.sock:ro
      - /mnt/primary:/mnt/primary
      - /mnt/secondary:/mnt/secondary
      - /etc/os-release:/etc/os-release:ro
    environment:
      - "GLANCES_OPT=-w"
      - TZ=Asia/Kolkata
    restart: on-failure
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

**Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p glances -f /tmp/glances_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

**Verify the container is running:**

```bash
docker ps -a
```

**Open your web browser:** On your local computer, navigate to:

```bash
http://your_pi_ip_address:8096
```

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/homepage_docker-compose.yml
```
