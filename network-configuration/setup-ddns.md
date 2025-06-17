---
icon: globe
---

# Setup DDNS

**Create the `portainer_docker-compose.yml` file:** Use `nano` to create the file in the `/tmp` directory:

```bash
nano /tmp/ddns_docker-compose.yml
```

**For Hosur - Paste the below Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3.8'
services:
  ddns_asherslife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_hsr_asherslife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=asherslife.in
      - SUBDOMAIN=hsr
      - PROXIED=false
    restart: always

```

**For Bangalore - Paste the below Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3.8'
services:
  ddns_vpn_asherslife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_blr_asherslife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=asherslife.in
      - SUBDOMAIN=blr
      - PROXIED=false
    restart: always   

```

**For Cloud VPS - Paste the below Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
version: '3.8'
services:
  ddns_asherslife:
    image: oznu/cloudflare-ddns:latest
    container_name: ddns_cld_asherslife
    environment:
      - API_KEY=<APIKEY>
      - ZONE=asherslife.in
      - SUBDOMAIN=cld
      - PROXIED=false
    restart: always

```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`

**Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p ddns_asherslife -f /tmp/ddns_docker-compose.yml up -d
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
