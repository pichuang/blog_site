---
layout: post
title: How do I use docker and docker-compose
date: 2017-03-29 09:34:36
updated: 2017-03-29 09:34:36
tags:
  - infra
  - linux
  - docker
  - docker-compose
categories:
  - container
---

## Objective
Build Bind service with Docker and auto startup it using docker-compose on Ubuntu

<!--more-->

## Environment
- Ubuntu 14.04.5 LTS Server
- docker-compose 1.11.2
- docker 17.03.0-ce

## About `docker-compose`

### Create `docker-compose.yml`
```bash
mkdir -p ~/bind

cat >> ~/bind/docker-compose.yml << EOF
pichuang-dns:
  image: sameersbn/bind:latest
  dns: 127.0.0.1
  environment:
    - ROOT_PASSWORD=pichuang
  ports:
   - 10000:10000
   - 53:53/udp
  volumes:
    - /srv/docker/dns:/data
EOF
```

<!--more-->

### Test
```bash
docker-compose -f ~/bind/docker-compose.yml up -d
docker-compose -f ~/bind/docker-compose.yml ps
```

## About Startup cript

### Create Daemon
```bash
cat >> /etc/init.d/bind << EOF
#!/bin/sh

### BEGIN INIT INFO
# Provides:             bind
# Required-Start:       $docker
# Required-Stop:        $docker
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Docker Services
### END INIT INFO

set -e

PROJECT_NAME=bind
YAMLFILE=/home/pichuang/docker-compose.yml

DOCKER_COMPOSE_BIN=$(whereis docker-compose|awk '{print $2}')
OPTS="-f $YAMLFILE -p $PROJECT_NAME"
UPOPTS="-d"

. /lib/lsb/init-functions

case "$1" in
    start)
        log_daemon_msg "Starting Docker Compose" "bind" || true
        $DOCKER_COMPOSE_BIN $OPTS up $UPOPTS
        ;;

    stop)
        log_daemon_msg "Stopping Docker Compose" "bind" || true
        $DOCKER_COMPOSE_BIN $OPTS stop
        ;;

    reload)
        log_daemon_msg "Reloading Docker Compose" "bind" || true
        $DOCKER_COMPOSE_BIN $OPTS up $UPOPTS
        ;;

    restart)
        $DOCKER_COMPOSE_BIN $OPTS stop
        $DOCKER_COMPOSE_BIN $OPTS up $UPOPTS
        ;;

    status)
        $DOCKER_COMPOSE_BIN $OPTS ps
        ;;

    *)
        log_action_msg "Usage: /etc/init.d/bind {start|stop|restart|reload|status}" || true
        exit 1
        ;;
esac

exit 0
EOF

chmod +x /etc/init.d/bind
```

### Update rcx.d
```bash
cd /etc/init.d/
update-rc.d -f bind defaults 80 20
```

### Test
```bash
reboot
...
service bind status
service bind stop
service bind start
service bind reload
service bind restart
```
