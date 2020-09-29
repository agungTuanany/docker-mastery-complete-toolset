# Swarm Basic Features

## Table of Contents

1. [Scaling Out With Overlay Networking](#scaling-out-with-overlay-networking)
2. [Scaling Out with Routing Mesh](#scaling-out-with-routing-mesh)

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
do I know which node it's going to be on and which port I need to make sure that
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

## Scaling Out with Routing Mesh
<br/>

![chapter-8-4.gif](./images/gif/chapter-8-4.gif "Scaling out with routing mesh")
<br/>

So that little bit of magic wasn't actually magic. It was the **_Routing
Mesh_**. The Routing Mesh is an _incoming or ingress network that distributed
packets for our service to the task for that service_, because we can have more
that one task.

This actually spans (reach out) all the nodes, and it's using Kernel primitives
that have been around a while actually called [IPVS](#what-is-ipvs). So we're
not talking about fancy new service. We're really just talking about the core
features of the Linux Kernel. What it's really doing here is it's _load
balancing_ across all the nodes and _listening on all the nodes traffic_.

There's a couple of ways we can talk about how this works. The **_first
example_** us from _one container to another_. If our backed system, like say
the database, were increased to _two replicas_, the frontends talking to the
backends, wouldn't actually talk directly to their IP address. They would talk
something called a [VIP](#what-is-vip) or a _Virtual IP_, that Swarm puts in
front of all services. This is a private IP inside the virtual networking of
Swarm, and it ensures that the load is distributed among all the task for
a service.

So if you can imagine if you had a worker role in your application, and it had
10 different containers, you don't have to put a [load balancer](#what-is-load-balancer)
in front of that. This does that for you. When you're talking about traffic from
one service inside your virtual network talking to another service inside your
virtual network.

The **_second example_** example of how the Routing Mesh work is external
traffic coming into your Swarm can actually choose to hit any of this nodes in
your swarm. Any other worker nodes are going to have that _published port open_
and listening for that container's traffic, and then it will reroute that
traffic to the proper container based on its load balancing.

What this means is when you're deploying containers in a Swarm, you're not
supposed to have to care about what server it's on because it might move, right?
If a container fails, and the task is recreated by Swarm, it might put that on
a different node; And you certainly don't want to have to change your firewall
around or your DNS settings to make that container work again.

Th Routing Mesh solves a lot of those problems by allowing our Drupal website on
port `80` to be accessible from any node in the Swarm; And in the background,
it's taking those packets from that server and then routing them to the
container. If it's on a different node, in it'll route it over the virtual
networks. If it's on the same node, it'll just reroute it to the port of that
container; And we didn't have to do anything to enable this. This was all out of
the box.

Let see diagram how it might works.

#### Example-1 Routing Mesh: one container to another
<br/>

![chapter-8-1.gif](./images/chapter-8-2.png "Scaling out with Routing Mesh: one container to another")
<br/>

If I created a new Swarm service, and I told it to have three replicas; And it
created three tasks, with three containers, on three nodes. Inside hat Overlay
network, it's actually creating a virtual IP that's mapped to DNS name of the
service, right? And the service, by default, the DNS name, is the name of the
service. In this case, I created a service called `my-web`, and any other
containers I have in my Overlay networks that need to talk to that service
inside the Swarm, they only have to worry about using the _my-web DNS_.  The
virtual IP properly load bounces the traffic amongst all the tasks in that
service.

This isn't DNS Round-Robin. That's actually a slightly different configuration.
We could enable that if we wanted to. There's an options to use DNS Round-Robin.
The benefits of [VIPs](#what-is-vip) over Round-Robin is that a lot of times our
DNS caches inside our apps prevent us from properly distributing the load.
Rather than fight with our DNS clients in DNS configuration, we're just relying
on the VIP, which is kind of like you would have if you bought a dedicated
hardware load balancer.

#### Example-2 Routing-Mesh: Working with external traffic
<br/>

![chapter-8-3.png](./images/chapter-8-3.png "Scaling out with Routing Mesh: working with external trafic")
<br/>

In second example, this actually showing what if would be like with external
traffic coming in. This is similar to what we just did with Drupal, where when
I created those yellow boxes, by creating on service, called _my-web_, and it
created two tasks, and applied them to two different nodes, each one of those
nodes has a built-in load balancer on the external IP address. For me, because
I'm using DigitalOcean, that IP address is the one that DigitalOcean gave me.

When I use `-p` and published it on a port, in this example it's using port
`80:80`, any traffic that comes in to any of these three nodes hits that load
balancer on port `80:80`. The load balancer decides which container should get
the traffic and whether or not that traffic on the local node, or it needs to
send the traffic over the network to the different node.

Again, this actually all happens in the background without any special effort on
your part.

### Jump into Command Action
<br/>

![chapter-8-5.gif](./images/gif/chapter-8-5.gif "Routing mesh in termnial")

All right. Let's see this Routing Mesh in action. We already saw the example
with Drupal and how it listens on all three nodes. But what if we had multiple
tasks that see that load balancer working.

If we do `docker service create`, and we use Elasticsearch container.

```bash
docker@node1: docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2
```

While the service is creating, I'll just mention that Elasticsearch is actually
a search database that's accessible via a JSON web API. So it's really easy to
hit which `curl` and give us good examples of how this works.

I do a `docker service ps search`

```bash
docker@node1: docker service ps search
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
xiw6cpgx9ju5        search.1            elasticsearch:2     node3               Running             Running 56 seconds ago
qsm84uflkp8s        search.2            elasticsearch:2     node2               Running             Running 36 seconds ago
gb597wwvib6a        search.3            elasticsearch:2     node1               Running             Running 43 seconds ago
```

It's smartly created each task on a different node. If I just do a `curl` on my
`localhost`, on `node1` on port `9200` because I published that port

```bash
docker@node1:~$ curl localhost:9200
{
  "name" : "Timeslip",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "4zf-MahqS1qYjgkrmeAVFw",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
```

I'll get back the Elasticsearch basic information. Part of that is it will
actually create a random `name` that just a feature out of Elasticsearch. If you
`curl` several times then you can get  a random `name`, and then it'll start
repeating itself  like such. That's actually he virtual IP acting as a load
balancer and distributing my load across the three tasks.

### Note on Routing Mesh

In `17.03`, which is the release that I'm, showing you this on, the Routing Mesh
and the load balancing are currently stateless load balancer. If you've ever
dealt with State inside of maybe Amazon's classic load balancer, or other load
balancer technologies, you know what this is about. This is basically saying
that if you have to use session cookies on your application, or it expects
a consistent container to be talking to a consistent client, then you may need
to add other things to help solve that problem.

Out of the box, every time you hit a service with multiple tasks, it's going to
give you potentially a different result.

Also, if you get into the details of this, it's actually a layer-3 load
balancer, and that actually operates at the IP and port layer. It doesn't
actually operate the DNS layer.

If you've have ever run multiple websites on the same port, on the same server,
this isn't going to do that yet. You're still going to need another piece of the
puzzle on top of that if you're actually wanting to run multiple websites on the
same port, on the same Swarm.

Luckily, that's a pretty common request, and there's several options you can do
to solve both on these problems. One of them to use Nginx or HAProxy, which
there are pretty good examples out there of containers that will sit in front
with your Routing Mesh, and actually act as a stateful load balancer or a layer
for load balancer, that can also do caching and lots of other things. If you
need that,you might want to check of those out in the resources of this section.

I should also mention that if you were o pay for a subscription of Docker
Enterprise edition, with it comes somethings called UCP or Docker Data Center,
which is a web interface. But I should mention that Docker Enterprise edition,
If you get a subscription to that for your Swarm nodes, it actually comes with
a built-in layer for _web-proxy_ that allows you to just throw DNS names in the
_web config_ of our Swarm services and everything just works.

### Miscellaneous

#### What is IPVS

IPVS (IP Virtual Server) implements transport-layer load balancing, usually
called Layer 4 LAN switching, as part of the Linux kernel.It's configured via
the user-space utility tool.. IPVS is incorporated into the Linux Virtual Server
(LVS), where it runs on a host and acts as a load balancer in front of a cluster
of real servers. IPVS can direct requests for TCP- and UDP-based services to the
...  [wiki](http://en.wikipedia.org/wiki/IP_Virtual_Server)

Controls how ipvs will deal with connections that are detected port reuse. It is
a bitmap, with the values being: 0: disable any special handling on port reuse.
The new connection will be delivered to the same real server that was servicing
the previous connection. This will effectively disable expire_nodest_conn...
[kernel.org](http://www.kernel.org/doc/html/latest/networking/ipvs-sysctl.html)

#### What is VIP

Virtual IP address - Wikipedia A virtual IP address (VIP or VIPA) is an IP
address that doesn't correspond to an actual physical network interface. Uses
for VIPs include network address translation (especially, one-to-many NAT)...
[wiki](http://en.wikipedia.org/wiki/Virtual_IP_address)

#### What is load balancer

Load balancing is defined as the methodical and efficient distribution of
network or application traffic across multiple servers in a server farm. Each
load balancer sits between client devices and backend servers, receiving and
then distributing incoming requests to any available server capable of
fulfilling them.  [citrix](http://www.citrix.com/glossary/load-balancing.html)

Load balancing techniques can optimize the response time for each task, avoiding
unevenly overloading compute nodes while other compute nodes are left idle. Load
balancing is the subject of research in the field of parallel computers. Two
main approaches exist: static algorithms, which do not take into account the
state of the different ...
[wiki](http://en.wikipedia.org/wiki/Load_balancing_(computing))

A load balancer is a device that acts as a reverse proxy and distributes network
or application traffic across a number of servers. Load balancers are used to
increase capacity (concurrent users) and reliability of applications.
[source](http://www.f5.com/services/resources/glossary/load-balancer)



**[⬆ back to top](#table-of-contents)**
<br/>
<br/>

