# Container Images, Where to find and How to build

## Table of Contents

1. [Module Introduction](#module-introduction)
2. [What is in An Image](#what-is-in-an-image)
3. [The Mighty Hub Using Docker Hub Registry Image](#the-mighty-hub-using-docker-hub-registry-image)

<br/>

## Module Introduction
<br/>

![chapter-4-1.gif](./images/gif/chapter-4-1.gif "Module Introduction")
<br/>

First we're going to discuss the basic of images and the concepts that you're
going to need.

What is actually in an image and, just as _importantly_, **what isn't in an image**.

We're going to talk a little bit about how to find images on the internet and
we'll actually go and look at some, dive into that whole process to finding good
images and how to manage those image once we've downloaded them or created them
on our won machines.

We'll jump into the fun part of making our own images.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## What is in An Image
<br/>

![chapter-4-2.gif](./images/gif/chapter-4-2.gif "What is in an image")
<br/>

So before we start playing with images and learning how to use them for
containers, we probably want to step into **what exactly is in an image** and
**what isn't**. The way I like to explain is very simple.

Images is the application binaries and dependencies for your app and the
metadata on how to run it.

The official definition, "image is an ordered collection of root filesystem
changes and the corresponding execution parameters for use within a container
runtime", [image definition](https://github.com/moby/moby/blob/master/image/spec/v1.md)

Inside an image is not actually a complete OS, there's no Kernel, there's no
Kernel modules like drivers. It's really just the binaries that your application
needs because the _host_ (server) provides the kernel. That one of distinct
characteristics around containers that make it different form a _virtual
machine_ (hypervisor); It's not booting up a full Operation System. It's really
just starting an application.

An image can be really small. It can be a single file. If you for instance,
using _GO_, one of _GO's_ features is that it can build a static binary and have
a single file as your application.

Or you could have a very big image that's actually using some distribution like
Ubuntu with its own Package Manager built in, and where you've installed Apache,
and PHP, and your source code, and all the added modules you need, you can have
multiple gigabytes.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## The Mighty Hub Using Docker Hub Registry Image
<br/>

![chapter-4-3.gif](./images/gif/chapter-4-3.gif "The Mighty Hub using Docker hub registry image")
<br/>

We going to take tour around [docker hub](https://hub.docker.com), and get to
know a few of it's basic features.

We'll kind of discuss the difference between an official image other just as
good images, and how to tell the difference between a good image and a bad
image.

Showing bit around downloading images and how you can see the different tags
that they'll use and difference between an Alpine image and other image options

### The right Images

There's several characteristics to figuring out what's the right image. Like
convention you just gonna start using images for your workflow that are related
to what you do and you're going to get used to specific ones. You're going
_like_ specific ones. Typically we always start with the **official** tags.

The official tags, it will also be the only one where is name doesn't have
a forward slash `/` in it. When you and I create images on Docker Hub, we have
to create them with our account name in front an image we create.

So when you look at all the other images, with account name like
[tuanany73/multi-nginx](https://hub.docker.com/repository/docker/tuanany73/multi-server)
it might be actually an organization or individual repo.  But the only ones that
get to have just a name of the repo are considered official.

Official images are one that Docker.inc, actually has a team of people that help
take care of them, ensure that they have quality documentation, that they're
well tested and that they're put together properly with _Dockerfiles_ that are
obeying best practice rules.

Official images usually work with the official team of that software who
actually makes it to ensure that it's doing all the things that it should be
doing.

### Tag list `latest`, `mainline`, `stable` and `number`
<br/>

![chapter-4-1.jpg](./images/chapter-4-1.png "Official repo")
<br/>

like I said before you want to start be using with the official to start with
and then you eventually you may find that you want to change it slightly or add
a few thing to it and you'll make your own images.

One of the best things about official images is their documentation. They're
always really great at documenting how to make the images work, what _options_
there might be, what _environment variables_, what's the _default port_ and so
on.

Versions are a common feature of official repositories. You don't have to have
versions in every image but official ones do. Because most open-source software
there's always at least a few official versions out in the wild that are
officially maintained and supported.

That's what we have here, `1.19.2` for Ngnix and then we have what's considered
the stable branch of Nginx, which is `1.18`. We can see the little name here
says `stable`. So let's break this down.

![chapter-4-4.gif](./images/gif/chapter-4-4.gif "check official tag")

When we start talking about images, images aren't necessarily named. Images are
tagged. A version of an image can actually have one more than on tag. We're
going to dive into this a little bit later when we start making our own images
and we can play around with tagging.

`latest` is a special tags, it doesn't necessarily guarantee it's always the
latest commit in the repositories. What it usually means is you're getting the
latest version of this products. In the official it's very well-defined and
consistent, so that if you didn't care right now exactly what version you
wanted, you just wanted the most current, then you could just say `docker pull
nginx`, It would download the latest.

A best practice in your software is when you've going to production and you're
actually testing software that you're going to be using for others, **it's rare
that you really want your software update automatically**. You usually want to
control that process with some other DevOps tools.

But when you're developing or just testing something locally, it's super easy
with official images to just type in the name and just assume you're going to
get the latest.

![chapter-4-5.gif](./images/gif/chapter-4-5.gif "Alpine base images")

You will notice the other one, `1.19.2-alpine`, these all have very similar
names to the first one, but they all have the world `alpine` in them. We're
going to get into base images and distribution later, but just for now, `alpine`
is actually a distribution of Linux that's very very small. It's actually less
than `5MB` in size. This version will actually mean that it comes form a base
image of `alpine`, keeping it very small and light, whereas the default one or
the latest image actually comes from Debian distribution, It;s a little larger
in size probably a little bit over `100MB`.

You'll also notice that the three version that I've downloaded `1.19.` all have
the same `IMAGE ID` because the `IMAGE ID` is based upon the cryptographic SHA
of each image in Docker Hub.

### Unofficial images

![chapter-4-6.gif](./images/gif/chapter-4-6.gif "Unofficial images")
<br/>

If you ever want to consider not using an official repository, what I usually
look for is a _number of stars_ and _number of pull_, because a popular
repository, to me , tends to establish trust.

I always recommend you download and _inspect_ the software before you use it,
and look in the _Dockerfile_ and hopefully they'll have an open source
repository that you can go look at exactly how they made that image.

### Lecture review

![chapter-4-7.gif](./images/gif/chapter-4-7.gif "lecture review")
<br/>

### Miscellaneous

#### List of Official Docker Images

[source](https://github.com/docker-library/official-images/tree/master/library)



**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

