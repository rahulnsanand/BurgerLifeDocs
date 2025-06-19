---
icon: folders
---

# ARR Stack

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/arr_stack_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    ports:
      - 8000:8000/tcp #API Access Port
      - 8888:8888/tcp
      - 8889:8889/tcp
      - 6881:6881 #qBittorrent
      - 6881:6881/udp #qBittorrent UDP
      - 8070:8080 #qBittorrent UI
      - 8989:8989 #Sonarr
      - 7878:7878 #Radarr
      - 5055:5055 #Overseerr
      - 9696:9696 #Prowlarr
      - 11023:3000 #Firefox
      - 11025:80 #Speedtest
      - 11030:6767 #Bazarr
      - 6246:6246 #Maintainerr
    volumes:
      - /opt/gluetun:/gluetun
    environment:
      - VPN_SERVICE_PROVIDER=surfshark
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=<Private_KEY>
      - WIREGUARD_ADDRESSES=10.14.0.2/16
      - TZ=Asia/Kolkata
      - UPDATER_PERIOD=24h
      - FIREWALL_OUTBOUND_SUBNETS=10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
      - SERVER_COUNTRIES=Singapore,Hong Kong,Netherlands,Switzerland
      
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /opt/radarr/data:/config
      - /mnt/primary/media/movies:/movies
      - /mnt/primary/media/downloads:/downloads
      - /mnt/primary/media/downloads/completed:/downloads/completed
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
    
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /opt/sonarr/data:/config
      - /mnt/primary/media/shows:/tv
      - /mnt/primary/media/downloads:/downloads
      - /mnt/primary/media/downloads/completed:/downloads/completed
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
    
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /opt/prowlarr/data:/config
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
    
  overseerr:
    image: sctx/overseerr:latest
    container_name: overseerr
    environment:
      - LOG_LEVEL=debug
      - TZ=Asia/Kolkata
      - PORT=5055
    volumes:
      - /opt/overseerr/config:/app/config
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy

  maintainerr:
    image: ghcr.io/jorenn92/maintainerr:latest
    container_name: maintainerr
    user: 1000:1000
    volumes:
      - /opt/maintainerr/data:/opt/data
    environment:
      - TZ=Asia/Kolkata
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
    
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /opt/bazarr/data:/config
      - /mnt/primary/media/movies:/movies
      - /mnt/primary/media/shows:/tv
      - /mnt/primary/media/youtube:/youtube
      - /mnt/primary/media/courses:/courses
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /opt/qbittorrent/appdata:/config
      - /mnt/primary/media/downloads:/downloads
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
        
  vpn_speedtest:
    container_name: vpn_speedtest
    image: henrywhitaker3/speedtest-tracker:dev
    volumes:
        - /opt/vpn_speedtest:/config
    environment:
        - TZ=Asia/Kolkata
        - PGID=1000
        - PUID=1000
        - OOKLA_EULA_GDPR=true
    logging:
        driver: "json-file"
        options:
            max-file: "10"
            max-size: "200k"
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
        
  firefox:
    image: lscr.io/linuxserver/firefox:latest
    container_name: vpn_firefox
    security_opt:
      - seccomp:unconfined #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /opt/vpn_firefox/config:/config
    shm_size: "1gb"
    restart: unless-stopped
    network_mode: "container:gluetun"
    depends_on: # Add this
      gluetun:
        condition: service_healthy
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

#### To Setup QBittorrent Configuration - Create below script

```bash
nano /tmp/qbittorrent-config.sh
```

```bash
#!/bin/bash

# Define ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
RESET='\033[0m'

CONTAINER_NAME="qbittorrent"
CONFIG_FILE_PATH="/opt/qbittorrent/appdata/qBittorrent/qBittorrent.conf"

# Function to print colored messages
print_message() {
  local color="$1"
  local message="$2"
  echo -e "${color}${message}${RESET}"
}

print_message "${YELLOW}" "Editing $CONTAINER_NAME Docker Config File"
# Update the configuration file on the host
if grep -q 'WebUI\HostHeaderValidation=' $CONFIG_FILE_PATH; then
  sed -i 's/WebUI\HostHeaderValidation=.*/WebUI\HostHeaderValidation=false/' $CONFIG_FILE_PATH
else
  echo 'WebUI\HostHeaderValidation=false' >> $CONFIG_FILE_PATH
fi

# Check for WebUI\CSRFProtection and update or add it
if grep -q 'WebUI\CSRFProtection=' $CONFIG_FILE_PATH; then
  sed -i 's/WebUI\CSRFProtection=.*/WebUI\CSRFProtection=false/' $CONFIG_FILE_PATH
else
  echo 'WebUI\CSRFProtection=false' >> $CONFIG_FILE_PATH
fi

print_message "${YELLOW}" "Starting $CONTAINER_NAME Docker Container"
# Restart the container
sudo docker start $CONTAINER_NAME
```

**Add execute permissions:**

```bash
chmod +x /tmp/qbittorrent-config.sh
```

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p arr_stack -f /tmp/arr_stack_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p arr_stack -f /tmp/arr_stack_docker-compose.yml create
```

You should see output indicating that the `nginx` container is being created and started.

**Verify the container is running:**

```bash
docker ps -a
```

```bash
sudo /tmp/qbittorrent-config.sh
```

**Execute the script:**

**Open your web browser:** On your local computer, navigate to:

```bash
http://your_pi_ip_address:11013
```

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/arr_stack_docker-compose.yml
```
