# Swarm Lifecycle

## Table of Contents
1. [Using Secrets with Local Docker Compose](#using-secrets-with-local-docker-compose)


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
