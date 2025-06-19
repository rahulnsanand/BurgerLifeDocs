---
icon: bowl-spoon
---

# Mealie

**Create the `docker-compose.yml` file:** We'll use `nano` to create the file in the `/tmp` directory.

```bash
nano /tmp/mealie_docker-compose.yml
```

**Paste the Docker Compose content:** Copy the following content and paste it into the `nano` editor.

```yaml
services:
  mealie:
    image: ghcr.io/mealie-recipes/mealie:latest
    container_name: mealie
    restart: unless-stopped
    ports:
        - "11020:9000"
    deploy:
      resources:
        limits:
          memory: 1000M
    volumes:
      - /opt/mealie/:/app/data/
    environment:
      ALLOW_SIGNUP: false
      PUID: 1000
      PGID: 1000
      TZ: Asia/Kolkata
      MAX_WORKERS: 1
      WEB_CONCURRENCY: 1
      BASE_URL: https://food.asherslife.in
      SMTP_HOST: smtpout.secureserver.net
      SMTP_PORT: 465
      SMTP_FROM_NAME: BurgerLife Mealie
      SMTP_AUTH_STRATEGY: SSL
      SMTP_FROM_EMAIL: admin@burgerlife.in
      SMTP_USER: admin@burgerlife.in
      SMTP_PASSWORD: RahSwas@2206
      OIDC_AUTH_ENABLED: true
      OIDC_SIGNUP_ENABLED: true
      OIDC_CONFIGURATION_URL: https://auth.asherslife.in/application/o/mealie/.well-known/openid-configuration
      OIDC_CLIENT_ID: NllUJx9ADlTUWJv19mNT6vGu5r6AA6SpLGwrBgTE
      OIDC_CLIENT_SECRET: mUunmwH25KHwSzyLOtes0rkHQM7uH8lh1UHItroGu27hyEKl5UgU13M7VrMR570x0BYjtuke9ZGXZQ3NteAT5B3S3nCO7P2JDpGLml05MEP1jr3FTOK8I6OUFfMF7Cfp
      OIDC_AUTO_REDIRECT: false
      OIDC_PROVIDER_NAME: SSO
      OIDC_REMEMBER_ME: true
      OIDC_SIGNING_ALGORITHM: RS256
      OIDC_USER_CLAIM: preferred_username
```

**Save the file and exit nano:**

* Press `Ctrl` + `O`
* Press `Enter`
* Press `Ctrl` + `X`&#x20;

<mark style="color:green;">**\[MASTER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p mealie -f /tmp/mealie_docker-compose.yml up -d
```

You should see output indicating that the `nginx` container is being created and started.

<mark style="color:orange;">**\[MANAGER]**</mark>**&#x20;Start the Docker container using Docker Compose:** The `-f` flag specifies the path to your Docker Compose file. The `-d` flag runs it in detached mode (in the background).

```bash
docker-compose -p mealie -f /tmp/mealie_docker-compose.yml create
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
rm /tmp/mealie_docker-compose.yml
```
