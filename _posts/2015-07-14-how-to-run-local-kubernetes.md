---
published: true
layout: post
title: How to run a local kubernetes
categories:
  - Tutorial
tags:
  - docker
  - linux
  - kubernetes
---

We will go through a quick install of kubernetes on top of docker.
We won't detail kubernetes so much in this post.  
We will use the [official docker getting started guide](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/docker.md).

This image shows what we are going to do :

![](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/k8s-singlenode-docker.png)

## prerequisites

You need docker.

- For windows, go [here]({% post_url 2015-07-12-How-to-install-docker-on-windows %})
- For linux, go [here](https://github.com/tdeheurles/docs/blob/master/docker)

## kubernetes services

Kubernetes run with 5 services :

- kube-apiserver
- kube-scheduler
- kube-controller-manager
- kubelet
- kube-proxy

It also needs etcd to run.

On cluster, we launch apiserver/ kube-scheduler/ kube-controller-manager on the master. Kubelet and kube-proxy run on each nodes.

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

This command will run the etcd container :

```
| COMMAND                                         | Info
|-------------------------------------------------|------------------------------
| docker run                                      | run a container
|   --net=host                                    | share network with host
|   -d                                            | as a daemon
|   gcr.io/google_containers/etcd:2.0.9           | from the google etcd image
|   /usr/local/bin/etcd                           | run this command at startup
|     --addr=127.0.0.1:4001                       |
|     --bind-addr=0.0.0.0:4001                    |
|     --data-dir=/var/etcd/data                   |
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
|   gcr.io/google_containers/hyperkube:v0.21.2    | from the hyperkube image
|   /hyperkube                                    | run this command at startup with args
|     kubelet                                     |    run kubelet
|       --api_servers=http://localhost:8080       |    address of apiserver
|       --v=2                                     |    verbose
|       --address=0.0.0.0                         |
|       --enable_server                           |
|       --hostname_override=127.0.0.1             |
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
|   gcr.io/google_containers/hyperkube:v0.21.2      | from the hyperkube image
|     /hyperkube                                    |   run this command at startup with args
|       proxy                                       |      run proxy
|       --master=http://127.0.0.1:8080              |         address of master
|       --v=2                                       |         verbose
```

## Run it

Go to a chosen folder and copy/paste this to a terminal to generate the scripts

```bash
cat <<EOF >> start.sh
# etcd
docker run                            \
  --net=host                          \
  -d                                  \
  gcr.io/google_containers/etcd:2.0.9 \
    /usr/local/bin/etcd               \
    --addr=127.0.0.1:4001             \
    --bind-addr=0.0.0.0:4001          \
    --data-dir=/var/etcd/data

# kubernetes
docker run                                     \
  --net=host                                   \
  -d                                           \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gcr.io/google_containers/hyperkube:v0.21.2   \
    /hyperkube                                 \
      kubelet                                  \
        --api_servers=http://localhost:8080    \
        --v=2                                  \
        --address=0.0.0.0                      \
        --enable_server                        \
        --hostname_override=127.0.0.1          \
        --config=/etc/kubernetes/manifests

# proxy
docker run                                   \
  -d                                         \
  --net=host                                 \
  --privileged                               \
  gcr.io/google_containers/hyperkube:v0.21.2 \
    /hyperkube                               \
      proxy                                  \
      --master=http://127.0.0.1:8080         \
      --v=2
EOF
cat <<EOF >> stop.sh
docker kill $(docker ps -aq)
EOF
chmod 755 start.sh stop.sh
```

Now you just have to :

```bash
./start.sh
```

And control that (after a download time), you have something like this :

```bash
➜  linux-k8s  docker ps
CONTAINER ID        IMAGE                                        COMMAND                CREATED             STATUS              PORTS               NAMES
f5e0f1708918        gcr.io/google_containers/hyperkube:v0.21.2   /hyperkube schedule    4 minutes ago       Up 4 minutes                            k8s_scheduler.b725e775_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_c27e095d
99685b4928cc        gcr.io/google_containers/hyperkube:v0.21.2   /hyperkube apiserve    4 minutes ago       Up 4 minutes                            k8s_apiserver.70750283_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_1c37abde
eef830aa7a6d        gcr.io/google_containers/hyperkube:v0.21.2   /hyperkube controll    4 minutes ago       Up 4 minutes                            k8s_controller-manager.aad1ee8f_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_be5ce858
8bdfe3cc9509        gcr.io/google_containers/pause:0.8.0         /pause                 4 minutes ago       Up 4 minutes                            k8s_POD.e4cc795_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_79dea59d
28760651e24e        gcr.io/google_containers/hyperkube:v0.21.2   /hyperkube proxy --    4 minutes ago       Up 4 minutes                            backstabbing_lovelace
663bfd8d97a5        gcr.io/google_containers/hyperkube:v0.21.2   /hyperkube kubelet     4 minutes ago       Up 4 minutes                            goofy_einstein
d549b25c97c4        gcr.io/google_containers/etcd:2.0.9          /usr/local/bin/etcd    4 minutes ago       Up 4 minutes                            dreamy_kilby
```

We can see here our services.

## Make it do something

To communicate with the kubernetes services, we can use CLI, API or SDK.  
We will use the CLI solution : `kubectl`.

You can [download](https://cloud.google.com/sdk/) it and install it, but we will here use a container for that.

Just copy/paste this in your terminal :

```bash
cat <<EOF >> run_kubectl.sh
docker run \
  -ti \
  --net=host \
  tdeheurles/gcloud-tools:latest \
  bash
EOF
chmod 755 run_kubectl.sh
./run_kubectl.sh
```

you should have now a prompt, and writing `kubectl` should give you the help.

So create a nginx container :

```bash
➜  linux-k8s  kubectl -s http://localhost:8080 run-container nginx --image=nginx --port=80
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
nginx        nginx          nginx      run=nginx   1
```

and add its services :

```bash
➜  linux-k8s  kubectl expose rc nginx --port=80
NAME      LABELS      SELECTOR    IP(S)     PORT(S)
nginx     run=nginx   run=nginx             80/TCP
```

after a short moment, `kubectl get services` should show you the IP where you can join the nginx. A `curl the_ip_of_service` should show you a nginx response.

```bash
➜  linux-k8s  kubectl get services
NAME         LABELS                                    SELECTOR    IP(S)        PORT(S)
kubernetes   component=apiserver,provider=kubernetes   <none>      10.0.0.1     443/TCP
nginx        run=nginx                                 run=nginx   10.0.0.208   80/TCP
➜  linux-k8s  curl 10.0.0.208
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href=http://nginx.org/>nginx.org</a>.<br/>
Commercial support is available at
<a href=http://nginx.com/>nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

cool ^^  
Our service is running ^^

## windows

I will post in a few days how to run this on windows.
