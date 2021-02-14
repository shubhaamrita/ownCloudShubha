---
layout: page
title: "Installation with Docker (port 8080)"
permalink: /installationdocker/
---
## Introduction
ownCloud can be installed using Docker, using the [official ownCloud Docker image](https://hub.docker.com/r/owncloud/server/tags). This official image works standalone for a quick evaluation, but is designed to be used in a docker-compose setup.

This document will guide you to enable users to connect to the Owncloud server using the server's IP address and port 8080.

## Pre-requisites

| Platform | Options |
| --------- | --------|
| Operating System | Ubuntu 18.04 LTS |
| Database | MariaDB 10+ |
| Web server | Apache 2.4 with ```prefork``` and ```mod_php``` |
| PHP Runtime | 7.4 |

## Quick Evaluation
```sudo docker run -e OWNCLOUD_DOMAIN=IP Address:8080 -p8080:8080 owncloud/server```
## Docker Compose
### Create a new project directory
```
sudo mkdir owncloud-docker-server
cd owncloud-docker-server
```
### Copy docker-compose.yml from the GitHub repository
```wget https://raw.githubusercontent.com/owncloud/docs/master/modules/admin_manual/examples/installation/docker/docker-compose.yml```

### Create the environment configuration file
```
cat << EOF > .env
OWNCLOUD_VERSION=10.6
OWNCLOUD_DOMAIN=IP Address:8080
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin
HTTP_PORT=8080
EOF
```

### Build and start the container
```sudo docker-compose up –d```
### Check all the containers
```sudo docker-compose ps```
- Database, ownCloud, and Redis containers will be running.
- ownCloud will be accessible via port 8080 on the host machine.
