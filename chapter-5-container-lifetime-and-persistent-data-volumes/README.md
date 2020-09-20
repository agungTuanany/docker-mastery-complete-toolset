# Container Lifetime and Persistent Data Volumes

## Table of Contents

1. [Module Introduction](#module-introduction)
2. [Container Lifetime and Persistent Data](#container-lifetime-and-persistent-data)
3. [Persistent Data and Data Volumes](#persistent-data-and-data-volumes)

<br/>

## Module Introduction
<br/>

![chapter-5-1.gif](./images/gif/chapter-5-1.gif "Module introduction")
<br/>

In this section is going yo be a **full of lectures** and **assignments** around
**persistent data**.

First we're going to discuss the **problem of persistent data** and how that
even became a problem in the first place.

**Key concepts** there that you need to understand are the **idea of immutable
infrastructure**; And the idea around containers being **naturally ephemeral
(impermanent) in design**. We'll cover those in first lecture.

Then we dive into **data volumes** and how those solve some of our problems
around _persistent data_.

Then we also dig into **bind mounts** and how that solves different problems
with _persistent data_.

Then we'll end up this section with a couple of _assignments_ around how to
_peresever database data_, while replacing containers for databases; And then
also actually **mounting code into a container from the host** while you're
_editing_ it, so that you can run that code inside the container **live**.


**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Container Lifetime and Persistent Data
<br/>

Requirement for this first lecture are just understanding how to run containers,
and the containers and image concepts we've discuss so far.

![chapter-5-2.gif](./images/gif/chapter-5-2.gif "Cointainer lifetime and persistent data")
<br/>

Until now, you've only worried about what's needed to run a container, not the
unique data that might be created once container is running. Container are
**meant to be immutable and ephemeral**. Those a fancy **buzzwords** for
_unchanging and temporary_, or disposal.

The idea here is that _we can just throw away a container and create a new one
form image_. Then we're not talking about an actual limitation of containers,
but more of a **design goal**, or a best practice.

This the idea of **[immutable
infrastructure](https://www.oreilly.com/radar/an-introduction-to-immutable-infrastructure/)**
where we don't change things once they're running. If a config change needs to
happen, or maybe the container version upgrade needs to happen, then we redeploy
a whole new container. This gives us huge benefits in **reliability and
consistency** and **making change reproducible**. But there's a tradeoff.

What about he unique data that your app will produce? Those database, or a key
value stores, or anything else that your app spits out into a file? Ideally, the
_containers shouldn't contain your unique data mixed in with the application
binaries_. This known as **separation of concerns**.

Docker actually gives us a big benefits with separation of concerns here. We can
update our application by recreating a new container, with an update version of
our app, and ideally, our unique data is still where it needs to be and was
preserved (maintain) for us while our container was recycled. In case you
haven't notice so farm, the containers that we've been using, by default, were
persistent (maintainable). Any changes in them actually were kept across restart
and reboots until we removed the container.

Just because we stopped the container or restarted the host, doesn't mean the
container's file change go away. It's only when we remove the container that
it's [UFS](#what-is-ufs-layers) layer goes away, but we want to be able to do
that at will.

This problem of unique data know in the industry as **persistent data** and
until containers, we didn't really have a name for that. Because, well, most
things we built were persistent, like you know those old servers that have been
running that old app for 5 or 10 year or longer. Everything was persistent by
default.

Now in the new world of containers and application auto scaling, persistent data
creates a unique problem. Docker has two solution to this problem, know as
**data volumes** and **bind mounts**.

**Docker volumes** are a configuration option for a container that create a special
location outside of that container's  Union File System to store unique data.
This preserves it across container removals and allows us to attach it to
whatever container we want; And the container just sees it like a local file
path.

Then there's **bind mounts**, which are simply us sharing or mounting a host
directory, or file, into a container. Again, it will just look like a local file
path, or directory path, to the container. It won't actually know that's it's
coming from the host.


#### What is UFS layers

UFS stands for Union File System, Docker images consist of multiple read-only
file system layers. For each statement in a Dockerfile, a new read-only layer
residing on top of all former layers will be created. If a container is created
based on an image, an additional read/write layer is created on top, which is
then ...
[source](www.dxc.technology/application_services/insights/146173-how_to_get_the_most_out_of_docker_technology)

#### What is persistent data

Persistent data structure is a data structure that always preserves the
previous version of itself when it is modified...
[wiki](wiki2.org/en/Persistent_data_structure)

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Persistent Data and Data Volumes
<br/>

![chapter-5-3.gif](./images/gif/chapter-5-3.gif "Peristent Data: Volumes")
<br/>

Volumes seem pretty simple up front, but there's actually a lot to it. The first
way you can tell a container that it needs to worry about a volume is in
a Dockerfile.

### Command `VOLUME` in image

If we go to official Docker Hub and search MySQL and search the _latest_ we find
[Dockerfile](https://github.com/docker-library/mysql/blob/285fd39122e4b04fbe18e464deb72977df9c6a45/8.0/Dockerfile)
I bet you because it's database, it's probably got a `VOLUME` command in it.

```Dockerfile
VOLUME /var/lib/mysql
```

The `VOLUME` command is the default location of MySQL databases, this image is
programmed in a way to tell Docker when we start a new container from it to
actually create a new _volume location_ and assign it to this directory in this
container. Which means _any files_ that we put in there, in the container, will
outlive the container until we manually delete the volume.

That's something we should point out here is that volumes need manual deletion.
You can't clean them up just by removing a container. They're are _extra step_.
That's just for _insurance really_, because the whole point of the `VOLUME`
command is to say that this data is **particularly important**, at least much
more important than the container itself.

If we do a `docker image inspect` on MySQL, we don't get to see the Dockerfile
because the Docker file **isn't actually part of the image metadata**, which
you'll notice in this config area, that if specified `Volumes` there.

```json
[
    {
        "ContainerConfig": {
            "Hostname": "4a20d34c487d",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {},
                "33060/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.12",
                "MYSQL_MAJOR=8.0",
                "MYSQL_VERSION=8.0.21-1debian10"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"mysqld\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:145f837222dc7be65e5db56e4ab4d4bcf62fed0902d522732101b21ea681b8d4",
            "Volumes": {                            << Volumes value
                "/var/lib/mysql": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        }
    }
]
```

I can always tell that the config, that came from the Dockerfile when it was
built, aassigned a volume to that path.

Let run a container from it with command
<br/>

![chapter-5-4.gif](./images/gif/chapter-5-4.gif "Inspect running container running values")
<br/>

```bash

$: docker container run -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql-inspect mysql
$: docker container inspect mysql-inspect

[
    {
    ...
    ...
        "Mounts": [
            {
                "Type": "volume",
                "Name": "519b150ee8edaec07ca7f5dd2cd7eeffb91c83879ce9c0cb0153f6a5b9175273",
                "Source": "/var/lib/docker/volumes/519b150ee8edaec07ca7f5dd2cd7eeffb91c83879ce9c0cb0153f6a5b9175273/_data",     << data living in host location
                "Destination": "/var/lib/mysql",    << Volumes values
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "7d512d3a9679",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {},
                "33060/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "MYSQL_ALLOW_EMPTY_PASSWORD=true",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.12",
                "MYSQL_MAJOR=8.0",
                "MYSQL_VERSION=8.0.21-1debian10"
            ],
            "Cmd": [
                "mysqld"
            ],
            "Image": "mysql",
            "Volumes": {                            << Volumes values
                "/var/lib/mysql": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        ...
        ...
    }
]
```
<br/>

![chapter-5-5.gif](./images/gif/chapter-5-5.gif "Docker volume command")
<br/>

We should see in the `inspect` command for container that tell us in the
`Config` that there's `Volumes` values, and under `Mounts`. What this is, is
this is actually the **running container getting its own unique location on the
host**, to store data, and then it's in the background, mapped or mounted, to
that location in the container, so that the location in the container actually
just thinks **it's writing** to `/var/lib/mysql` in `Mounts` values.  In this
case, we can see that the data is actually living in that location on the host.


By looking at this, you notice that it's not very user friendly in terms of
telling us what's in it or what this `volumes` in `Mounts` is assigned to. We
can see from the **container's perspective what volume it's using**, but we
can't see from the **volume perspective what it's connected to**.

There's no easy way here to tell one from the other at
`/var/lib/docker/volumes/`. If we deleted the container the data is still safe
in `/var/lib/docker/volumes`. Let's prove it
<br/>

![chapter-5-6.gif](./images/gif/chapter-5-6.gif "Docker volume command")
<br/>

So even the container is deleted, the container `values` are still there and my
data is still safe. So we solved one problem. The databases outlive the
executable. How do we make this a little more user friendly? That's where
**named volumes** come in..

> **NOTE**: named volumes
>
> Friendly way to assign volumes to containers

And the ability for us to specify things on the `docker run` command with `-v`
options for create new container.

```bash
$: docker container run --help
Usage:  docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]

-v, --volume list                    Bind mount a volume
    --volume-driver string           Optional volume driver for the container
    --volumes-from list              Mount volumes from the specified container(s)
```
<br/>

![chapter-5-7.gif](./images/gif/chapter-5-7.gif "docker create volume mount points")
<br/>

A `-v` command allow us to specify either a **new volume we want to create** for
this container that's about to run, or it allow us two other other options
**create a named volume**,

```bash
$: docker container run -v /var/lib/mysql ...
```

This would do the same thing is what our volume command in the Dockerfile did.
We don't really need to do that here. But what we can do we can put a name in
front of it with colon `:`. That know as a **named volume**.

```bash
$: docker container run -v mysql-volumes:/var/lib/mysql ...
```

When I do that, and then do `docker volumes ls`, you'll see that my new
container is using new volumes with friendly name.

### Using existing volume with new container
<br/>

![chapter-5-8.gif](./images/gif/chapter-5-8.gif "Affirmation Mounts volume exist even container delete ")
<br/>

A little tips here is that when I am running for days or weeks at a time,
a particular database, in a particular container that I need to keep using over
and over and I don't want a blank database server. I'll end up creating my
_containers this way and naming_ them for the project so that I know what that
_volume's_ for, and that it needs to stick around.

### Docker volume Create command

Why would you want to do `docker volume create`?

> **NOTE**: `docker volume create`
>
> Required to do this before `docker run` to use customs drivers and labels

If we can create them form a `docker container run` command at runtime, and we
can create them by specifying them in the Dockefile, there's only a few cases
where you'd want to create it ahead of time; And you can figure that out by
pretty quickly by using `--help` command:

```bash
$: docker volume --help

Usage:  docker volume COMMAND
Manage volumes

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes

$: docker volume create --help

Usage:  docker volume create [OPTIONS] [VOLUME]
Create a volume

Options:
  -d, --driver string   Specify volume driver name (default "local")
      --label list      Set metadata for a volume
  -o, --opt map         Set driver specific options (default map[])
```

Because here's the only way that we can actually specify a different driver.
_Remember that _plug-in_  stuff I'm going to talk later? Then any _driver
options_  that we want to use the `-o` command options on. Then if we want to
put labels on it, which we'll also talk about later in the _production section_.

Sometimes, in special cases, you do need to create the Docker `volume` ahead of
time, but usually for your _local development_ purposes, just specifying it in
a Dockerfile or at the `run` command is fine.


**[⬆ back to top](#table-of-contents)**
<br/>
<br/>
