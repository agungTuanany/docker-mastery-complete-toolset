# Docker Compose The Multi-Container Tool

## Table of Contents

1. [Module Introduction](#module-introduction)
2. [Docker Compose and The YAML file](#docker-compose-and-the-yaml-file)
3. [Trying Out Basic Compose Commands](#trying-out-basic-compose-commands)
4. [Assignment Building a Compose File For Multi-Container Service](#assignment-building-a-compose-file-for-multi-container-service)
5. [Adding Image Building to Compose File](#adding-image-building-to-compose-file)
6. [Assignment Image Building with Compose](#assignment-image-building-with-compose)

<br/>

## Module Introduction

This section is going to focus exclusively on **Docker Compose**, which is
a combination of a _command line tool_ and a _configuration file_. Requirement
for this section are that you really need to **know a lot of the fundamentals**
from the last few section around _images_ and _containers_.

As we're going through this course, you're probably going to start to hear me
get more and more excited because things just getting better.

As you learn one component, you're thinking, 'man, **bind mounts** and
**volumes** are really cool'. That's going to save me some time. I can imagine
how I'm going to use that. Then I'm going to come with the next thing that's
going to be even better that what you just learned. In this case, it couldn't be
more true.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Docker Compose and The YAML File

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
Compose**.  The _first part_ is the YAML file, and it's super simple to
understand. It's written in YAML, and if you've never dealt with YAML, that is
a very easy _configuration language_. It's almost as easy as an
[INI](#what-is-ini) file would be.  I actually think it's easier because it show
hierarchy. We'll dive into that.  That file _is where you would specify all the
containers you need yo run_, the _networks_ you need, any _volumes_ you might
need, _environment variables_, _images_, and all sorts of other configuration
options.

Then the _second part_ of Docker Compose is **CLI tool**, which is
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

The synopsis of YAML basic elements is given here: Comments in YAML begins with
the (#) character. Comments must be separated from other tokens by whitespaces.
Indentation of whitespace is used to denote structure. Tabs are not included as
indentation for YAML files. List members are denoted by a leading hyphen (-).
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
But I always make it at least version 2. If you don't add that line there, it's
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
related at all. It could be your name for the `servicename`, but it **will be
the DNS name**, that we'll find out later, is used inside of your _Docker
networks_.  Similar to when you give the `--name` to a `docker run` command.

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

# Trying Out Basic Compose Commands
<br/>

![chapter-6-3.gif](./images/gif/chapter-6-3.gif "Trying out basic commands")
<br/>

We previously talked about the Docker Compose YAML file and in this lecture,
we're going to be talking about the _compose command line_. Requirement for this
lecture are like the rest for this section where you really need to know
_images_ and _containers_, and _how to use_ them and the _concepts around_ them from
previous sections, so that you can use all of those inside the `compose` command
and the YAML file.

The Docker Compose command line tool is actually _separate from the Docker
tool_. It's actually a separate binary. If you're on Docker for Windows or
Docker for Mac, it actually comes bundled with that. If you also use the toolbox
on Windows 7, it bundled with that. But if you're on Linux, you have to download
it separately and you can get that on [github](http://github.com/docker/compose),
but you you can aslo just search it on duckduckgo, and there'll be a link in the
reference for this section.

But I should _state really quick_ that it's not designed to be
a _production-grade_ tool. It's super ideal for local and testing things really
quickly that might otherwise be complex to type in a bunch of commands from the
command line.

_Two common command_ that we use will be,

```bash
# Setup Volumes/networks and start all containers
$: docker-compose up

# Stop containers and remove cont/vol/net
docker-compose down
```

Above command is by far probably what you're typing going to be typing 90% of
the time. It's one stop shop. It's basically a single command to do everything
in the compose file, including _creating volumes_, _creating networks_,
_starting all the containers_ with all their configuration options.

Then to clean up when you're done, the `docker compose down`, which a lot of
people actually forget to do, will actually clean up after you. It'll get rid of
the containers, _remove the networks_ and _volumes_ that aren't needed anymore.

It allows you, with these two commands, to quickly jump in and out of different
environments for development. This is where _I really get the attention_ of
developers that are used to using [_vagrant up_](#what-is-vagrant) and other
tool to customize complicated environments locally they need for development.
Once I show them Docker Compose and the compose file and how easy it is to use
along with Dockerfile, it really to start click for them that this could be
a better way and a more reliable way to set up their environment and it doesn't
require the time to set up different VMs or the complexity of managing a virtual
machine environment.

Just to give you a little taste, if you had a project where you had the typical
new developer on-boarding process that was all about learning how to the right
tools for their system, and the right virtual machine downloaded, and the right
add-ons, and all the things that normally they would just have to build
a replicated environment for, with Docker Compose, it's as easy as cloning the
environment that has your compose file from some repo somewhere and then running
the `docker compose up` command. It can really be that easy.

### Jump into Terminal

Just go into [compose-sampe-2](./compose-sample-2/) directory, we got
[docker-compose.yml](./compose-sample-2/docker-compose.yml)

```Dockerfile
version: '3'

services:
    proxy:
        image: nginx:1.13 # this will use the latest version of 1.13.x
        ports:
            - '80:80' # expose 80 on host and sent to 80 in container
        volumes:
            - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    web:
        image: httpd  # this will use httpd:latest
```

This docker compose file, has got _two services_ or _two containers_, and the
first one is an Nginx server which is configured as a _proxy_ in this case; And
that's going to be listening on my port `80` my local  machine (host).

You'll notice the `volumes` command there where I've actually _done a bind
mount_ to a file in this directory called `nginx.conf` file, and it's really
a very simple little file. It's not necessarily important that we know how that
file work yet.  It just shows how, in this case, I'm actually taking a file on
my local environment. Let's say that this directory is a Git repository that
stores these config files for me. It allows me to _map that  file in to the
container_ and I actually, at the end, you'll see it say, `:ro`, that means read
only, meaning I can't it in the container. That's not necessarily required here.
I just wanted to show you that, that's how you would do that in this case.

That config file is actually telling Nginx, instead being a _web-server_ I would
like you yo b a _reverse proxy_, and sit in front of another server; which in
this case we're calling our web-server, with its _service name_, and it's
running the _Apache 2 server_, which is the image `httpd` from the official
image repository on the Docker Hub.

You can also see, at the top, I've specified a `version`. That's how I would do
that inside the `image` line. So, what should happen is when I type this `docker
compose up`, it should start up both containers. It should create a private
network for the two of them, it will _automatically bind mount_ that file, it'll
_open up the port_,  and it will _start dumping logs out_ to my screen.
<br/>

![chapter-6-4.gif](./images/gif/chapter-6-4.gif "basic docker-compose CLI")
<br/>

You will notice the [docker-compose.yml](./compose-sample-2/docker-compose.yml)
file, I didn't specify `networks` area, or `volumes` area because they're not
actually required. They're only necessary if I need to do some custom in the
network, like maybe change the default _IP addresses_ or change the _network
driver_ that's used. Or in the `volumes` case, I can create _named volumes_, or
use a _different volume driver_ and we'll get into all of that later in advanced
sections.

Just so yo know, that's how simple these file can be. You don't need all that
stuff. Those features are there if you need them, but usually these very simple,
little commands like this will get you going.

What have I done? This is now my _Apache server_ and my _Nginx server_. You'll
notice the logs are actually _colored_. That's a neat feature of Compose that
you don't necessarily get out-of-the-box with Docker, is that it will log all
the containers that it's running on the screen in the foreground. I can always
do the same things as I do with Docker and run `docker compose up -d` command
which will run it in the background. But it's nice to see when I'm developing or
testing things locally, that I've got logs right on the screen.

If I jump over to my browser, and then got to `localhost:80`, you'll see that it
says `it works`. That's actually the _Apache server_ **replying**, **not** the
_Nginx server_. The traffic is actually going through the Nginx reverse proxy.
It;s repeating the traffic over the Apache server. The Apache server is
responding with its default basic `.html` file because we didn't change
anything, Then the Nginx is repeating it back to me.

So, it's _your basic web server, web proxy configuration_. You can tell that
traffic hit both of them. They actually work through the proxy because the proxy
is actually a different color in the logs. The logs are pumping out here with
a random color for each container in my setup so that I can easily see what's
going on.
<br/>

![chapter-6-5.gif](./images/gif/chapter-6-5.gif "basic docker-compose CLI")
<br/>

If I hit refresh on the browser several times, you'd see the traffic showing up.
Then it's hitting first the proxy and then the backend web server.

If I want to stop this here, I'd hit `contorl-C` to stop the compose in my
terminal screen. I could actually run it again with the `-d` to run in the
background with `docker-compose up -d`; And you can see back in the browser with
refresh it and it's still there.

I can do a `docker-compose logs` to see to see the same output of logs that
I saw I while a go.

A lot of the command that you might be used to in Docker are also in Docker
Compose. Really what Compose is just doing is it talking to the Docker API in
the background on behalf of the Docker CLI. It's kind of like a replacement of
the Docker CLI still talking to the Docker server API in the backend.

Of course, the great help here, `docker-coompose --help` shows me all the
commands I can run. You'll see a lot of these look really familiar because
they're the equivalent of the `docker COMMAND` just using he context of the
configuration of the compose file.

I can do `docker-compose ps` which will show me that both my containers are
running.

I can do `docker-compose top` to actually list all of the `services` running
inside of them in a nice formatted output.

Then finally, I can do `docker-compise down` to stop and clean up my stuff
I just started. You'll see that it's telling me what it's done there. Cleaned it
all up.

This is just a little taste. We're going to be using `docker-compose` a lot
through the rest of this course for the local development and testing stuff.
Then we'll keep using the Docker Compose files as we transition into the
production concerns for how we deploy complex environments with a simple YAML
file.

### Miscellaneous

#### What is Vagrant

Vagrant is a tool for building and managing virtual machine environments in
a single workflow. With an easy-to-use workflow and focus on automation, Vagrant
lowers development environment setup time, increases production parity, and
makes the "works on my machine" excuse a relic of the past.
[source](http://automationrhapsody.com/what-is-vagrant-and-why-to-use-it/)

You can use vagrant up when you're ready to boot it again. The benefit of this
method is that it will cleanly shut down your machine, preserving the contents
of disk, and During vagrant up you can see the check in acton. If for
example there is a newer version of your box, you will get a notification
[source](http://stackoverflow.com/questions/25966283/should-i-use-vagrant-resume-or-vagrant-up)

If you aren't already using Vagrant Up to make your workflow faster and
smoother, here's why you should be! Let's first break down the primary
components and explain what we're doing. Here's how to build a development
environment on UBUNTU 14.04 with a traditional LEMP stack.
[source](http://www.freshconsulting.com/how-vagrant-up-can-make-development-easier/)

Vagrant is an open-source software product for building and maintaining portable
virtual software development environments; e.g., for VirtualBox, KVM, Hyper-V,
Docker containers, VMware, and AWS.
[source](http://en.wikipedia.org/wiki/Vagrant_(software))

Vagrant is a command-line program that's used in combination with
a configuration file to define, configure, and run virtual machines. Vagrant is
compatible with most of the major hypervisors, including VirtualBox, Hyper-V,
and VMware.
[source](http://www.lynda.com/Vagrant-tutorials/What-Vagrant/685028/736508-4.html)

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Assignment Building a Compose File For Multi-Container Service

All right. For this assignment, I want you to go through the actions of creating
a compose file from scratch. This is very normal, everyday, basic compose file.
It's an exercise you'll go through a lot in building on applications that you
need to work o, or test, or develop on.

As you get used to Compose, you'll actually kind of prefer it locally. It's just
so easy to use and it makes the `docker COMMAND` so much easier to remember.
<br/>

![chapter-6-6.gif](./images/gif/chapter-6-6.gif "Assignment Build compose file for multi-container service")

On this one, you're actually going to build a basic _Drupal_ content management
site. If you're not familiar with Drupal, ti;s an open source content management
server, which is basically just a website builder. You don't have to know a lot
about Drupal, but it's a good example of a fully functional web app, and it
needs a database running behind it.

You're going to need to look up the Drupal images, and the Postgres images on
Docker Hub, and read a little bit through their documentation. Of course
I recommend using the official images, and there are some other things there.

You'll want to use the _ports_ inside the compose file to actually expose
Drupal. I recommend just doing it on `8080`. You can really do it on any port
you want, but let's just pick `8080` for this assignment.

You're going to end up with _two services_. I recommend `version 2` top your
file. You'll have those two services underneath it. One for Drupal, one for
Postgres. You can call the services whatever you want.

Inside the Postgres service, you're going to need to make sure you set
a `password` for it.

Then the way that Drupal works is that once you've started it with `docker
compose up`, you'll actually open up your browser and walk through the setup.
Drupal has a nice, web-based setup. It'll ask you some questions.

A tip there, is that the way these container work in compose file is, remember,
that it uses the _service name_ as the _DNS name_ for one container to talk to
the other. As you're setting it up, one of the options you'll see under advanced
is where the _database server_ is. It defaults to `localhost`, and that's not
going to work because that would mean that you'd have to have your database and
your web server in the same container; And that's not really a best practice for
Docker.

In that case, you're going to change the `localhost` to whatever you call the
Drupal service. Then the Drupal web server will actually talk over the Docker
network to the database server.

If you want to go a little bit extra credit, there's an option that you'll see
on the Drupal documentation on Docker Hub, about using `volumes` to store the
unique data, like the _template data_, and the _uploads_, and the _add-ons_ for
Drupal that are called modules, you'll see that some documentation around that.
I recommend you experiment with that and see if you can get it working in the
compose file.

Then after you started it, going and listing out the `volumes` to see if you can
see them there, and that they look correct.

This should be a fairly short compose file, probably 10 or so lines. Of course
if you get stuck, no worries. The next lecture will have me walking through the
same exercise you're about to go through.

### Assignment Answer

#### Create docker-compose.yml
<br/>

![chapter-6-7.gif](./images/gif/chapter-6-7.gif "Assignment answer create docker-compose.yml")
<br/>

#### Access Drupal website
<br/>

![chapter-6-8.gif](./images/gif/chapter-6-8.gif "Assignment answer Access Drupal website")
<br/>

### Miscellaneous

#### What is Drupal

Drupal is a free and open-source content-management framework written in **PHP**
and distributed under the GNU General Public License. It is used as a back-end
framework for at least 2.1% of all Web sites worldwide ranging from personal
blogs to corporate, political, and government sites including WhiteHouse.gov and
data.gov.uk. It is also used for knowledge management and business
collaboration. [Docker Hub](https://hub.docker.com/_/drupal)

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Adding Image Building to Compose File

This lecture is to be all about _adding image building_ into your
`docker-compose.yaml` file so that it automatically ensure that you have an
image ready to go when you run `docker-compose` command. Like other lectures in
this section, you **really** need to have a good sense of Docker container and
Docker image command line functionality.
<br/>

![chapter-6.9.gif](./images/gif/chapter-6-9.gif "compose adding image building | create your own image with docker-compose")
<br/>

Another thing that Compose can do for you is build your image at runtime. It'll
actually look in the cache for images, and if it has build options on it, it
will build the image when you use the `up` command.

It won't build the image every single time. It'll only build if it doesn't find
it. You'll need to either use the `docker compose build` to rebuild your images
if you change them, or you can use `docker compose up --build`. We'll actually
see that in a minute.

This is really great for when you're using Compose locally and you have images
you want to build. Maybe you don't want to use `docker volume mount`. You
actually want to copy files in, or something, into the images.  Your build
command might be fairly complex, it might have some custom environment variables
or build arguments.

By the way, build arguments, we haven't talked about. That's really just
environment variables that are available only during builds. You can look at
Docker documentation on Dockefiles and it'll talk about build arguments in
depth.

Here, we're going to look at an example of how you might use that locally.

### Jump into case

I'm in the [compse-sample-3](./compose-sample-3) directory.

```bash
$: tree -L 1 compose-sample-3
├── docker-compose.yml
├── html
├── nginx.conf
└── nginx.Dockerfile
```

You can see here I have a pre-built `docker-compose.yml` file, I have
a Dockerfile called `nginx.Dockerfile`. Then a similar `nginx.conf` to what we
have earlier, where actually using Nginx as _reverse proxy_ for our website.o

If we go into [docker-compose.yml](./compose-sample-3/docker-compose.yml), you
can see how I've architected this solution.

```yaml
version: '2'

# based off compose-sample-2, only we build nginx.conf into image
# uses sample HTML static site from https://startbootstrap.com/themes/agency/

services:
  nginx-proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    ports:
      - '80:80'
  apache-web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs/
```

I've actually got **two services**. One is the Nginx proxy like before, and one
is the Apache server like before. But a couple things are different.  Instead of
me specifying the _default`image`_ for Ningx, I'm actually building a _custom
one_. You can see here that I've actually given it some argument where I'm
telling it that the Dockerfile `nginx.Dockerfile` it needs to use is a specially
named Dockerfile. I want to build that Dockerfile in this current directory that
it's in. I want it to name that image `nginx-custom`. It's going to store that
in my local cache.

Then Down there at the bottom, we have a _web server_ running Apache. What I've
done here is I've actually mounted some HTML source file that I have into the
Apache server.

So the scenario here is maybe I'm a web developer, I have a static website here,
underneath the `html/` directory that I don't really need to worry about too
much. It's just using a little bootstrap template I'm going to be editing on
that locally, in my editor here, but I know when I go to production with this
it's actually going to be sitting behind an Nginx proxy.

So, I want to emulate that production environment as much as possible, locally,
just to make sure that I've weeded out any bugs or any problems. In this case
I want this Nginx, and I know that this `nginx.conf` is the one they're going to
use on the production server,

```conf
# nginx.conf
server {

	listen 80;

	location / {

		proxy_pass         http://web;
		proxy_redirect     off;
		proxy_set_header   Host $host;
		proxy_set_header   X-Real-IP $remote_addr;
		proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header   X-Forwarded-Host $server_name;

	}
}
```

So, that I'm just going to use that here. This actually the same Nginx config as
before. When I run the `docker compose up` command, it's going to first check
for the name of this image in my cache. If it doesn't find it, then it's going
to use these `build` command in `docker-compose.yml` to look up the Dockerfile
and build my image `nginx.Dockerfile`.

```Dockerfile
FROM nginx:1.19

COPY nginx.conf /etc/nginx/conf.d/default.conf
```

You can see from `nginx.Dockerfile`, it's pretty simple actually. It's going to
use a specific version of Nginx and then it's just `COPY` a file in. It's going
to get the rest of its information from that Nginx `default.conf` Docker Hub and
then it's going to copy in my `nginx.conf`.

If you remember in a previous example, we actually did a `bind mount` to get
this `nginx.conf` in through our Compose file. In this case, we're just wanting
it to build the whole image. We don't really need to be changing this _proxy
config_, so I don't really need to mount it so that I can change it all the
time. I just want it to run. Right? So, I'm going to build it. Let it sit in the
image cache. Once it's built the first time, it won't need to be really built
very often. So I'm going to leave there.

Then when I'm editing and developing on my website, I'm going to have this
configuration so I can do live edits.
<br/>

![chapter-6.10.gif](./images/gif/chapter-6-10.gif "compose adding image building | create your own image with docker-compose")
<br/>

```bash
$: docker-compose -f docker-compose.yml up
Creating network "compose-sample-3_default" with the default driver [1]
Creating compose-sample-3_nginx_1 ... done [2]
Creating compose-sample-3_web_1   ... done [3]
Attaching to compose-sample-3_nginx_1, compose-sample-3_web_1 [4]
```
> **NOTE**: This steps through has have own image for Nginx and httpd locally

Let's step through what happened. It first `[1]` created the `network`. That's
a normal thing. It does that.  Then it actually is seen `[2]` that _proxy
service_ have the image `Nginx:1.19` it's looking for. It's pulling out
`nginx.Dockerfile` and build it just like a normal `docker build` command. Then
`[2] and [3]` create the containers.  Then's starting my _web server_ and my
_proxy server_. Everything work correctly. `[3]` Docker Compose attach
containers and you can list with `docker ps` command or you can inspect both
containers with `docker container inspect` command.

If all working correctly, we can see at `localhost:80`, it pulled up my default
template, which is a bootstrap template actually. I go over and I can see all my
logs coming up and obviously there's a lot of assets. There's a log entry for
each one and it's going to go through the proxy and trough the web server.

### Bind mount Live changing code

Because _I've bind mounted_ that directory for editing I can actually go back
into my editor. As a web developer I could go in and maybe, you know, edit
something on he website. With change bootstrap `title`.
<br/>

![chapter-6.11.gif](./images/gif/chapter-6-11.gif "bind-moud live changing code")
<br/>

This is a great example of a **very command developer setup** where you need to
build some _custom image_ locally. You also need to mount some files into your
application so that you can edit them at at runtime; And of course, if this
needed to be a database-backed application, I would just add a third `service`
for _database server_. Then, I'll go back and do my cleanup by `docker-compose
down` command.  You'll notice by default, it won't actually delete that image.

You'll also notice, throughout these examples of Compose, that it actually names
containers, volumes, and networks, with the _name of the directory_ as the
_project-name_.

```bash
Creating compose-sample-3_nginx_1 ... done [2]
Creating compose-sample-3_web_1   ... done [3]
```

That's actually something in Compose where, to prevent name conflicts, it will
actually always add the directory name on the beginning of all the assets so
that they don't conflict with other ones. You can actually change the proxy name
through the command line. If you go check out the Compose documentation. But the
point I want to make here is if I do `docker image ls` here, you'll see that it
actually named my image for me.

```bash
EPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
compose-sample-3_nginx        latest              f2284cdfa07f        About an hour ago   133MB
```
So I don't technically have to use that `image` option on my
`docker-compose.yml`. But if I wanted a specific name for me for it so that
I could maybe identify it later, I could.

The reason I was bringing that up is my `docker-compose.yml` actually a little
bit cleaner because what I can do the is `docker-compose down`, and remember in
the `--help`, it said I could use an `--rmi` type.

```bash
$: docker-compose --help
Stops containers and removes containers, networks, volumes, and images
created by `up`.
By default, the only things removed are:
- Networks defined in the `networks` section of the Compose file
- The default network, if one is used

Networks and volumes defined as `external` are never removed.
Usage: down [options]

Options:
    --rmi type              Remove images. Type must be one of:
                              'all': Remove all images used by any service.
                              'local': Remove only images that don't have a
                              custom tag set by the `image` field.
    -v, --volumes           Remove named volumes declared in the `volumes`
                            section of the Compose file and anonymous volumes
                            attached to containers.
    --remove-orphans        Remove containers for services not defined in the
                            Compose file
    -t, --timeout TIMEOUT   Specify a shutdown timeout in seconds.
                            (default: 10)

```

So when I clean it up, ad I like to keep my Docker environment clean, I can do
`--rmi local`, and what will do is actually delete those images as well.

Note something that I do all the time but maybe if I have a big project with
lots of _custom images_ and I don't want it to blow up my system when I'm not
developing on that project, I'll use the `--rmi local`, and it'll always keep
track of the custom images it had to create.

Of course if I use the `--rmi all`, it will actually delete every image that was
used in this project, which you _may not want to do_ because in this case, it
might delete the `httpd` Apache web server, and you might want that image to
stay around of a while.

You learned a lot about Compose in this section. We're going to be using Compose
throughout the rest of this course. We'll keep building on these skills as we
go.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Assignment Image Building with Compose

Now that we've learned how to _add build environment configuration_ to our
Compose file, we're going to start an assignment that is going to build on the
last assignment.
<br/>

![chapter-6-12.gif](./images/gif/chapter-6-12.gif "Assignment image building with compose")
<br/>

We're going to use the same Compose file you had before. This time, we're
actually going to create a custom _Drupal image_ from the official repository,
and w;re going to add some stuff to it.

The scenario here would be is that you're not actually a developer this time.
Maybe you're _sysadmin_, or maybe you're a _software tester_, or maybe you're
just wanting to learn Drupal and you just want to be a Drupal admin.  Docker is
great for that, too.

Compose can really make things so much easier. If you can imagine before, just
a few years ago, if you had a Mac or Windows machine and someone said, 'hey, you
need to learn Drupal,' now you've got to get _postgres_ or _MySQL_ installed.
Then you've got to get PHP and Apache installed. Then you got to get Drupal
installed and all the modules that go with it. Then the templating. You know, it
just it goes on and on. The complexity there really discourages people from
actually trying to run it themselves, which is a shame.

Compose solves all above problem for those people, and you might be one of them,
where you just want to run a piece of software to use it. You don't want to
develop on it. You're not trying to change it. You just want to try it. But the
complexity of getting running was so much before, that it would discourage some
people from even trying.

Again, we're going to start with your Compose file that you built in the
previous assignment for this section. We're going to create a _Dockerfile
first_, that is a _custom image of a Drupal_, with a template added in it.

Basically, all we're doing here is we're creating the same thing as before, only
we're going to build that Dockerfile with a bootstrap template. That's the name
of the template, and it's going to allow you to add the template into the GUI of
Drupal. This really isn't about learning Drupal. Obviously, we're here to learn
Docker.

But I want you to go through to this exercise of understanding how this could be
useful for someone who is not a developer; And how we can build _environment
options_ to our Compose so that we can still just do the single, one line,
`docker compose up`, just like we did before, but now it's actually on the fly
building images if it needs them.

We're going to use Drupal and Postgres just like before. Inside our Repo, the
[compose-assignment-2](./compose-assignment-2/) directory, will have
a [README.md](./compose-assignment-2/README.md), and you're going to use the
`docker-compose.yml` file Hope you saved it form last time. You're going to put
that in that folder. Then also build a Dockerfile. Then follow the instruction
in the README.md.

Most of the lines for the Dockerfile are in README.md, it should be very
explicit about exactly what you have to do. Then you're using the Compose file
before and you're only changing a few lines of it.

Of course, if you get stuck or you just want to see how I would do it, watch the
next lecture. You can watch me run through it as I would do it. Hopefully you'll
take a few minutes and **give this a try** and have some fun.

### Jump into case

#### Create Custom Drupal image
<br/>

![chapter-6-13.gif](./images/gif/chapter-6-13.gif "Create custom drupal image")
<br/>

We focused on the Dockerfile. It says in README.md I need to do af`FROM`
_drupal:8.8.2_. Then n I need to do a `RUN` command to install Git with update
Drupal image; And this talks about some _proper cleanup_. Any time you use
`apt-get`, `apt-get` actually **creates a cache** when you do the `apt-get update`.
So we really want to delete that cache _because_ it's at least `10MB` taking up
our image that we don't need by remove specific file in `/var/lib/apt/lists/*`.
This command in pretty common one. You see this in a whole lot of images. I just
add it to all of my images just to keep them clean. It's not actually
a requirement to make it function. It's just there to keep it clean.

So we need change the working directory for our _template_, or our _themes_.

> **NOTE**: This tip is priceless

Then we're need going to do a clone Drupal bootstrap template. So if you're
a Git Person, you might see the command are familiar. Git, obviously is pulling
or cloning the particular Git repository down that is a template. It's using
a specific branch `--branch`. It's using `--single-branch` with `--depth`
variables are to tell it to **only download the latest copy and only that
particular branch**. It basically saves you a whole lot of time. It actually
saves you minutes on downloading Git repository. If you only need the absolute
latest commit and all the files in it, and you only need a particular branch
which is probably all you need when you're going to deploy an image. Because if
we were going to change that we would just build a new image. Right? In case
you're using Git a lot for image building, it saves you a lot of time and space
really because you don't need the whole history of the repository in there. You
just need the latest commit.

Then the problem here is that cloned repository, this is a common problem in
Docker, that any command you run in Dockerfile, they're all going to run as
a root. So the downloaded files and put it in `/var/www/html/themes` directory
as root. But the problem is that the Apache Web server expect those file to be
the `www-data` user. So I have to change the permission afterward in order to
make it work well.

So the command is very common one `chwon` which stand for change owner and `-R`
means all files, including subdirectories and subfiles, to this user `www-data`
and group. We'll confine (restrict) it to just the directory we created. Save us
a couple microseconds.

I then need to change my working directory back _just in case_ the application
expect to be in the working directory that it started in.

I don't need to specify any other, like `CMD` or anything, because those are
already specified in the from image, the Drupal image that I'm pulling from. All
right. Think I'm done, let's save it.

#### Create Compose YAML file
<br/>

![chapter-6-14.gif](./images/gif/chapter-6-14.gif "Create Compose YAML file")
<br/>

So we have copied the `docker-compose.yml` from last assignment. It has
`services`, and then it has the Drupal with th `image` named `drupal`, and then
the `volumes` for Drupal image; and then the Postgres image, and then the
`enviroenvironment` variable for the Postgres password; and then we define the
general `volumes`.

What we have to do to make this work in this situation, according to our
instructions in README.md. What we need to rename Drupal `image`. Any time there
is, in a Compose file,  an `image` and a `build` keyword, it changes the purpose
of the image, key. Now instead of us telling it, 'hey, I want you to download
the Drupal official image' we're saying, once we add in the `build`, we're
saying, 'hey, I want to build from a specific location on my local machine and
then I want to call the image you build this image name.'.

Now, I don't actually have to specify the image name. I could delete the `image`
line altogether and it would actually create an automatically named name based
on the Compose naming, that you probably noticed. Whenever you use Compose, it
names everything to be friendly to that particular project. But we're going to
hard code it in here with `name: custom-image`.

Notice I'm putting in `.` for `build` key. That's just a shortcut, a shorthand
for just saying, 'hey just build in this directory and use the Dockerfile that's
the default name.' I'm not giving it any `build` options. I'm not telling it
a custom build file or custom location. I'm just telling it the simplest way
which is how we built it. That should build a custom image based on my
Dockerfile I'm just telling it the simplest way which is how we built it. That
should build a custom image based on my Dockerfile.


Then down in Postgres, I need to actually put in a `volume`, we want it to
**preserve our data**. The reason we want to preserve the data is if you're
testing this, and you do a `docker-compose down`, it will actually delte the
database, and then when we turn it back on, it won't have the data in there. So
we want to ensure that we're keeping the data across Compose restarts.

If we do a `docker-compose down` now, after we have this data volume in there,
it won't actually remove that data volume, unless we do a `-v` on the end, which
tells it, 'hey, I also want to delete the volumes.'

Then I need to also put that down at general `volumes`; And we're ready.


#### Start Container and Configure Drupal
<br/>

![chapter-6-15.gif](./images/gif/chapter-6-15.gif "Start container and configure Drupal")
<br/>

We can go through Drupal installation, Everything work as it should.

#### Delete The Image and Rebuild
<br/>

![chapter-6-16.gif](./images/gif/chapter-6-16.gif "Delete the image and rebuild")
<br/>

If I went back and actually stopped this, abort it, and then did a `down`
command, it's actually going to delete the images. But remember, it's not going
to delete the data.

If I did a `docker-compose up` again, what should happen is all those config
files in the Drupal modules and profiles, and sites, and themes and then the
data that's actually in the database, should all still be there. I should be
able to hit refresh and it still works.

What's great about this way is that you can actually keep your images and your
containers relatively clean. That way, when you're shifting between project you
could technically get rid of that stuff and just leave the volumes alone don't
delete the data in them. Just leave the volumes alone and don't delete their
data.

Then when you come back to the project, you've already got all the sample data
already in there and you don't start from scratch.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>
