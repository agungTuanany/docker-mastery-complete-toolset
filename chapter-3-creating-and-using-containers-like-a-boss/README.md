# Creating and Using Containers Like a Boss


## Table of Contents
1. [Module introduction](#module-introduction)
2. [Starting a Nginx Web Server](#starting-a-nginx-web-server)
3. [Debrief What Happens When We Run a Container](#debrief-what-happens-when-we-run-a-container)
4. [What is Going On In Containers CLI Process Monitoring](#what-is-going-on-in-containers-cli-process-monitoring)
5. [Getting a Shell Inside Container](#getting-a-shell-inside-container)
6. [Docker Networks Concepts for private and Public Communication in Container](#docker-networks-concepts-for-private-and-public-communications-in-container)

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
you should setup your `iptables` and `iptables-ntf`, [arch-wiki](https://bbs.archlinux.org/viewtopic.php?id=245053)

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

## Debrief What Happens When We Run a Container
<br/>

![chapter-3-4.gif](./images/gif/chapter-3-4.gif "docker run behind the scenes")
<br/>

Let's have a quick discussion about what actually happens in the background when
we run the Docker. There's a misconception that Docker is really just running
containers and that's its. But there actually so much that's happening in the
background that it does in additions to those containers.

When we type `docker container run`, in the background it's actually going to
look for the `image` that specified at the end of that command. So you remember
when you type `nginx` at the end, that was the name of the `image` we wanted to
to run as a **new** container. It's going to look for that locally in the `image
cache`.

If it doesn't exists, it's going to hop (jump) over to hub.docker.com, which
is its default remote image repository.

By default, it'll look it in hub.docker.com and download it and store it in the
image cache `/var/lib/docker`

```bash
$: sudo ls -la /var/lib/docker/containers/
total 12
drwx------  3 root root 4096 Sep 13 20:46 .
drwx--x--x 15 root root 4096 Sep 13 15:20 ..
drwx------  4 root root 4096 Sep 13 20:47
152cfe030a5f75f42c646872452ff5ed7c97dbf038fbaed1ec53c96b3913b6e3

// @NOTE container ID
$: docker container ls -la
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
152cfe030a5f        nginx               "/docker-entrypoint.…"   4 hours ago         Up 29 minutes       0.0.0.0:80->80/tcp   webgate
```

So, if we didn't specify a `version` and we didn't, we just typed `nginx`, you
can type `nginx:version`, it'll just choose the latest. Once it's got the image
and ready to go. It's going to start up (create) a new container based on that
image.

It's not going to make a copy of the image. It's actually going just to start
a **new layer** of changes, right on top of where the image left of
`/var/lib/docker/`, and it's going to customize the networking.

It's going to give a specific `virtual ip address` that's inside a _docker
virtual network_.

It's actually going to open up the port that we specified. If we didn't
specify the `--publish` command, it's **not** going to open any ports at all.
Since we did with `80:80`, that's telling it to take the **port 80** on the host
and forward all that traffic to the **port 80** in the container. Then container
finally will actually start using a command specified in the DockerFile, which
will also talk about that in next chapter.

### Example of changing the default images version
<br/>

![chapter-3-1.png](./images/chapter-3-1.png "Example changing the defaults images")
<br>

So you can actually change a majority of above command from CLI. Above command
is a convention command (defaults). We can specify the **version** of image
`nginx:1.11` specify different port `8080:80`,  or the default command to run when it
starts `nginx -T`.

Since we had a very simple command, it just used a lot of defaults coming out of
the box.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## What is Going On In Containers CLI Process Monitoring
<br/>

![chapter-3-5.gif](./images/gif/chapter-3-5.gif "Docker CLI process monitoring")
<br/>

### Docker top
<br/>

![chapter-3-6.gif](./images/gif/chapter-3-6.gif "Docker CLI process monitoring with top")
<br/>

### Docker inspect
<br/>

![chapter-3-7.gif](./images/gif/chapter-3-7.gif "Docker CLI process monitoring with inspect")
<br/>

### Docker stats
<br/>

![chapter-3-8.gif](./images/gif/chapter-3-8.gif "Docker CLI process monitoring with stats")
<br/>

### Docker logs
<br/>

![chapter-3-9.gif](./images/gif/chapter-3-9.gif "Docker MySQL access random password")
<br/>

We have create container named `mysql-dev` with parameter `--env
MYSQL_RANDOM_ROOT_PASWORD=true` for access MySQL database in next future.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Getting a Shell Inside Container
<br/>

![chapter-3-10.gif](./images/gif/chapter-3-10.gif "Getting a shell inside container")
<br/>

### How `-it` works in Docker

Looking the container from the outside is very handy (easy) at times; But how we
can get **into** the container, to look up what is the files or folders inside the
container?, the most often question to ask is **how do I get into a container?**
and do things in it live from the command line.

Sometimes people confuse this with wanting an **SSH server** inside their
container so the can `ssh` into it. In Docker we don't actually need to do that
because we have several commands at our disposal, that let us get a shell inside
the container itself while it's running.

Docker itself have `docker cli` substitute for adding ssh for adding SSH to
containers.

```bash
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

-t, --tty                            Allocate a pseudo-TTY
-i, --interactive                    Keep STDIN open even if not attached

// @NOTE: [COMMAND] after image
$: docker container run -it --name nginx-bash nginx bash
```

![chapter-3-11.gif](./images/gif/chapter-3-11.gif "How '-it' works in Docker")
<br/>

When we exit the shell, the container stopped. Because containers only run as
long as the command that it ran on startup runs. Since we changed it to `bash`
simply exiting `bash` quit the container.

### How to run existing container interactively

```bash
Usage:  docker container start [OPTIONS] CONTAINER [CONTAINER...]
Start one or more stopped containers

-a, --attach                  Attach STDOUT/STDERR and forward signals
-i, --interactive             Attach container's STDIN


$: docker container start -ai nginx-bash
```

![chapter-3-12.gif](./images/gif/chapter-3-12.gif "How to run existing container interactively")
<br/>

### Using full distribution Linux OS
<br/>

![chapter-3-13.gif](./images/gif/chapter-3-13.gif "Using full distribution Linux OS")
<br/>

### How `exec` works in Docker

`docker container exec` run a command in a **running (existing) container** as
a second process.

```bash
Usage:  docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
Run a command in a running container

Options:
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY

// @NOTE: [COMMAND] after image
$: docker container exec --it mysql-dev bash
```
The `exec` command is very useful for jumping into container when you need to
troubleshoot or when you need to change slightly on a running system, as well as
using container of different distribution to give you the environment you would
have, if you had a full machine (OS image) like `arch` or an `alpine`.

![chapter-3-14.gif](./images/gif/chapter-3-14.gif "Docker use 'exec' in a running container")
<br/>

### Different Linux distro in containers

#### What is `alpine`

`alpine` is another distribution of Linux, but Alpine is designed to be very
small. [alpine](https://www.alpinelinux.org/)

The neat thing about Alpine it also come with **own package manager**
named **apk**. Alpine it doesn't come with `bash` if you run a command

```bash
$: docker container run -it alpine bash
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370:
starting container process caused: exec: "bash": executable file not found in $PATH: unknown.
ERRO[0004] error waiting for container: context canceled
```

You can replace a command with `sh`

```bash
$: docker container run -it alpine sh
```

![chapter-3-15.gif](./images/gif/chapter-3-15.gif "Docker different Linux distro in containers")
<br/>

### What is TTY

`tty` is a small program written **C** simulates a real terminal, like what `SSH`
does; and the real definition in Unix system is `tty` - print the file name of
the terminal connected to standard input

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Docker Networks Concepts for Private and Public Communication in Container
<br/>

![chapter-3-16.gif](./images/gif/chapter-3-16.gif "Docker Network: Concepts")
<br/>

Requirement for this section include understanding how to **start** a container.
Then basick **TCP/IP** (Transmission Control Protocol) networking concepts, such
as **Subnets**, **IPs**, **Port**, and **Firewalls**. You don't have to know
that, but it'll definitely make it a lot easier for you to understand this
section which is focused on _conceptual_ stuff, before we actually get into
a bunch of the command line stuff in the next lecture.

I just want to remind you`-p` (publish) command  on your container run commands
which expose the **port** on your machine. There's actually a lot more going on
in the background of Docker that we'll talk about.

Docker has this concept of `batteries` included but removable which basically
means that the defaults are pretty easy and common to work with, but that you
can change a lot of the options under the hood.

We'll quickly check out in this lecture the **container port command** that
gives you a quick output of what _port_ are _open_ for that container on your
network.

Then we gonna break down some of the concepts of **Docker Networking**,
**Virtual Network** and how **packets flow** around the network.

We finished up with network diagram showing how **containers talk among each
other** and how that's different from the `-p` of exposing them onto the
physical network.

### Docker Network Defaults
<br/>

![chapter-3-17.gif](./images/gif/chapter-3-17.gif "Docker Network Defaults")
<br/>

When you actually start a container, you're really in the background connecting
into particular Docker network. By default, that is the **bridge network**, as
we'll check out in a minute.

Then each one of those networks that you would connect to actually _route out_
through **NAT firewall**, which actually the Docker daemon configuring the
**host IP** address on its defaults interface so that your containers can get
out to the internet or the rest of your network and then get back.

We don't actually need to use the `-p` when we have specific containers wanting
to talk to each other inside our host.

For example, if you had an application that had a `SQL server`, and `PHP-Apache`
container, those two containers should be on the same network and they're able
to talk to each other without actually _opening their ports up_ to the rest of
your physical network.

If you have another application that was unrelated, that let's say was using
MongoDB and NodeJS, you could create a network for that so that they could talk
with each other without using the `-p` to expose them to the network; But they
couldn't actually talk to the other network where you might have an unrelated
app running.

### Docker Network Container

![chapter-3-18.gif](./images/gif/chapter-3-18.gif "Docker Network Cont")
<br/>

The things is that just about all those settings I described are actually
changeable; And this brings up a _good saying_ that Docker likes to use
`batteries` included but removable.

You'll see that throughout this course that a lot of times there are defaults
that just work out of the box. You don't even have to specify them. You just
notice that things are configured in a short standard way; But a lot of
`batteries` options are _configurable_ either at **runtime** or **changeable
after the fact**; And we'll see some of that take place here when we play around
with networks.

Some of the things that you can actually change would be creating **multiple
virtual network**, maybe one per app, or different ones based on different
security requirements.

You can actually, just like in the physical world, have two physical **NICs** on
a real computer. You can actually have two virtual networks connected to one
container, or you can actually have the container talk to no networks.

You can skip any the virtual network configuration that comes out of the box and
actually use `--net=host`; and you'll lose some of the containerization
benefits; But in some cases it might be required.

Later on, we're actually going to get into **Docker network drivers**.

There's this whole plugin ecosystem around Docker that extends the capabilities
of Docker to a lot of third party tools. But in this case we're going to look at
a couple of different **Docker network drivers** and how they might change up
our networking and give us new abilities.

This is really just scratching the surface. There's actually networking
concepts throughout this course where we'll be talking about more advanced topic
of **multi host private networking** and concepts like **sub interfaces** and so
on.

### Command Line Stuff Docker Network
<br/>

![chapter-3-19.gif](./images/gif/chapter-3-19.gif "Comand line stuff Docker Network")
<br/>

Create a new Nginx container named `ningx-network`

```bash
$: docker container run --publish 80:80 --name nginx-network --detach nginx
```

Check which ports are forwarding traffic to `nginx-network` container from the
host (server) into container itself

```bash
$: docker container port nginx-network
80/tcp -> 0.0.0.0:80
```

We haven't talked about the `IP address` of the container; You might just assume
that the `nginx-network` container is using same `IP` as the host (server); But
by default, that's not true. We can easily get the Docker `IP` of that container
by inspecting it and we're going to use `inspect` with `--format` option using
`GO` templates

```bash
Usage:  docker container inspect [OPTIONS] CONTAINER [CONTAINER...]
Display detailed information on one or more containers

Options:
-f, --format string   Format the output using the given Go template
-s, --size            Display total file sizes

$: docker container inspect --format '{{ .NetworkSettings.IPAddress  }}' nginx-network
172.17.0.2
```

The `--format`, it's actually a little cleaner and consistent rather then use
`grep`.

Lets check our home network `IP address` and compare with `IP address` of
container `nginx-network`

```bash
$: ip add

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:60:6e:e7:16:33 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.102/24 brd 192.168.0.255 scope global dynamic noprefixroute enp3s0                << Host (server) IP
       valid_lft 74298sec preferred_lft 74298sec
    inet6 fe80::1cb8:c70c:b78d:dcf3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:a5:68:3f:31 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0                                      << Docker has create own IP
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a5ff:fe68:3f31/64 scope link
       valid_lft forever preferred_lft forever
```


### Miscellaneous

#### What is NAT

NAT is **Network Address Translation**, is a method of remapping `IP address`
space into another by modifying network address `IP header` of packets while
they are in transit across a traffic routing device.
[wiki](https://en.wikipedia.org/wiki/Network_address_translation)


#### What is TCP/IP

Short for transmission control protocol/Internet protocol, TCP/IP is a set of
rules governing communications among all computers on the Internet.More
specifically, TCP/IP dictates how information should be packaged (turned into
bundles of information called packets), sent, and received, and how to get to
its destination.TCP/IP was developed in 1978 and driven by Bob Kahn and Vint
Cerf. [computerhope](https://www.computerhope.com/jargon/t/tcpip.htm)

#### What is Subnet

Subnet abbreviate Subnetwork, is a logical subdivision of an `IP network`. The
practice of dividing a network into or more networks.
[wiki](https://en.wikipedia.org/wiki/Subnetwork)

#### What is Firewall

In computing, a firewall is a network security system that monitors and controls
and outgoing network traffic on predetermined security rules. A firewall
typically establishes a barrier between a trusted network and a untrusted
network. [wiki](https://en.wikipedia.org/wiki/Firewall_(computing))

#### What is NICs

Is abbreviate Network Interface Controller, is  a hardware component without NIC
a computer cannot be connected over a network.
[wiki](https://en.wikipedia.org/wiki/Network_interface_controller)

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>
