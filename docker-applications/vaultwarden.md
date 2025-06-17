---
icon: key
---

# VaultWarden

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/vaultwarden_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3'

services:
  vaultwarden:
    container_name: vaultwarden
    image: vaultwarden/server:latest
    restart: on-failure
    volumes:
      - /opt/vaultwarden/data:/data
    ports:
      - "11013:80"
      - "11014:3012"
    environment:
      - ADMIN_TOKEN=Yt7kcHGSMRXdRGeyrg/soEljAdjcx0K51eQtSIj4vXU=
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

**Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p vaultwarden -f /tmp/vaultwarden_docker-compose.yml up -d
```

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
rm /tmp/vaultwarden_docker-compose.yml
```

