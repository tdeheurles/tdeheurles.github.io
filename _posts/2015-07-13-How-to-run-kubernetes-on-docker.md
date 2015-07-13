---
published: false
layout: post
title: How to install docker on windows
categories:
  - Tutorial
tags:
  - docker
  - windows
  - kubernetes
---

In this tutorial, we will run kubernetes on our host. Everything will run in containers so we do not need to install anything else than docker.  
To run docker on windows, you can follow this [post]({% post_url 2015-07-12-How-to-install-docker-on-windows %}).

This post follow one of the official kubernetes `getting starting guide`. You can find it [here](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/docker.md).

This image shows what we are going to do :

![](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/k8s-singlenode-docker.png)

## presentation

### etcd

etcd is a distributed, consistent key-value store for shared configuration and service discovery, with a focus on being:

- Simple: curl'able user facing API (HTTP+JSON)
- Secure: optional SSL client cert authentication
- Fast: benchmarked 1000s of writes/s per instance
- Reliable: properly distributed using Raft

etcd is written in Go and uses the Raft consensus algorithm to manage a highly-available replicated log.

You can find the official documentation [here](https://github.com/coreos/etcd).

---

Here is the command to run the etcd container

```
| COMMAND                                         | Info
|-------------------------------------------------|------------------------------
| docker run                                      | run a container
|   --net=host                                    | share network with host
|   -d                                            | as a daemon
|   gcr.io/google_containers/etcd:2.0.13          | from the google etcd image
|   /usr/local/bin/etcd                           | run this command at startup
|     --addr=127.0.0.1:4001                       | ???
|     --bind-addr=0.0.0.0:4001                    | ???
|     --data-dir=/var/etcd/data                   | ???
```

run it :

```bash
docker run --net=host -d gcr.io/google_containers/etcd:2.0.13 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data
```

### master and kubelet

this is four services :

- api-server          (master service)
- scheduler           (master service)
- controller-manager  (master service)
- kubelet             (node service)

```
| COMMAND                                         | Info
|-------------------------------------------------|---------------------------------------
| docker run                                      | run a container
|   --net=host                                    | share network with host
|   -d                                            | as a daemon
|   -v /var/run/docker.sock:/var/run/docker.sock  | share a volume host:container
|   gcr.io/google_containers/hyperkube:v0.18.2    | from the hyperkube image
|   /hyperkube                                    | run this command at startup with args
|     kubelet                                     |    run kubelet
|       --api_servers=http://localhost:8080       |    address of apiserver
|       --v=2                                     |    verbose
|       --address=0.0.0.0                         |    ???
|       --enable_server                           |    ???
|       --hostname_override=127.0.0.1             |    ???
|       --config=/etc/kubernetes/manifests        |    manifests path for configuration
```

### kube proxy

```
| COMMAND                                           | Info
|---------------------------------------------------|-----------------------------------------
| docker run                                        | run a container
|   -d                                              | as a daemon/service
|   --net=host                                      | share network with host
|   --privileged                                    | root
|   gcr.io/google_containers/hyperkube:v0.18.2      | from the hyperkube image
|     /hyperkube                                    |   run this command at startup with args
|       proxy                                       |      run proxy
|       --master=http://127.0.0.1:8080              |         address of master
|       --v=2                                       |         verbose
```

###

## code

This is a resume of the code, everything should already be running.

```bash
docker run --net=host -d gcr.io/google_containers/etcd:2.0.13 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data
# Start etcd

# start kubernetes
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock  gcr.io/google_containers/hyperkube:v0.18.2 /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests
docker run -d --net=host --privileged gcr.io/google_containers/hyperkube:v0.18.2 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
```

```bash
docker run --net=host -ti tdeheurles/gcloud-tools bash
```
