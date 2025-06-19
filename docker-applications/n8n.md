---
icon: robot
---

# N8N

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/n8n_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
services:
  n8n:
    container_name: n8n
    image: docker.n8n.io/n8nio/n8n
    restart: unless-stopped
    ports:
      - "11006:5678"
    environment:
      - N8N_HOST=automate.asherslife.in
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://automate.asherslife.in/
      - GENERIC_TIMEZONE=Asia/Kolkata
    volumes:
      - /opt/n8n:/home/node/.n8n
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

**Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p n8n -f /tmp/n8n_docker-compose.yml up -d
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
rm /tmp/n8n_docker-compose.yml
```
