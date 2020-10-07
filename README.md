## What This Course About?

This course is from [Bret Fisher](https://github.com/BretFisher) on
[udemy](https://www.udemy.com/course/docker-mastery/). Bret as Docker Captain
has tremendous effort to explain the core concept about container and why as
developer should use Docker in the server.

I wrote all this lecture cause I know I should reviewing about Docker not only
twice after I learn from Udemy. So I need to write it down in case I forgot.

In every chapter I write the details, from the concepts of theory, the challenge
that Bret give with a `.gif` cause I know reading is boring, so I create a `gif`
picture in every chapter.

## Why So Important Using Container?

The concept about Hypervisor make tremendous effort for container. At last
decade we just use the Virtual Machine into our lab (own PC) for practicing in
development and that use most amount resources of CPU and RAM.

As a web developer when you host your own Web you have choose to use either you
use `[1]` **_Shared Hosting_** like build your own static web and you don't have
any authority to manage your own resource of CPU and RAM and that is sucks as
you want to develop your skill, cause you don't want just to be a web developer
in the Computer Science. You also need to know how the DevOps works, or become
a Machine Learning engineer, or you want to become a Big Data engineer and all
of that need to know how to use your own machine (server).

So you choose `[2]` a **_VPS_** cause you can manage your own resources as free
as you want; And you can create or add more websites in your server as you want.
But the limit is the power of your server? How can you handle the **_web
engine_**, the **_database engine_**, the **_worker engine_**, etc. If you only
have limited resources.

Docker came as game changer, or I can say the revolutionary approach for server,
Docker use a little amount of CPU and RAM in your server. Because Docker is only
a Binary run on top of your kernel server (as I use Unix system as my OS
server). You can mix in all your engines in small of resources.

The big advantage using Docker is you want to develop your app in `[3]` **_Cloud
Server_**; Cause in the next decade (2020 - 2030) all your apps or I can say all
your engines all bundled in one app, and you need to maintain your server
machines if you use multiple servers; And how to develop without the main base
engine in every servers? Docker came with this solution to reduce the gap in
development process, to equate the engines in every servers, so you don't need
to rewrite the tedious script in every single time you launch your apps in any
servers cause you have the same images in each server you deploy your engines in
a single application.

## The Course content

- [Howto Create and Using Container](./chapter-3-creating-and-using-containers-like-a-boss)
- [Howto Build own Container or own Images](./chapter-4-container-images-where-to-find-and-how-to-build)
- [Container Lifetime and Persistent Data Volume](./chapter-5-container-lifetime-and-persistent-data-volumes)
- [Docker Compose](./chapter-6-docker-compose-the-multi-container-tool)
- [Docker Swarm and Cluster](./chapter-7-swarm-and-swarm-cluster)
- [Swarm Features](./chapter-8-swarm-basic-features)
- [Swarm Lifecycle](./chapter-9-swarm-lifecycle)


## License
<br/>

![handshake.gif](./images/gif/handshake-1.gif)
<br/>

This code project is licensed under MIT Licence see the
[LICENCSE.md](./LICENSE.md) file for details

## Acknowledgment

[Bret Fisher](https://github.com/BretFisher)
