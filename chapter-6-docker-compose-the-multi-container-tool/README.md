# Docker Compose The Multi-Container Tool

## Table of Contents

1. [Module Introduction](#module-introduction)
2. [Docker Compose and the docker-compose.yml](#docker-compose-and-the-docker-compose.yml)

<br/>

## Module Introduction

This section is going to focus exclusively on **Docker Compose**, which is
a combination of a _command line tool_ and a _configuration file_. Requirement
for this section are that you really need to **know a lot of the fundamentals**
from the last few section around _images_ and _containers_.

As we're going through this course, you're probably going to start to hear me
get more and more excited because things just getting better.

As you learn one component, you're thinking, 'man, bind **mounts** and
**volumes** are really cool'. That's going to save me some time. I can imagine
how I'm going to use that. Then I'm going to come with the next thing that's
going to be even better that what you just learned. In this case, it couldn't be
more true.

## Docker Compose and the docker-compose.yml

### What is  Docker Compose
<br/>

![chapter-6-1.gif](./images/gif/chapter-6-1.gif "Docker compose")
<br/>

What is Docker Compose? And why do we care? So few software services are truly
standalone. When you think about your containers, they're a single process
solution, and we're _rarely going to use just a single container_ to solve
a problem or provide a service to our customer.  Our containers will often
require other containers such a SQL or a key value; And other applications that
we'll need to run in containers, like _proxies_ or _web frontends_ or _backend
workers_, and so on.

What if we had a way to connect all those pieces of our solution together, where
we didn't need to remember all of our`docker run` options, and then get them
_into discreet_, _virtual networks with relationship between them_, and only _expose
the public ports_ and then spin them all up and tear them down with one command.

Well, that's Docker Compose.

Before we dive in, we need to clear up that there's **two parts to Docker
Compose**.  The first part is the YAML file, and it's super simple to
understand. It's written in YAML, and if you've never dealt with YAML, that is
a very easy _configuration language_. It's almost as easy as an INI file would
be.  I actually think it's easier because it show hierarchy. We'll dive into
that.  That file _is where you would specify all the containers you need yo
run_, the _networks_ you need, any _volumes_ you might need, _environment
variables_, _images_, and all sorts of other configuration options.

Then the second part of Docker Compose is **CLI tool**, which is
`docker-compose`, that we use normally for just local dev and test, using that
YAML file we created to simplify our Docker commands.

### What is docker-compose.yml
<br/>

![chapter-6-2.gif](./images/gif/chapter-6-2.gif "Docker compose")
<br/>

There's actually versions to the things that you can put in the compose file. So
when Docker Compose was created years ago, it was actually called **Fig** and it
was just assumed `version 1`. It didn't actually even list a version in the
file. As it's matured and added new features to what the file can have in it,
such as _networks_ and _volumes_, they have **created the version statement**,
which is actually the first line in the file.

This file can actually be used with a Docker Compose CLI, which we'll talk about
in a minute. This is, again, mainly for local development management and just
making it easier to get around in your environments on you local machine (host).

Now, starting with the beginning 2017, we have `version 1.13` and anything
beyond that, these files can now be used directly with the docker command line
in production with Swarm. Of course, there's great help at the command line with
the `--help`.

As w look at some of the examples, know that the Docker Compose file is
_a default name_, but you can actually use any file name you want, as long as
it's proper YAML and that you use the `-f` to specify which file you're trying
to use.

#### YAML indentation

Synopsis of YAML Basic Elements. The synopsis of YAML basic elements is given
here: Comments in YAML begins with the (#) character. Comments must be
separated from other tokens by whitespaces. Indentation of whitespace is used
to denote structure. Tabs are not included as indentation for YAML files. List
members are denoted by a leading hyphen (-).
[source](http://www.tutorialspoint.com/yaml/yaml_basics.html)

#### YAML: `version`

[template.yml](./compose-sample-1/template.yml)

```yaml

version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum

services:  # containers. same as docker run
    servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```

We have the `version` value in the beginning. The newest version in first
quarter of 2017 is `3.1` and now in 2020 is `3.8` for Docker Engine `19.03.0+`.
But I always make it at least version 2. If you don;t add that line there, it's
always assumed to be `version 1`. But I don't recommend that because you lose
a lot of features. [source](https://docs.docker.com/compose/compose-file/)

So I typically start with `version 2`. Then if I need specific features out of
`version 3` and `3.1`, I end up typing the version number i there. You'll see
the other three main section are `services`, `volumes` and `networks`.

#### YAML: `services`

YAML is hierarchical. Under `services`, everything can be **two** or **four
spaces**, but it does need to be consistent. If you just look up the YAML on the
internet, you'll find some sites that guide you with the specific of how to
format this file.

If you using a modern editor, it's going to actually speak of YAML language and
know how to deal with these file and their formatting. Usually here, you have,
at minimum, `services`, which is really just **containers**. The reason they
actually call them `services` is because each container that you create in here,
you could actually have multiple ones of those containers for redundancy. So
they needed to come up with a different word. I thinks `services` is great
because that's basically what you're doing. Right?  Each container, or
containers, that are identical, that you're using are usually providing
a service to something.

#### YAML: `servicename`

Underneath the `servicename`, and again you can call it whatever you want. It
doesn't have to be the name of the image or it doesn't have to be anything
related at all. It could be your name for the `servicename`, but it _will be the
DNS name_, that we'll find out later, is used inside of your _Docker networks_.
Similar to when you give the `--name` to a `docker run` command.

At `servicename`, I've shown that we can _optionally_ specify the `image`. We can
specify the _alternate_ `command` to run. If we wanted to overwrite the actual
command that was specified in the image when we run it, we can do that.

Basically, all the things we do from the `docker run` command, we can save them
here. Because that's what really this about, is taking all of the work out of
remembering all the different things in `docker run` command for those things
you're running constantly.

If you have your own development environment, or your own tools, you probably
don't want to type those commands over and over; And shell scripts can only do
so much. Right? So this file would actually replace a shell script that would
automate your `docker run` commands. This is a much better way to do it. It's
easier to read and it's better documented.


#### YAML: spaces
You'll notice that spaces are allowed in YAML. You can have spaces in comments
wherever you want. If you have multiple services, you just need to make sure
they're _unique names_.

#### YAML: `volumes` and `networks`

The other part that we specify are `volumes` and `networks`; And again, these
are _optional_ as well. It's basically same rules as when you're running from he
command line. If you were to ever need to use the `docker volume create` or the
`docker network create` commands, and again, you don't always have to. It just
depends on your situation. You can put them here as well.

### YAML Real World Simple example 1

[docker-compose.yaml](./compose-sample-1/docker-compose.yml)

```yaml
version: '2'

# same as
# docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - '80:4000'
```

You remember in the last section, we had a lecture we used **Jekyll**  really
quick to just do a _bind mount_ example. So this is what the compose file would
look like for that `docker run` command, and you'll see that i have specified
the `image: bretfisher/jekyll-serve`, I've named it `jekyll`. I've given it
`volumes`. You'll notice instead of using the `pwd`, Compose actually understand
that `.` in `- .:/site`, meaning this is the current working directory I'm in.
Just use this directory that I'm running the compose file from. Then the `ports`
or the `-p` option.

You'll notice some little things about YAML, where when something only has one
option, like if there's only going to be _one_ `image` for each service, it
would usually be a **key** and **value** format. If it's going to be a list of
_items_, like `volumes` and `ports` where you can have multiple ones, you'll
notice that the compose file usually has a _plural_. Instead of `volume`, it's
`volumes`; And instead of `port`, it's `ports`; And then there'll be a list
format with a `-`.

### YAML Real World Simple example 2

[compose-2](./compose-sample-1/compose-2.yml)

```yaml
version: '2'

services:

    wordpress:
        image: wordpress
        ports:
              - 8080:80
        environment:
            WORDPRESS_DB_HOST: mysql
            WORDPRESS_DB_NAME: wordpress
            WORDPRESS_DB_USER: example
            WORDPRESS_DB_PASSWORD: examplePW
    volumes:
      - ./wordpress-data:/var/www/html

  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: examplerootPW
      MYSQL_DATABASE: wordpress
      MYSQL_USER: example
      MYSQL_PASSWORD: examplePW
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```
We jump into another example real quick. You'll see that in this one, this is
actually a sample WordPress setup where you hav the _database service_ down here
that's running a database container named `mysql` with image `mariadb` and then
the WordPress _web server_ container up top.

You'll notice a little thing that's different about this one is that we're using
the `environment` variables, which would be the same as a `-e` at the command
line. But instead of a list format where we have a`-`, this is, again, another
key and value statement. In this format we don't use the `-`, we _just list the
key and the value_. If I needed to add another one, I would just add below.

```yaml
version: '2'

services:

    wordpress:
    ...
    ...
    environment:
    ....
    ....
            WORDPRESS_DB_PASSWORD: examplePW
            key: value
```

All of these would be passed into the container, when it runs, using the compose
command line.

### YAML Real World Simple example 3

[compose-3.yml](./compose-sample-1/compose-3.yml)

```yaml
version: '3'

services:
    ghost:
    image: ghost
    ports:
        - "80:2368"
    environment:
        - URL=http://localhost
        - NODE_ENV=production
        - MYSQL_HOST=mysql-primary
        - MYSQL_PASSWORD=mypass
        - MYSQL_DATABASE=ghost
    volumes:
        - ./config.js:/var/lib/ghost/config.js
    depends_on:
        - mysql-primary
        - mysql-secondary
    proxysql:
    image: percona/proxysql
    environment:
        - CLUSTER_NAME=mycluster
        - CLUSTER_JOIN=mysql-primary,mysql-secondary
        - MYSQL_ROOT_PASSWORD=mypass

        - MYSQL_PROXY_USER=proxyuser
        - MYSQL_PROXY_PASSWORD=s3cret
    mysql-primary:
    image: percona/percona-xtradb-cluster:5.7
    environment:
        - CLUSTER_NAME=mycluster
        - MYSQL_ROOT_PASSWORD=mypass
        - MYSQL_DATABASE=ghost
        - MYSQL_PROXY_USER=proxyuser
        - MYSQL_PROXY_PASSWORD=s3cret
    mysql-secondary:
    image: percona/percona-xtradb-cluster:5.7
    environment:
        - CLUSTER_NAME=mycluster
        - MYSQL_ROOT_PASSWORD=mypass

        - CLUSTER_JOIN=mysql-primary
        - MYSQL_PROXY_USER=proxyuser
        - MYSQL_PROXY_PASSWORD=s3cret
    depends_on:
        - mysql-primary
```

The last example. This one is a little more complicated. It's actually one
I was working on recently to set up a three database server cluster behind
a _Ghost web server_. Ghost is actually a blog system similar to WordPress. You'll
actually see that I have multiple `environment` variables for each one of these
containers.

I've got my Ghost container up top. Then I actually have this thing known as
a _SQL proxy_, which it sits in front of _MySQL servers_, and acts as a _load
balancer_ and _failover solution_.

Then I have each of my two, _MySQL servers_. You'll notice another option down
that says `depends_on`, and `depends_on` is a pretty common one. It basically
helps Compose understand the relationship between services. It known that if
I need to start my _Ghost web server_, then I also need to start other ones as
well.

You can see that my _Ghost web server_ actually depends on _mysql-primary_ and
_mysql-secondary_. We'll get into the detail of how this works in later
lectures.


But for now, this is probably a lot of digest, and all these different commands
and whatnot, but trust that Docker has some really great documentation on their
website. If you just look for Docker Compose file on duckduckgo, you'll probably
find under docs.docker.com/compose-file website as usual, and it's going to be
documentation about every single little key and value option you have.

What's great about Docker documentation is it gives you plenty of examples.
**Personally I'm on this site every day**, just like I'm on the `docker run`
command options page or the `compose` command options page, I'm constantly
referencing these because there's just frankly so many features nowadays that
I can't remember them all.

Now that we have a general idea of what the compose file format might look
like, let's jump over to the command line and learn about Docker Compose CLI
tool.

### Miscellaneous

#### What is YAML

YAML (a recursive acronym for "YAML Ain't Markup Language") is a human-readable
data-serialization language.It is commonly used for configuration files and in
applications where data is being stored or transmitted. YAML targets many of
the same communications applications as Extensible Markup Language (XML) but
has a minimal syntax which intentionally differs from SGML.
[wiki](http://en.wikipedia.org/wiki/YAML)

YAML is a data serialisation language designed to be directly writable and
readable by humans. It's a strict superset of JSON, with the addition of
syntactically significant newlines and indentation, like Python. Unlike Python,
however, YAML doesn't allow literal tab characters for indentation.---
[source](http://learnxinyminutes.com/docs/yaml/)

#### What is INI

An INI file is a configuration file for computer software that consists of
a text-based content with a structure and syntax comprising key-value pairs for
properties, and sections that organize the properties. The name of these
configuration files comes from the filename extension INI, for initialization,
used in the MS-DOS operating system which popularized this method of software
configuration.  [wiki](http://en.wikipedia.org/wiki/INI_file)

An INI file is a Windows Initialization file, used most often by software
programs. These are plain text files that contain settings dictating how
programs should operate.
[source](http://www.lifewire.com/how-to-open-edit-ini-files-2622755)

An INI file is a configuration file used by Windows programs to initialize
program settings. It contains sections for settings and preferences (delimited
by a string in square brackets) with each section containing one or more name
and value parameters.  [source](http://fileinfo.com/extension/ini)

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

