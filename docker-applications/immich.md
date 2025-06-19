---
icon: aperture
---

# Immich

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/immich_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
name: immich
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    # extends:
    #   file: hwaccel.transcoding.yml
    #   service: cpu # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
      - /mnt/primary/nextcloud/ncdata/vaishnavee/files:/mnt/primary/nextcloud/ncdata/vaishnavee/files
      - /mnt/primary/nextcloud/ncdata/Swastika/files:/mnt/primary/nextcloud/ncdata/Swastika/files
      - /mnt/primary/nextcloud/ncdata/rahul/files:/mnt/primary/nextcloud/ncdata/rahul/files
      - /mnt/primary/nextcloud/ncdata/nas/files:/mnt/primary/nextcloud/ncdata/nas/files
    env_file:
      - .env
    ports:
      - 11024:2283
    depends_on:
      - redis
      #- database
    restart: on-failure

  immich-machine-learning:
    container_name: immich_machine_learning
    # For hardware acceleration, add one of -[armnn, cuda, openvino] to the image tag.
    # Example tag: ${IMMICH_VERSION:-release}-cuda
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    # extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
    #   file: hwaccel.ml.yml
    #   service: cpu # set to one of [armnn, cuda, openvino, openvino-wsl] for accelerated inference - use the `-wsl` version for WSL2 where applicable
    volumes:
      - /opt/immich/machine_learning/cache:/cache
    env_file:
      - .env
    restart: unless-stopped

  redis:
    container_name: immich_redis
    image: docker.io/redis:6.2-alpine@sha256:e3b17ba9479deec4b7d1eeec1548a253acc5374d68d3b27937fcfe4df8d18c7e
    volumes:
      - /opt/immich/redis:/data
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped
    
  backup:
    container_name: immich_db_dumper
    image: prodrigestivill/postgres-backup-local:16
    restart: always
    env_file:
      - .env
    environment:
      BACKUP_ON_START: TRUE
      POSTGRES_HOST: ${DB_HOSTNAME}
      POSTGRES_CLUSTER: 'TRUE'
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      SCHEDULE: "0 3,17 * * *"
      POSTGRES_EXTRA_OPTS: '--clean --if-exists'
      BACKUP_DIR: /db_dumps
      WEBHOOK_ERROR_URL: http://10.21.22.5:6719/notify/immich_pg_dump_error
      WEBHOOK_POST_BACKUP_URL: http://10.21.22.5:6719/notify/immich_pg_dump_success
      TZ: Asia/Kolkata
    volumes:
      - /mnt/backup/postgres/immich/db:/db_dumps
      - /opt/immich_db_dumper/backups:/backups
      - /opt/immich_db_dumper/data:/var/lib/postgresql/data
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

**Create the `.env`file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/.env
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```bash
UPLOAD_LOCATION=/mnt/primary/gallery
DB_DATA_LOCATION=/opt/immich/postgres
DB_URL='postgresql://postgres:<Password>@10.10.10.10:5432/immichdb'

TZ=Asia/Kolkata

IMMICH_VERSION=v1.134.0

DB_PASSWORD=<Password>
DB_USERNAME=postgres
DB_DATABASE_NAME=immichdb
DB_HOSTNAME=10.10.10.10

```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p immich -f /tmp/immich_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p immich -f /tmp/immich_docker-compose.yml create
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
rm /tmp/immich_docker-compose.yml
```
