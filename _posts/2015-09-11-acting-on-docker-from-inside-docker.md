---
published: true
layout: post
title: How to run docker command from inside a container
categories:
  - Tutorial
tags:
  - docker
---

Here is a tiny post on how to access your docker daemon from inside a container.

You probably often see the share of the volune `/var/run/docker.sock`. It's the way to share your docker to the container. Not really hard ^^

So, for example :

```bash
host $ # run a ubuntu container sharing that sock
host $ docker run -v /var/run/docker.sock:/var/run/docker.sock -ti ubuntu bash

container $ # update apt-get and install curl
container $ apt-get update && apt-get install curl
container $ # [... lots of log ...]

container $ # install docker
container $ curl -sSL https://get.docker.com/ | sh
container $ # [... lots of log ...]

container $ # look to the running container
container $ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                               NAMES
c1170902690a        ubuntu                       "bash"                   4 minutes ago       Up 4 minutes
                                jolly_brahmagupta
container $ # commit himself
container $ docker commit c117 dubuntu
container $ # leave himself
container $ docker kill c117

host $
```

This command is very simple but offer lots of possibilities.