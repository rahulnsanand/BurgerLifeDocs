---
icon: globe
---

# Setup DDNS

**Create the `portainer_docker-compose.yml` file:** Use `nano` to create the file in the `/tmp` directory:

```bash
nano /tmp/ddns_docker-compose.yml
```

**For Hosur - Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3.8'
services:
  ddns_asherslife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_asherslife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=asherslife.in
      - SUBDOMAIN=hosur-vpn
      - PROXIED=false
    restart: always

```

```yaml
version: '3.8'
services:
  ddns_asherslife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_asherslife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=asherslife.in
      - PROXIED=true
    restart: always
    
  ddns_vpn_asherslife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_vpn_asherslife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=asherslife.in
      - SUBDOMAIN=vpn
      - PROXIED=true
    restart: always
    
  ddns_burgerlife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_burgerlife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=burgerlife.in
      - PROXIED=true
    restart: always
    
  ddns_rahulanand:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_rahulanand
    environment:
      - API_KEY=<APIKEY>
      - ZONE=rahulanand.in
      - PROXIED=true
    restart: always

```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Navigate to the `/tmp` directory:**

```bash
cd /tmp
```

**Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -f ddns_docker-compose.yml up -d
```

You should see output indicating that the `portainer` container is being created and started.

**Verify the container is running:**

```bash
docker ps -a
```

You should see `portainer` listed with `Status: Up ...`.

**Remove the temporary Docker Compose file:**&#x42;ash

```bash
rm /tmp/ddns_docker-compose.yml
```

You now have Portainer up and running, providing a powerful GUI for managing your Docker setup!
