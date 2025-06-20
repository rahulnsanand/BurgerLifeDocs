# Table of contents

* [Welcome](README.md)

## Getting Started

* [Quick Reference](getting-started/quick-reference.md)
* [Important Commands](getting-started/important-commands.md)
* [Server Prep Checklist](getting-started/server-prep-checklist.md)
* [Linux Installations](getting-started/linux-installations.md)
* [Docker Containers](getting-started/docker-containers.md)
* [Backup Plan](getting-started/backup-plan/README.md)
  * [Scenario 1 - Normal Usage](getting-started/backup-plan/scenario-1-normal-usage.md)
  * [Scenario 2 - Master Server Down](getting-started/backup-plan/scenario-2-master-server-down.md)
  * [Scenario 3 - Manager Server Down](getting-started/backup-plan/scenario-3-manager-server-down.md)
  * [Scenario 4 - Worker Server Down](getting-started/backup-plan/scenario-4-worker-server-down.md)
  * [Scenario 5 - Bangalore Off Network](getting-started/backup-plan/scenario-5-bangalore-off-network.md)
  * [Scenario 6 - Hosur Off Network](getting-started/backup-plan/scenario-6-hosur-off-network.md)
  * [Scenario 7 - Master NAS HDD Down](getting-started/backup-plan/scenario-7-master-nas-hdd-down.md)
  * [Scenario 8 - Manager NAS HDD Down](getting-started/backup-plan/scenario-8-manager-nas-hdd-down.md)
* [Prepare Debian Server](getting-started/prepare-debian-server/README.md)
  * [Debian OS Setup](getting-started/prepare-debian-server/debian-os-setup.md)
  * [RPi 5 - OS Setup](getting-started/prepare-debian-server/rpi-5-os-setup.md)
  * [RPI Installing Proxmox VE](getting-started/prepare-debian-server/rpi-installing-proxmox-ve.md)
  * ['burger' User Creation](getting-started/prepare-debian-server/burger-user-creation.md)
  * [Auto Mount Drives](getting-started/prepare-debian-server/auto-mount-drives.md)
  * [Essential Packages](getting-started/prepare-debian-server/essential-packages.md)
  * [SSH Setup](getting-started/prepare-debian-server/ssh-setup.md)
  * [BurgerLife Automation Script](getting-started/prepare-debian-server/burgerlife-automation-script.md)
* [Firewall/Network Rules](getting-started/firewall-network-rules.md)

## Docker

* [Docker Setup](docker/docker-setup.md)
* [Docker Swarm Setup](docker/docker-swarm-setup.md)
* [Portainer Setup](docker/portainer-setup.md)

## Borg backup

* [Borg Backup](borg-backup/borg-backup.md)

## Network Configuration

* [Setup DDNS](network-configuration/setup-ddns.md)
* [WireGuard Configuration](network-configuration/openapi.md)
* [Adguard Configuration](network-configuration/openapi-1.md)
* [Cloudflared Configuration](network-configuration/openapi-2.md)

## KeepAlived

* [Configure KeepAlive](keepalived/configure-keepalive.md)

## MariaDB

* [Setup MariaDB](mariadb/setup-mariadb.md)
* [Setup Galera Cluster](mariadb/setup-galera-cluster.md)
* [User burger Creation (P)](mariadb/user-burger-creation-p.md)
* [Restore .SQL Backup](mariadb/restore-.sql-backup.md)

## PostgreSQL

* [Immich PostgreSQL](postgresql/immich-postgresql.md)
* [Authentik PostgreSQL](postgresql/authentik-postgresql.md)
* [Mealie PostgreSQL](postgresql/mealie-postgresql.md)
* [Dev - PostgreSQL Script](postgresql/dev-postgresql-script.md)

## Syncthing

* [SyncThing Configuration](syncthing/syncthing-configuration.md)

## HA Orchestration Setup

* [Application Affected by HA](ha-orchestration-setup/application-affected-by-ha.md)
* [Setup Passwordless SSH](ha-orchestration-setup/setup-passwordless-ssh.md)

***

* [Master Orchestration Scripts](master-orchestration-scripts.md)
* [Manager Orchestration Scripts](manager-orchestration-scripts.md)

## BurgerLife API

* [BurgerLifeAPI Setup](burgerlife-api/burgerlifeapi-setup.md)

## Docker Applications

* [Nginx Proxy Manager  (P)](docker-applications/nginx-proxy-manager-p.md)
* [Immich (P)](docker-applications/immich-p.md)
* [VaultWarden (T|P)](docker-applications/vaultwarden-t-or-p.md)
* [Mealie (T)](docker-applications/mealie-t.md)
* [ARR Stack (T)](docker-applications/arr-stack-t.md)
* [NextCloud](docker-applications/nextcloud.md)
* [File Browser](docker-applications/file-browser.md)
* [Jellyfin](docker-applications/jellyfin.md)
* [Plex](docker-applications/plex.md)
* [Homepage](docker-applications/homepage.md)
* [Firefox](docker-applications/firefox.md)
* [N8N](docker-applications/n8n.md)
* [Glances](docker-applications/glances.md)
