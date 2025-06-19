---
icon: film
---

# Jellyfin

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/jellyfin_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /opt/jellyfin/library:/config
      - /mnt/primary/media/movies:/movies
      - /mnt/primary/media/family:/family
      - /mnt/primary/media/shows:/tv
      - /mnt/primary/media/music:/music
      - /mnt/primary/media/prive:/prive
      - /mnt/primary/media/youtube:/youtube
      - /mnt/primary/media/courses:/courses
    ports:
      - 8096:8096 #HTTP
      - 8920:8920 #HTTPS
      - 7359:7359/udp #Allows clients to discover Jellyfin on the local network
      - 1900:1900/udp #Service discovery used by DNLA and clients
    restart: on-failure
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p jellyfin -f /tmp/jellyfin_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p jellyfin -f /tmp/jellyfin_docker-compose.yml create
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
rm /tmp/jellyfin_docker-compose.yml
```
