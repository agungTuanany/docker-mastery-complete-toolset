# Swarm Basic Features

## Table of Contents

1. [Scaling Out With Overlay Networking](#scaling-out-with-overlay-networking)

<br/>


## Scaling Out With Overlay Networking
<br/>

![chapter-8-1.gif](./images/gif/chapter-8-1.gif "scaling out with overlay networking")
<br/>

Before we expand our `services` to tart running across a bunch of node and all
of the services talking to each other, let's go over a couple of new concepts
that Swarm brings to the table.

The first one is new _networking driver_ called **_Overlay_**, and you really
just create a network with `docker network create` command, and you put in an
option `--driver overlay`. What that's is basically creating a _Swarm-wide
bridge_ network where the containers across hosts on the same virtual network
can access each other kind of like they're on a _VLAN_.

The _overlay_ driver doesn't play a _huge amount in traffic coming inside_, as
it's trying to take a wholistic  Swarm view of the network so that's you're not
constantly messing around with networking setting on individual nodes.

You can also enable full network encryption using **_IPSec_** where it'll will
actually set up IPsec tunnels between all the different nodes of your Swarm.
But it's _off_ by default really just for performance reasons.

When you create your services, you can add them to no Overlay networks, or one
or more Overlay networks. It really depends on the design of your application.
You know a lot of traditional design would have the databases on a _backend
network_ and the Web servers on a _frontend network_. Then maybe you would have
an API between the two that would be on both networks, or something like that;
And you can totally do that in Swarm.

### Jump into Command Action

#### Docker create Postgres service
<br/>

![chapter-8-2.gif](./images/gif/chapter-8-2,gif "Docker create Postgres service")
<br/>

What I want to do is I want to show you what it would be like if we deployed the
Drupal example from a previous assignment with the Postgres Database, as
services, and then created an Overlay network for them to talk to each other.

First we need to create a network, so `docker network create` and this hasn't
changed for Swarm other the fact that we're going to use a new driver. I'm just
called `mydrupal`

```bash
Usage:  docker network create [OPTIONS] NETWORK

Create a network

Options:
  -d, --driver string        Driver to manage the Network (default "bridge")

docker@node1: docker network create --driver overlay my drupal

docker@node1: docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
d42e0acfbab5        bridge              bridge              local
4c754bfc3f37        docker_gwbridge     bridge              local
ba69be945f69        host                host                local
slgv4i5f2jdc        ingress             overlay             swarm
rnaclan8j849        mydrupal            overlay             swarm
e41facb30fac        none                null                local
```

The `ingress` one is there by default. You'll also see new one `docker_gwbridge`
because a Swarm, which is actually an outgoing network that we won't need to
mess.

Let's create our Postgres service.

> **NOTE**: Specific Host
>
> We can specific the service run on which node by add the options `--hostname`,
> if not specified the host the service will run in random node

```bash
docker@node1: docker service create --name psql --network mydrupal \
                --hostname node1 -e POSTGRES_PASSWORD=mypass  postgres
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

You'll notice here that we don't get the whole image downloading and all that,
because services can't be run in the foreground, because they have to go through
the orchestrator and scheduler. So, we can do `docker service ls`, and we can
see that one of one of them is running.

```bash
docker@node1: docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
nviq6nwu3wfj        psql                replicated          1/1                 postgres:latest
```

If we do a `docker service ps` on `psql` we can see that this

```bash
@docker@node1: docker service ps psql
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
tffvhhhw8zzj        psql.1              postgres:latest     node1               Running             Running 23 seconds ago
```

#### Docker create Drupal service
<br/>

![chapter-8-3.gif](./images/gif/chapter-8-3.gif "Docker create Drupal service")

```bash
docker@node1: docker service create --name drupal --network mydrupal \
                --hostname node2 \
                -p 80:80 drupal:8:8.10
```

While, the service created on background we can use a trick to `watch` command
line  in Linux to watch.

```bash
docker@node1: watch docker service ls
```

Basically, what it does it's rerunning a command over and over again. It's
installed by default. Now we can  list the task of the services by run command
`ps` in service, and see that Drupal is actually running on  `node2`.

```bash
docker@node1: docker service ps drupalwatch
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
tuuhu5zlzgju        drupal.1            drupal:8.8.10       node2               Running             Running 9 minutes ago
```

We have the database running on `node1`, we have the Drupal website running on
`node2`. The question in my head how they know to talk to each other? Well,
using the _service name_.

So we paste one nodes of the IP addresses, it should come up on the Drupal
install; And if I tell it the **_database host_**, just like when we were with
Compose, we made the service name in the Compose file `docker-compose.yml`, here
we're going to use the _service name_ that was created for database server. So,
that was `psql`,
<br/>

![chapter-8-1.png](./images/chapter-8-1.png "Drupal connected with Postgres with service name")
<br/>

So above screen is basically telling us that it's able to talk to database
across the `nodes` and  setup the system.

That's the great thing about _Overlay_ is it really just **_acts like everything
on the same subnet_**.

Note, that Drupal and Postgres running smoothly, but I have three nodes, so how
do I know which node it's going to be on And which port I need to make sure that
my DNS points to when I create this website name?

In my case, I have three local IP addresses on my nodes, and I can copy all
three of them and put all three of those IP addresses in the browser. If you
see, it appears like the website is running on all three nodes because I'm
looking at all three IP addresses. But if I do `docker service ps`

```bash
docker@node1: docker service ps drupal
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
tuuhu5zlzgju        drupal.1            drupal:8.8.10       node2               Running             Running 33 minutes ago
```
I know It's running on `node2`. I can do a `docker service inspect`, and see
that indeed it's only got one IP address on that overlay network

```bash
docker@node1: docker service inspect drupal
[
    {
        "Endpoint": {
            "Spec": {
                "Mode": "vip",
                "Ports": [
                    {
                        "Protocol": "tcp",
                        "TargetPort": 80,
                        "PublishedPort": 80,
                        "PublishMode": "ingress"
                    }
                ]
            },
            "Ports": [
                {
                    "Protocol": "tcp",
                    "TargetPort": 80,
                    "PublishedPort": 80,
                    "PublishMode": "ingress"
                }
            ],
            "VirtualIPs": [
                {
                    "NetworkID": "slgv4i5f2jdc7y8fqor8bqhwz",
                    "Addr": "10.0.0.7/24"
                },
                {
                    "NetworkID": "koef15cr8saw05y2dirtz2inn",       >> The ID is point to mydrual network ID
                    "Addr": "10.0.2.5/24"
                }
            ]
        }
    }
]

docker@node1: docker network ls | grep overlay
NETWORK ID          NAME                DRIVER              SCOPE
slgv4i5f2jdc        ingress             overlay             swarm
koef15cr8saw        mydrupal            overlay             swarm
```

So, why is it responding on all three hosts? That lead us to next new features.

**[⬆ back to top](#table-of-contents)**
<br/>
<br/>
