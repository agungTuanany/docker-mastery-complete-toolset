# Swarm Lifecycle

## Table of Contents
1. [Using Secrets with Local Docker Compose](#using-secrets-with-local-docker-compose)
2. [Full App Lifecycle with Compose](#full-app-lifecycle-with-compose)
3. [Service Update Change Swarm app Lifecycle](#service-update-change-swarm-app-lifecycle)
4. [Healthcheck Dockerfile](#healthcheck-dockerfile)

<br/>

## Using Secrets with Local Docker Compose

What about secrets for local development using the Docker Compose command line.
We've been talking about Swarm up until now. But I'm back on my machine, and
I've got Docker Compose installed which I did not have in my Swarm, right?
Because again, **_Compose is not a production tools_**. It's designed for
development.

I'm in the [secrets-sample-2](./secrets-sample-2), You can see that I have two
password files, and the Docker Compose file that had in the Swarm. Just to prove
that I'm not in a Swarm,

```bash
$: docker node ls
Error response from daemon: This node is not a swarm manager.
 Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

So I don't have access to the Swarm database or the ability to put secret in it.
So, how do we deal with this in local development? Ideally, we can still use the
same Compose file. We can use the same objects like the environment variables
for Postgres. Docker had to come up with a way to make this work in test and
dev.
<br/>

![chapter-9-1.gif](./images/gif/chapter-9-1.gif "Secrets with compose local simulated secrets")
<br/>

If we do, `docker-compose up -d` , and then we did `docker-compose exec pqsl`.
then did a `cat` on `/run/secrets/psql_user`

```bash
$;docker-compose up -d

$ docker-compose exc psql cat /run/secrets/psql_user
```

How did our secret get in there, right? Because we don't have the database.
Well, it turns out there's a little bit of magic here.

Well it's not magic. It's just hiding behind the scenes, that what's actually
happening with Compose is **_not secure_**, but it does work. It basically bind
mounts at runtime that actual file on my hard drive into the container. It's
really just doing `-v` with that particular file in the background.

**_Again, this is totally not secure and it's not supposed to be_**. It's just
a way to get around this problem and allow us to develop with the same process
and the same environment variable secret information that we would have in
production, only now we can do it locally to.

Which is great because now that means now that means we can develop using the
same _launch script_ and the same way we get environment variables into our
container just like we would in Swarm; And that just what we really want. We
want to match our production environment as much as we possibly can locally.

You need the latest Docker Compose to do this. I believe it only works in Docker
Compose `11`.

I hope you think that's pretty cool because I thought that was a good compromise
for them to make in order to let us use the Secrets commands.

Now I will point out this works only with **_file-based secrets_**. It will not
work with the external that we talked about.

If we look at the Compose file real quick,

```yaml
version: "3.1"

services:
  psql:
    image: postgres
    secrets:
      - psql_user
      - psql_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_password
      POSTGRES_USER_FILE: /run/secrets/psql_user

secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    file: ./psql_password.txt
```

I would have to use _file-based_ ones for my local development. Maybe if you're
using external in your production you just might have to have a different
Compose file for development that would have the _file_ attribute and specify
_sample_, dummy file in the same directory or somewhere else you might store
them, that are just using simple password for development.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Full App Lifecycle with Compose
<br/>

![chapter-9-2.gif](./images/gif/chapter-9-2.gif "Full App Lifecycle with Compose")
<br/>

In this section, we've covered a lot about Swarm. Swarm, Stacks, and Secrets are
kind of like a trilogy of awesome features that can really make things much
easier in production for us. I want to show you what it might be like if you
actually took your Compose file, organize them in a couple of ways, and this is
what I basically call living the dream.

It turns out you can actually use a single file to do a lo of things, but
sometimes your complexity grows and you're going to need multiple Compose file.

I want just run through real quick. You don't actually have to do this yourself
if you don't want to. I'm just going to show you an example of how these Compose
files might work together to build up your environment as you go.

In this scenario, we're going to use `docker-compose up` for our local
development environment.

We're going to use a `docker-compose up` config and file for our CI environment
to do integration testing.

Then when we're in production, we're going to use those file for `docker stack
deploy` to actually deploy the production environment with a stack.

So, I'm on my local machine and we're going to be using the
[swarm-stack-3](./swarm-stack-3) example before of the Drupal scenario, with
a database server, and web frontend. We have the Dockerfile we have used in our
previous assignments. We're rebuilding a custom, yet simple, Drupal config with
a template.

### Multiple Compose File

We're going to have this default Compose file
[docker-compose](./swarm-stack-3/docker-compose.yml)

```yaml
version: '3.1'

services:

  drupal:
    image: custom-drupal:latest

  postgres:
    image: postgres:12.1
```

This is called _override_, what we're about to do.  An override is where I have
the standard Docker Compose file `docker-compose.yml`, and it sets the defaults
that are the same across all my environments.

> **_By defaults_**, Compose _read two files_, a `docker-compose.yml` and am
> optional `docker-compose.override.yml` File **_by convention_**. the
> `docker-compose.yml` contains your base configuration. The `override.yml`
> file, as its name implies, can contain configuration overrides for existing
> services or entirely new services.
> [source](https://raw.githubusercontent.com/docker/compose/1.27.4/contrib/completion/bash/docker-compose)

Then I have this override files `override.yml` named by default, Docker Compose,
if it's named this exact name,
[docker-compose.override.yml](./swarm-stack-3/docker-compose.override.yml), it
will automatically bring this in whenever I do a `docker-compose up`

```yaml
version: '3.1'

services:

  drupal:           # No image name
    build: .
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - ./themes:/var/www/html/themes

  postgres:         # No image name
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

secrets:
  psql-pw:
    file: psql-fake-password.txt
```

You'll see that in this scenario, we're assuming _local development_ because we
really want the hand-typed command we're going to type to be easiest locally.
Normally in your CI environment, it's all automated so we don't really care if
those commands are a little bit longer.  But we really want locally, is the easy
`docker-compose up`.

The cool thing is Docker Compose will read this file automatically and it will
apply this over _top_, or _override_, any settings in the Docker Compose YAML
file.

So here notice I don't have the _image name_, because that's already specified
over [docker-compose.yml](./swarm-stack-3/docker-compose.yml); And this override
file, I override by saying 'I want to build (`build: .`) the image locally using
the Dockerfile in this current directory'. I want to create a port on `8080` for
local development; And I'm setting up some `volumes`, and you'll even notice
I gave you an example here of a **_bind mount_**,

```yaml
volumes:
    - ./themes:/var/www/html/themes
```

Where I might be doing a custom theme; And I want to mount my theme on my host
into the container like we did in previous sections, so that I can change it
locally and then see it right away on the server.

By the way, for this example, I don't actually know how to develop themes in
Drupal. I'm not exactly sure that if I change a file in there, it will
automatically be reflected. I just want to show an example of how when you're
doing development in web, typically this is the way you would do it without
having to stop and start the Compose every time.

Down here, under Postgres, we have the `environment` variable and the `secrets`
setting like before. We have the defined `volumes`, and you'll see at the bottom
I actually have the _file-based_ secrets because when we're doing local
development, _we have to use the file-based secrets_.

Things get a little interesting when I look at
[docker-compose.prod.yml](./swarm-stack-3/docker-compose.prod.yml), or
[docker-compose.test.yml](./swarm-stack-3/docker-compose.test.yml). The way this
is going to work is, remember that the `.override.yaml` automatically gets
picked up by the Docker Compose command line. In prod or test, I'm going to
specify them manually.

So for the test we're going to use `-f` command. If you remember from earlier
sections, the `-f` is when we do a `docker-compose` that we want to specify
a custom file. I'll show you that in a minute.

Then in production, since we're not going to actually have the Docker Compose
command line on a production server, what we're going to do here is we're
actually going to use a `docker compose` config command. The config command is
actually going to do an output by squishing together, or combining, the output
of multiple config files. So that will be really cool.

Real quick, the
[docker-compose.test.yml](./swarm-stack-3/docker-compose.test.yml) file just has
the Drupal and the Postgres. Imagine if this was your Jenkins CI or Codeship CI
solution, where I want it to build the image every time I commit my code, and
I want to call it this file, and I want to be on this port `80:80` for testing
purposes. Then I'm going to use
a [psql-fake-password.txt](./swarm-stack-3/psql-fake-password.txt) . But I don't
need to define of the volumes because I'm not going to actually try to keep
named volume data because again, it's just a CI platform. As soon as it passes
test or fails tests, it'll get rid of everything.

Then in this scenario, you might see that I've actually got this sample data
scenario
```yaml
postgres:
    volumes:
        - ./sample/data:/var/lib/posrgresql/data
```
Where maybe in your CI solution, you have simply database sitting there that
come from either a custom Git repository or maybe they're an FTP download; Or
something happens during the initialization of your CI where it actually
download a database file; And instead of us having to create our database every
single time we do CI testing, we would just mount this directory of sample data
into where our Postgres data is suppose to be; And that way, we could guarantee
we had the same sample data every single time we do a CI test.

I'm not going to go into any further. I just wanted to show hat might be how
this file for CI would be different.

Then in production
[docker-compose.prod.yml](./swarm-stack-3/docker-compose.prod.yml), we have all
of our normal production concerns.

```yaml
version: '3.1'

services:

  drupal:
    ports:
      - "80:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - drupal-themes:/var/www/html/themes

  postgres:
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

secrets:
  psql-pw:
    external: true
```

We're specifying `volumes` for our specific data. We're specifying our `secret`;
And notice down at the bottom, we have the `external secret`. Because we're
going to have put the secret in already via the command line like we did
earlier assignment.

The point here is all three of these configs are different on some way, but they
all relate to the core config, or base config, which just defines the two
services and their images (docker-compose.yaml).

### Jump into command

#### Default Compose and Override YAML file
<br/>

![chapter-9-3.gif](./images/gif/chapter-9-3.gif "Default Compose and Override YAML file")
<br/>

If you're in the [swarm-stack-3](./swarm-stack-3) directory, you'll see that
I have the base file, and then the three override files. Again remember, that
the `override.yml` is the default.

If I do a `docker compose up`, what it will actually do here is use the
_docker-compose.yml_ first, and then it will overlay the
_docker-compose.override.yml_ one on top. I want to put `-d` in so we can take
a quick look after is started.

```bash
swarm-stack-3$: docker-compose up -d

Creating network "swarm-stack-3_default" with the default driver
Creating volume "swarm-stack-3_drupal-data" with default driver
Creating volume "swarm-stack-3_drupal-modules" with default driver
Creating volume "swarm-stack-3_drupal-profiles" with default driver
Creating volume "swarm-stack-3_drupal-sites" with default driver
Creating volume "swarm-stack-3_drupal-themes" with default driver
Creating swarm-stack-3_postgres_1 ... done
Creating swarm-stack-3_drupal_1   ... done
```

I'm going to do a `docker inpsect` on the Drupal image,

```bash
$: docker inpsect swarmstack3_drupal_1
...
...

        "Mounts": [
            {
                "Type": "volume",
                "Name": "swarm-stack-3_drupal-profiles",
                "Source": "/var/lib/docker/volumes/swarm-stack-3_drupal-profiles/_data",
                "Destination": "/var/www/html/profiles",
                "Driver": "local",
                "Mode": "rw",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "bind",
                "Source": "/home/daun/Project/docker/docker-mastery-the-complete-toolset/chapter-9-swarm-lifecycle/swarm-stack-3/themes",
                "Destination": "/var/www/html/themes",
                "Mode": "rw",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "volume",
                "Name": "swarm-stack-3_drupal-sites",
                "Source": "/var/lib/docker/volumes/swarm-stack-3_drupal-sites/_data",
                "Destination": "/var/www/html/sites",
                "Driver": "local",
                "Mode": "rw",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "swarm-stack-3_drupal-modules",
                "Source": "/var/lib/docker/volumes/swarm-stack-3_drupal-modules/_data",
                "Destination": "/var/www/html/modules",
                "Driver": "local",
                "Mode": "rw",
                "RW": true,
                "Propagation": ""
            }
        ],
...
...
```

What I want to show you was in here, it's got all the `Mounts` listed. So we
know that **_it took the override file_** because the override file was where
defined all of these `Mounts`. So we know that it picked that up.

Obviously, if it didn't pick up the base one, it wouldn't even know what images
to use so it would actually be complaining to us and saying that the _Compose
file was incomplete_.

#### Working with CI solution
<br/>

![chapter-9-4.gif](./images/gif/chapter-9-4.gif "Working with CI solution")
<br/>

If we were going to actually do the command we needed for our CI solution, what
we would have to do on our CI solution was to make sure that Docker Compose was
there, and installed, and available so that we could do `docker-compose`
command. Then we specify `-f` and the order of the `-f` is the **_base file always
needs to be first_**.

```bash
$: docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d
Creating network "swarm-stack-3_default" with the default driver
Creating swarm-stack-3_drupal_1   ... done
Creating swarm-stack-3_postgres_1 ... done
```

Then I went and inspected that same Drupal, you will notice that there's no
**_bind `Mounts`_**

```bash
...
...
    "Mounts": [],
....
....
```

They're completely missing because in the test file, we didn't specify those. We
didn't actually need Drupal to save information because it was going to be
thrown away at the end of our CI run.

#### Working with Production Compose file
<br/>

![chapter-9-5.gif](./images/gif/chapter-9-5.gif "Working with Production Compose file")
<br/>

The third, we have the production config. The production config is going to be
a little bit different. I run the command,

```bash
$: docker-compose config --help
Validate and view the Compose file.

Usage: config [options]

Options:
    --resolve-image-digests  Pin image tags to digests.
    --no-interpolate         Don't interpolate environment variables.
    -q, --quiet              Only validate the configuration, don't print
                             anything.
    --services               Print the service names, one per line.
    --volumes                Print the volume names, one per line.
    --hash="*"               Print the service config hash, one per line.
                             Set "service1,service2" for a list of specified services
                             or use the wildcard symbol to display all services.
```

Instead I'm use `up` command, what I want to do here is `config`. If you just
look at the help real fast for `config` options command, it has several options.
What we want just do is just run `config` by itself.

```bash
$: docker-compose -f docker-compose.yml -f docker-compose.test.yml config
secrets:
  psql-pw:
    external: true
    name: psql-pw
services:
  drupal:
    image: custom-drupal:latest
    ports:
    - 80:80/tcp
    volumes:
    - drupal-modules:/var/www/html/modules:rw
    - drupal-profiles:/var/www/html/profiles:rw
    - drupal-sites:/var/www/html/sites:rw
    - drupal-themes:/var/www/html/themes:rw
  postgres:
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql-pw
    image: postgres:12.1
    secrets:
    - source: psql-pw
    volumes:
    - drupal-data:/var/lib/postgresql/data:rw
version: '3.1'
volumes:
  drupal-data: {}
  drupal-modules: {}
  drupal-profiles: {}
  drupal-sites: {}
  drupal-themes: {}
```

What it's going to actually do is look at both files and push them together into
a single Compose file equivalent.

So what we could do here is just run this command somewhere in our CI solution.
Then have it output a file, maybe with you know, something like
`output-ci-prod.yml`. That  output file would be use officially in production to
create or update our stack.

```bash
$: docker-compose -f docker-compose.yml -f docker-compose.test.yml config > output-ci-prod.yml
```

### Caveats

However, I want to throw in a little caveat here. This is all relatively new
stuff. We know the Secrets and Swarm Stack are relatively new. They're only
a couple of months old as this videos I'm recording.

So there's a couple of rough edges. We just ran that `config` command, and
you'll actually notice that the `secrets` weren't listed in there. That's a bug
currently. I'm actually working with the Docker team to see if we can't squash
that bug. So by the time you read this, it may have already been fixed. Make
sure you inspect that output of the config line before you go deploying in
production.

Secondly, the Compose `extends` option, which I did not discuss here, is another
way to override these Compose file where you actually use the override file and
you actually define an extends section in there. It's a little bit more
declarative, so it's easier to understand. I'll provide a link in the
references for this section. Just know that `extends` options doesn't yet work
in Swarms Stacks.

I didn't mention it here because it doesn't really give you the full app
lifecycle that were hoping for, but I do expect them at some point to do
something about that. Like either add it into Swarm or create a better workflow.
Because that's really the idea we're trying to get to with all of these tools,
is to have a complete and easy lifecycle from development all the way through
test, into production, with the same set of configuration, in the same images.

I hoe this got you thinking about how you might make your apps this way, and how
you might extend your own Compose files for complex scenario.

#### References

- [Use Compose In Production](https://docs.docker.com/compose/production/)
- [Use Multiple compose Files](https://docs.docker.com/compose/extends/#multiple-compose-files)

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Service Update Change Swarm app Lifecycle

Service updates. You've probably assumed all long that there's some way to
update your service even though we haven't been focusing on that yet. Let's talk
about it because updates has a whole lot of stuff going on under the covers.

Swarm's update functionality is centered around a rolling update pattern for
your replicas. Which means if you have a service with more than one replica, and
hopefully yo do, it;s going to roll through them by default, one at a time,
updating each container by replacing it with the new settings that you're
putting in the `update` command.

A lot of people will say that orchestrators prevent downtime in updates, but I'm
not going to say that this prevents downtime, I'm going to say it limits
downtime. Because preventing downtime in any system is the job of testing. You
really need to start testing how you do your updates and determining, does it
impact my users? Does updating a database impact my web application? It probably
does.

Each application that you have a _service_ for is going to update and impact the
other things around it differently. That's not the job of an orchestrator,
orchestrator can't determine that this one is a database protocol, and this one
is REST application protocol that's easy.

So those, are going to be different, complicated things that you need to deal
with. In the case of _service updates_, if it;s just a REST API or a web
frontend that's not doing anything fancy like _Web Sockets_ or long polling, if
it's something very simple like that or a static website, you'll probably have
an easy update and it won't be a problem. But other services like database or
persistent storage or anything that requires a persistent connection, those are
always going top be a challenge no matter what you're trying to update. _So test
early and test often_.

Like I said before, this will definitely replace containers in most updates.
Unless you're updating a label or some other metadata with the service, it's
going to roll through and change out each container with a totally new one. Just
be prepared for that.

It has many options. In fact, the last I counted, there was at least 77 options
for the update command. But just above everything you want to do can be tweaked.

A lot of the options in the `update` command are actually create options that
just have a `-rm` and `-add` on the end of them. Because if it's an option that
can be used for multiple values, let's say a port to publish or an environment
variable, those you can  use many of them. Right? So, you need to be able to
tell the update command which ones you're adding and which one you're removing.
We'll see those in a minute.

This also includes `rollback` and `health check` options. You should look at the
options for those and see if their default values and aren't ideal for your
application and test different settings to see if it makes an update easier for
you and your system.

You also will see that we have `scale` and `rollback` options in here that are
their own commands now. They used to be options that you had to specify with the
`--rollback` or `--scale`, but so many people have been using those so
frequently that Docker is now making them sort of first class citizens in the
command line. They might be adding more of those in the future.

Lastly, before we get to some examples, if you're doing stacks, a `stack deploy`
to the same stack is an update. In fact, there is no separate option for stack
updates. You just do a stack deploy with the same file that's been edited. Then,
it will work with the `service` command and the `networks`, and every other
things it does. It will work with them to make sure if any changes are
necessary, that they get applied.

Let's look at some quick examples and then we'll get to the command line.

### Swarm update Example

#### Update the image used to a newer version

```bash
$: docker service update --image myapp:1.2.1 <servicename>
```
This one is probably the most common example that you'll be using, which is to
change the image of a service. Every time you update your app and you build
a new image, you're going to have to do a `servic update` command, with the
_image name_, and then the _service name_. So in this case maybe I had my
application with a tag of `1:2.0`. and this case I'm now applying a `1.2.1`
image, and the service will determine, ah yes, that's a different image than
I have running in my service, and we'll go and update them.

#### Adding an environment variable and remove a port

```bash
$: dockr service update --env-add NODE_ENV=production --publish-rm 8080
```
On this next one, we're going to showing how you can do multiple things at once
inside a single update command. You can add an environment variable withe the
`--env-add`, and then you can remove a port with the `--publish-rm`.

We could also be adding and removing environment variables and publish ports in
the same update command as much as we want.

#### Change number of replicas of two services

```bash
$: docker service sclae web=8 api=6
```
On this last one, this is showing how we can use these how _scale_ and
_rollback_ commands on multiple services at the same time. Which is one o the
advantages of using them over the `update` commands is that they can apply to
multiple services.

In this case, I'm actually going to be changing the number of replicas of the
web and API services at the same time. Like I said while ago, in the Swarm
updates, you don't have a different deploy command. It's just the same `docker
stack deploy`, with the file you've edited, and it's job is to work with all of
the other parts of the API to determine if there's any changes needed, and then
roll those out with a `service update`.

### Jump Into Examples

#### Update the image used to a newer version
<br/>

![chapter-9-6](./images/gif/chapter-9-6.gif "Update the image used to a newer version")
<br/>

Let's start by actually creating a service so that we can manipulate it with
some service so that we can manipulate some `update` commands.

```bash
$: docker swarm init --advertise-addr 192.168.0.102
Swarm initialized: current node (rr8sira8ve87201i4assb5wwd) is now a manager.

$: docker service create -p 8080:80 --name web nginx:1.18
0rsyhug5oix1bt6e6yyik
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

We use Nginx, and we're going to specify a version like we always in production.
The detach option with our `service create` and `service update` commands, we
can actually see this happen synchronously in real time.

So this will be good for update commands to see how updates actually roll out
via the command line.

Now, if we do `docker service ls`,

```bash
$: docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
0rsyhug5oix7        web                 replicated          1/1                 nginx:1.18          *:8080->90/tcp
```

We see that our service is running and that it's got `1/1` replicas, so it's
good to go.

Now lets **_scale our service up_** so we can have some _more replicas_ to work
with. `docker service scale web=5`;

```bash
$: docker service scale web=5
web scaled to 5
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged
```

And you just saw that one of those was already running, and the other 4 had to
start. It went pretty quickly on mine because I already had the image
downloaded, but yours may take a few minute while they're in pending state as
the image downloads on the other nodes.

Let's do a rolling update by changing the image of that Nginx, with command
`docker service update --image`

```bash
$: docker service update --image nginx:1.19 web
web
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged

$: docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
0rsyhug5oix7        web                 replicated          4/5                 nginx:1.19          *:8080->90/tcp
```

Docker doesn't care about what the image actually is, it could be a completely
different application for all it cares. It just knows that in this service I'm
changing t to a different image.

Remember, by default, it's going to go through here one at a time. It will first
remove it, create a new one, and then when that one's good to go, and it's look
healthy, it'll start in the next one.

#### Adding an environment variable and remove a port
<br/>

![chapter-9-7.gif](./images/gif/chapter-9-7.gif "Adding an environment variable and remove a port")
<br/>

In this example, we're going to change the published port. But you can't change
a port. You **_actually have to add and remove them at the same time_**.

So in this case, because we first published it with an `8080`, we need to do
a `docker service update`

```bash
docker service updae --publish-rm 8080:80 --publish-add 9090:80 web
web
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged

$: docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
9t89k0c3gpuo        web                 replicated          5/5                 nginx:1.19          *:9090->80/tcp
```
#### Rebalancing Nodes of Swarm
<br/>

![chapter-9-8.gif](./images/gif/chapter-9-8.gif "Rebalancing Nodes of Swarm")
<br/>

The last example I want to talk about is kind of a tip. Because you'll often
have a challenge with something called **_rebalancing_**. Or if you change the
number of nodes or if you move a lot of things around, if you have a lot of
containers in your Swarm, you may find that they're not really evened (equalize)
out. You've got maybe some nodes that are pretty light in how many containers
are running and other ones that have a lot.  If you have a lot of things
changing, Swarm will not move things around to keep everything balanced in terms
of the number of resources used.

But, what you ca do is you can force an update of a service even without
changing anything in that service. Then, it will reissue tasks, and in that
case, it will pick the least used nodes, which is a form of rebalancing.

A lot of times in a smaller Swarm when I move something big, or add a bunch of
nodes, I suddenly have a bunch of empty servers doing nothing and I need to get
work on them. So what I'll do is take one or two of my services, and I will do
a `docker service update --force <service-name>`, and in this case it's going to
roll through and completely replace the tasks.

```bash
$: docker service update --force web
web
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged
```

Of course it will use the schedule's default of looking for nodes with the least
number of containers and the least number of resources used.

That's kind of a trick to get around an uneven (odd) amount of work on your
nodes.

**_Remember to clean up by removing the service that we created in this
lecture_** with `service rm` command.

```bash
$: docker service rm web
```

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

## Healthcheck Dockerfile

#### Supported in Dockerfile, Compose YAML, `docker run`, and Swarm services

### Docker Healthcheck

The Docker Healthcheck command It was a new feature add in `1.12`, which came
out _mid 2016_, the same time that Swarm Kit and Swarm Mode were available in
Docker. It was added already as a part of that toolkit, but it still works in
all the different files like the _Dockerfile_, the _Compose file_, the `docker
run` command uses it, the _Stack files_ support it, the `service update` and
`service create` command, support it. It's everywhere.

I highly recommend that when you're going production, you do engage in testing
options for this heath check command. It's going to work right out of the box
with an `exec` command.

#### Docker engine will `exec's` the command in the container (e.g. `curl localhost`)

It's going to execute that command inside the container just like if you were
running your own `exec` command.

So it's not running it from outside the container it's just running it inside
which means that even simple workers that don't have exposed ports, you can run
a simple command in them to validate whether they're returning good data or
whatever.

#### It expects `exit 0` (OK) or `exit 1` (error)

It's a simple execution of a command, which means it gets a simple return. It
expects a `0` (OK) or `1` (Error). In Linux and Windows, you have exit codes
from commands and a `0` is a good thing. It mean everything was fine. Anything
other than `0` is going to be an error in most applications. But in Docker, we
need that applications to exit a `1` specifically. We'll show in a minute how
you do that.

#### Three container states: starting, healthy, unhealthy

There's only **_three states_** to a Healthcheck in Docker. It start out with
**_starting_**. Starting is the first `30` seconds, by default, where it hasn't
run a Healthcheck command yet. Then it's going to run one. If that returns
a `0`, it'll start with the healthy. It'll change to the healthy option. It'll
take that command and it'll run it every `30` seconds by default again. If it
ever receives an unhealthy return, like an exit `1`, then it marks it as an
unhealthy container.

We have options for controlling all of this including retries. We'll see that in
a minute.

#### Much better then "is binary still running?"

This is much better option that we've had in the past because Docker, until
now, was just making sure the application was still running. It didn't have any
insight into whether that application was doing what it was supposed to. Now we
can do that inside the Docker container itself.

But this isn't replacement for your third party monitoring solution. This isn't
going to give you a graphs, or status over time, or any sort of third party
tooling that you would expect out of a monitoring solution.

This is about Docker understanding if the container itself has a basic level of
healthy. So in Nginx, it might return a localhost of the root index file.
A return `200` or `300` is fine and gives it an exit code of `0`, and it
considers it healthy.

That's not a super advanced monitoring tools. But if did return `404` or `500`
error, it would then consider it unhealthy and we can do something about that.


### Docker Healthcheck Cont.

Where are we going to see this Docker Healthcheck in the GUI?

#### Healthcheck status show up in `docker container ls`

The first place is in `container ls`. It'll just see it as this new option. It's
in the middle. We'll see in a second where it' show us one of the three states
if the health check is running, and that's how we actually know that there's
a Healthcheck. That's the easiest way, at least, to know.

#### Check last 5 Healthcheck with `docker container inspect`

We'll see the history, the last five of that Healthcheck, show up in the inspect
for that container; And we ca see some basic trend over time there.

#### Docker run does nothing with Healthcheck

But the `docker run` command does not take action on an unhealthy container.
Once the Healthcheck considers a container unhealthy, `docker run` is just
going to indicate that in the `ls` command, and in the `inpsect`, but it's not
going to take action.

#### Service will replace task if they fail Healthcheck

That's where we expect the Swarm Services to take action. So the stacks and
services will actually replace that container with a new task, on a new host
possibly, depending on the scheduler.

#### Service updates wait for them before continuing

Even in the `update` command, we see a little extra bonus by using the
Healthcheck because the updates will consider the Healthcheck as a part of the
readiness for that container before it goes and changes the next ones.

If a container comes up, but it doesn't pass its health check, then the `service
update` won't go to the next one. Or it'll take action based on the changes you
give it.

### Healthcheck Docker run Example

``` bash
$: docker run \
    --health-cmd="curl -f localhost:9200/_cluster/health || false" \
    --health-interval=5s \
    --health-retries=3 \
    --helath-timeout=2s \
    --health-start-period=15s \
    elasticsearch:2
```

Let's look at few examples before we go to the command line. This is one that
we're using on `docker run`. This allow us to use an existing image that doesn't
have a health check in it, and we're adding the health check in at runtime. In
this case, we're using the _Elasticsearch_ image.

You can see the command is a `curl localhost:9200`, which is the port of the
Elasticsearch is running on inside the container, not the published port. For
Elasticsearch, there is an actual _health URL_. So, we can use that here.

You'll notice the two pipes `||` with _false_ at the end of that command; And
that's going to be pretty common if using something like `curl` or another tool
that will send out an error code that's other than `1`. Remember when
I mentioned that while ago? We need it to exit with `1` if there's a problem.
Because that's the one error code that Docker is going to do something about.

We need to make sure that in this case, a shell will always return the _false `1`
exit code_. If there's anything coming out of that command other than `0`. It's
a nice way to get around that problem. It just so happens with `curl`, `curl`
will give other potential error codes and we don't want it to do that.

### Healthcheck Dockerfile Example

#### Options for Healthcheck commands

```Dockerfile
# Options for healtcheck command
- --interval=DURATION (default: 30s)
- --timeout=DURATION (default: 30s)
- --start-period=DURATION (default: 0s)(17.09+)
- --retries=N (default: 3)
```

In the actual Dockerfiles, we can add the same command. That format's a little
bit different. But you see that we have these options here. We have the
`interval`, the `timeout`, the `start-period` (which is new), and `retries`.

The `--interval` is what you would think it is. It's, by default, every _30
seconds_. How often it's going to run this Healthcheck.

The `--timeout` is how long it's going to wait before it errors out and return
a bad code. If maybe the app is slow.

The `--start` period is a new features that allows us now in `17.09` and newer, to
give a longer but wait period that the first `30` seconds of the duration.
Before, it would always just wait the interval time before it started the
Healthcheck. But maybe you have a Java app, or database or something that takes
a lot longer to start. Maybe it takes five minutes.

You could add that `start-period` in there. It'll still do Healthchecks. But what
it will do is it _won't alarm on an unhealthy check_ until that time has
elapsed. So if you set two minute in there, even through it's health checking
every `30` seconds, it's going to only consider it unhealthy once it's past that
two minute mark.

The last one there, `--retries`, means that we will try this Healthcheck `x`
number of times before we consider it unhealthy. That gives maybe a potentially
unstable app a chance to come back with a healthy and recover on its own before
we consider this a truly unhealthy container.

#### Basic command using default options

```Dockerfile
HEALTHCHECK curl -f http://localhost/ || false
```

The basic Healthcheck command you would use in Dockerfile is called
`HEALTHCHECK`, all capital letters there. The same format exists where if we're
just doing a simple `curl` of the localhost beause mayble it's PHP app or
something. We can do that.

#### Custom options with the command

```Dockerfile
HEALTHCHECK --timeout=2s --interval=3s --retries=3 \
    CMD curl -f http://localhost/ || exit 1
```

This is how you would add all those options in to a Dockerfile so you would see
how I add the `timeout`, `interval`, `retries` before the command itself.

The first one there for the basic command, notice I don't have to put in a `CMD`
if I'm just giving it the command to run. But if I want to show options, if
I want to give it custom options out of the box with the `timeout` and so on,
then I have to specify which one is the command.

Now these aren't two different lines. Notice the `\` on the end of the first
line there. So don't get that confused.

#### Static website running in Nginx, just test default URL

```Dockerfile
FROM nginx:1.13

HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost/ || exit 1
```

Here we have a simple example of what it might be like if you had a static
application running inside an Nginx server. You could set the `interval` and the
`timeout` form your Dockerfile, and you would just have it simply do a `curl`
command on localhost.

If it return a `200` or `300`, it considers that fine. If it return a `4` or `5`
or something else, it considers that an error.

You notice here that I have an `exit 1`, which is the same thing as a _false_.
I did that just to show you that certain examples on the internet will have
a `false`. Certain examples will have an `exit 1`. They bot do the same thing.

#### Healthcheck in PHP Nginx Dockerfile

> PHP-FPM running behind Nginx, test the Nginx and FPM status URLs

```Dockerfile
FROM your-nginx-php-fpm-combo-image

# Don't do this if PHP-FPM is another container
# Must enable  PHP-FPM ping/status in pool.ini
# Must forward /ping and /status URLs from Nginx to PHP-FPM

HEALTHCHECK --interval=5s --timeout=3s \
    CMD curl -f http://localhost/ping || exit 1
```

Here's an example, little bit more advances. In this case, we're using a PHP app
that's combined with Nginx.

What I've done is, in the resources, you'll find a link to this [PHP
example](#php-docker-good-defaults) I've added in a custom Nginx config file
that uses Nginx and PHP-FPM status URLs.

Both of those applications have their own status page and sort of a Healthcheck
ping URL. You can use those in our apps if you're using PHP on Nginx. There are
two different URLs, but you can use both of them inside the same Healthcheck. In
this case, we're using just one of them, and we're throwing in the
`localhost/ping`, which is actually a PHP-FPM status command, but you have to
enable that inside our PHP-FPM, Again, in this resources of this lectures,
there's a link to a [PHP-Docker-Good-Defaults](#php-docker-good-defaults). You
can go check that out on a GitHub where I've shown in this example in a little
bit more detail.

#### Healthcheck in Postgres Dockerfile

> Use a PostgreSQL utility to test for ready state

```Dockerfile
FROM postgres

# Specify real uer with `-U` to prevent errors in log

HEALTHCHECK --interval=5s --timeout=3s \
    CMD pg_isready -U postgres || exit 1
```

Next example, we have Postgres example so in the Dockerfile I can use
a different URL. Here we have a Postgres application where in the Healthcheck
command, I'm using a command of `pg_isready`.

Now , with different apps, there's different tools,  With Postgres, it comse
with a built-in tool, that's a very simple testing of a connection to a Postgres
server. It doesn't validate that you have a good data, or that your database is
mounted properly, It's simply going to say, 'Does this database server allow
connections? Yes or no?'.

That's a neat one that you can do out of the box.

#### Healthcheck in Compose/Stack files

```yaml
version: "2.1" # minimum for Healthcheck
services:
    web:
        image: nginx
        healtcheck:
            test: ["CMD", "curl", "-f", "http://localhost" ]
            interval: 1m30s
            timeout: 10s
            retries: 3
            start_period: 1m # Version `3.4` minimum
```

Here's what it would look lie in a Composer/Stack file. very similar. You'll
notice that the `start_period` down there requires a different version.

Since the Healthcheck command came out in `1.12`, it was actually supported in
`2.1` of this Compose file. But if you gonna use the `start_period`, that means
you have to update your Compose file to version `3.4` in order to support that.
Because the `start_period` came out over a year later after the Healthcheck
command did.

### Jump Into Command Line

#### Healthcheck Docker run Example

What we're gonna do here is we're going to start a Postgres database server
without the Healthcheck, because by default, it doesn't come with one.

Then we're going to run it again with a manual Healthcheck command that will
add the command line, and we'll see the difference.
<br/>

![chapter-9-9.gif](./images/gif/chapter-9-9.gif "")
<br/>

```bash
$: docker container run --name p1 -d postgres
```

Here, we're just going to call the first one `p1`. We'll run it detached from
the official Postgres image.

```bash
$: docker container run --name p1 -d postgres:12.1
c30903858b04499501aeaa4ee6cf304fd9c3b1ef791c5a37fbd3cc70a7f53c3b

$: docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
c30903858b04        postgres:12.1       "docker-entrypoint.s…"   11 seconds ago      Up 7 seconds        5432/tcp            p1
```

You'll see that there's nothing indicating a Healthcheck here if I do `container
ls`. If we do that same command again, and call it `p2` this time, we're going
to add a `health` command.

This time, we're going to use the `pg_isread`, which we talked about earlier, to
test that the connections are available on this Postgres server. We're going to
tell it that the `-U` _user_ we need is the Postgres user. We don't actually need
to give it a password. It's not going to try log in. It's just going to try to
validate. We'll use the Postgres image.

```bash
$: docker container run --name p2 -d  --health-cmd="pg_isready -u postgres || exit 1" postgres:12.1
734d5fa6e8b4f460612cd0d8bb53ca84ee11d17d20fcf77d3c3df6ca2da026f2
```

Now if we do a `container ls`,

```
$: docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
734d5fa6e8b4        postgres:12.1       "docker-entrypoint.s…"   5 seconds ago       Up 2 seconds (health: starting)   5432/tcp            p2
c30903858b04        postgres:12.1       "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes                      5432/tcp            p1

$: docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
e74b6cb647a4        postgres:12.1       "docker-entrypoint.s…"   34 seconds ago      Up 31 seconds (healthy)   5432/tcp            p2
c30903858b04        postgres:12.1       "docker-entrypoint.s…"   28 minutes ago      Up 28 minutes             5432/tcp            p1
```

You will see that it says 'up 2 second (health: is starting)'. Now we get this
additional feature in our status of our `ls` command. It will stay in there
starting state for the default `30` second until it runs the Healthcheck command
for the first time.

If we do a `inspect` command on `p2` we'll see at the very top of `inspect`
result, that we had this new `Health` status options.

```bash
$: docker container inspect p2
[
...
...
"State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 2863182,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-10-07T11:34:51.410095546Z",
            "FinishedAt": "0001-01-01T00:00:00Z",
            "Health": {                                     # New status options
                "Status": "healthy",
                "FailingStreak": 0,
                "Log": [
                    {
                        "Start": "2020-10-07T18:35:52.063752534+07:00",
                        "End": "2020-10-07T18:35:52.568437609+07:00",
                        "ExitCode": 0,
                        "Output": "/var/run/postgresql:5432 - accepting connections\n"
                    },
                    {
                        "Start": "2020-10-07T18:36:22.673843466+07:00",
                        "End": "2020-10-07T18:36:23.311736153+07:00",
                        "ExitCode": 0,
                        "Output": "/var/run/postgresql:5432 - accepting connections\n"
                    },
                    {
                        "Start": "2020-10-07T18:36:53.441763701+07:00",
                        "End": "2020-10-07T18:36:54.008824516+07:00",
                        "ExitCode": 0,
                        "Output": "/var/run/postgresql:5432 - accepting connections\n"
                    },
                    {
                        "Start": "2020-10-07T18:37:24.107659664+07:00",
                        "End": "2020-10-07T18:37:24.661505481+07:00",
                        "ExitCode": 0,
                        "Output": "/var/run/postgresql:5432 - accepting connections\n"
                    },
                    {
                        "Start": "2020-10-07T18:37:54.780321757+07:00",
                        "End": "2020-10-07T18:37:55.31158842+07:00",
                        "ExitCode": 0,
                        "Output": "/var/run/postgresql:5432 - accepting connections\n"
                    }
                ]
            }
        },
...
...
]
```

In this case, I've only been able to run it four times. you can see the `output`

```bash
"Output": "/var/run/postgresql:5432 - accepting connections\n"
```

That it's showing it's accepting connections.

#### Healthcheck on Service
<br/>

![chapter-9-10.gif](./images/gif/chapter-9-10.gif "Healthcheck on Service")
<br/>

All right. Let's do some `service create` command to that same database, in that
same test Healthcheck. What we'll see here when we do this is that there are
**_three different states_** that a service goes through on starting up.

It's _preparing_, which usually means it's downloading the image. It's
_starting_, which means it's executing the container and bringing it up. Then
it's _running_.

Without the Healthcheck command, the _starting_ and _running_ are very quick.
They're almost instantaneous. We'll see that here with a `docker service create`
command.

```bash
$: docker service create --name p1 postgres:12.1
dd8c20eas4scgkilyk6b2i1b7
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

Once it's done  preparing by downloading the image, you'll see that it goes
immediately from _starting_ to _running_, because there is no Healthcheck. It
doesn't have anything else to do other that _start_ the container and say, 'Yep,
the binary is running'.

But if we do that same command `service create` and call it `p2` like before,
and give it that same health command.

```bash
$: docker service create --name p2 --healt-cmd="pg_isready -U postgres || exit 1" postgres:1.12
```

We start this service with the Healthcheck command built in. What we'll see is
that it'll go from _preparing_ to _starting_, and it will sit at the _starting
state_ for the default `30` second until the first Healthcheck runs.

This is now the Docker Service expecting a healthy state before it considers
this service fully running. After `30` second is over, It'll shift to _running
state_. Then we get the last little verify there, just to make sure that it's
considered stable; And then we're done.

You can already see, out of the box, that with `services` as well as `services
update`, we're going to get this extra bonus of health concept if we use these
commands whenever we can.


### Miscellaneous

#### References

- [PHP-Docker-Good-Defaults](https://github.com/BretFisher/php-docker-good-defaults)
- [Healthcheck in Compose file](https://docs.docker.com/compose/compose-file/#healthcheck)
- [Healthcheck in Dockerfile](https://docs.docker.com/engine/reference/builder/#healthcheck)



**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

