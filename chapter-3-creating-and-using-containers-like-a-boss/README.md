# Creating and Using Containers Like a Boss


## Table of Contents
1. [Module introduction](#module-introduction)
1. [Starting a Nginx Web Server](#starting-a-nginx-web-server)

<br/>

## Module Introduction
<br/>

![chapter-3-1.gif](./images/gif/chapter-3-1.gif "Module introduction")
<br/>

### Image vs Containers
<br/>

![chapter-3-2.gif](./images/gif/chapter-3-2.gif "module introduction")
<br/>

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Starting a Nginx Web Server

![chapter-3-3.gif](./images/gif/chapter-3-3.gif "Starting Nginx")
<br/>

If you get an error with your `iptables` like below at first attempt

```bash
$ docker container run --publish 80:80 nginx

$: docker: Error response from daemon: driver failed programming external
connectivity on endpoint crazy_albattani (3960f55d04c8252c8611e549a88f079efbdf8e21c1c079bd5ce0700b34849ed7):
(iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 80 -j DNAT
--to-destination 172.17.0.2:80 ! -i docker0: iptables v1.8.5 (legacy): unknown
option "--dport".)
```
you should setup your `iptables` you should setup your
`iptables-ntf`, [arch-wiki](https://bbs.archlinux.org/viewtopic.php?id=245053)

```bash
$ sudo ln -s /usr/bin/iptables-nft /usr/local/bin/iptables

reboot
```

### Docker --detach

```bash
$: docker container run --publish 80:80 --detach nginx
```
`--detach` is to run docker in background

### Docker stop container

```bash
Usage:  docker container ls [OPTIONS]
List containers
Aliases: ls, ps, list

// @NOTE: docker with name "update-nginx" is running
$: docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
1bdd5223febc        nginx               "/docker-entrypoint.…"   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp   update-nginx
f152412e6ea2        nginx               "/docker-entrypoint.…"   2 hours ago         Created                                  last-nginx
86734c1b6d7c        nginx               "/docker-entrypoint.…"   2 hours ago         Created                                  some-nginx
703dbac10404        nginx               "/docker-entrypoint.…"   2 hours ago         Created                                  keen_noyce


Usage:  docker container stop [OPTIONS] CONTAINER [CONTAINER...]

$: docker container stop 1bdd5223febc
1bdd5223febc

// @NOTE: docker with name "update-nginx" was stopped
$: docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
1bdd5223febc        nginx               "/docker-entrypoint.…"   About an hour ago   Exited (0) 2 minutes ago                       update-nginx
f152412e6ea2        nginx               "/docker-entrypoint.…"   2 hours ago         Created                                        last-nginx
86734c1b6d7c        nginx               "/docker-entrypoint.…"   2 hours ago         Created                                        some-nginx
703dbac10404        nginx               "/docker-entrypoint.…"   2 hours ago         Created                                        keen_noyce

// @NOTE: docker container ls command is only show by defauly running containers
$: docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
```

### Use existing container | run vs start

When we ran each time `docker container run` command, it started a new container
from that nginx image with **random unique** container **names**. If we don't
specify containers names, it will be created for us.

Use `docker container start` command to start an existing stopped container.

```bash
Usage:  docker container start [OPTIONS] CONTAINER [CONTAINER...]

$: docker container start last-nginx
last-nginx

$: docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
f152412e6ea2        nginx               "/docker-entrypoint.…"   2 hours ago         Up 47 seconds       0.0.0.0:80->80/tcp   last-nginx
```

### The convenient way create container

```bash
$: docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]

--publish , -p      Publish a container’s port(s) to the host           | have an arguments
--detach            Run container in background and print container ID  | didnt have any arguments
--name string       Assign a name to the container                      | have an arguments

$: docker container run --publish 90:80 --detach --name webengine nginx
8826d95b2e78eac12fdd2c79c4c0c50b744612cc90b3476f3c1dc7c5ce1ad00b

$: docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
8826d95b2e78        nginx               "/docker-entrypoint.…"   49 seconds ago      Up 46 seconds       0.0.0.0:90->80/tcp   webengine
f152412e6ea2        nginx               "/docker-entrypoint.…"   2 hours ago         Up 15 minutes       0.0.0.0:80->80/tcp   last-nginx
```

### Containers logs

```bash
Usage:  docker container logs [OPTIONS] CONTAINER

$: docker container logs -t webengine
2020-09-13T10:26:13.496050271Z /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
2020-09-13T10:26:13.496118163Z /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
2020-09-13T10:26:13.496123414Z /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
2020-09-13T10:26:13.614655382Z 10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
2020-09-13T10:26:13.645962599Z 10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
2020-09-13T10:26:13.646426133Z /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
2020-09-13T10:26:13.655755695Z /docker-entrypoint.sh: Configuration complete; ready for start up
2020-09-13T11:13:47.401738723Z 172.17.0.1 - - [13/Sep/2020:11:13:47 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:81.0) Gecko/20100101 Firefox/81.0" "-"
2020-09-13T11:13:47.493910932Z 2020/09/13 11:13:47 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:90", referrer: "http://localhost:90/"
2020-09-13T11:13:47.494100910Z 172.17.0.1 - - [13/Sep/2020:11:13:47 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "http://localhost:90/" "Mozilla/5.0 (X11; Linux x86_64; rv:81.0) Gecko/20100101 Firefox/81.0" "-"
```

### Containers top

```bash
Usage:  docker container top CONTAINER [ps OPTIONS]

$: docker container top webegine
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                57326               57309               0                   17:26               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 57373               57326               0                   17:26               ?                   00:00:00            nginx: worker process
```
### Containers remove

```bash
Usage:  docker container rm [OPTIONS] CONTAINER [CONTAINER...]

// Remove running container
$: docker container rm -f webegine last-nginx
```

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>
