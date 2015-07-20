---
published: false
layout: post
title: Exploring ETCD
categories:
  - Tutorial
tags:
  - CoreOS
  - etcd
---

etcd is a distributed, consistent key-value store for shared configuration and service discovery, with a focus on being:

- Simple: curl'able user facing API (HTTP+JSON)
- Secure: optional SSL client cert authentication
- Fast: benchmarked 1000s of writes/s per instance
- Reliable: properly distributed using Raft

etcd is written in Go and uses the Raft consensus algorithm to manage a highly-available replicated log.

You can find the official documentation [here](https://github.com/coreos/etcd).

I will implicitly name etcd the etcd2 version for all the post.

## Prerequisites
For this tutorial I will use the coreos etcd. CoreOS was 745.1.0 at the writing time.  
You can use a container if you want :

```bash
# etcd
docker run                            \
  --net=host                          \
  -d                                  \
  gcr.io/google_containers/etcd:2.0.9 \
    /usr/local/bin/etcd               \
      --addr=127.0.0.1:4001           \
      --bind-addr=0.0.0.0:4001        \
      --data-dir=/var/etcd/data
```

## CoreOS

*You can use the [coreos-vagrant](https://github.com/tdeheurles/coreos-vagrant) project to run this code.*

When you define CoreOS cloud-config, here are some informations needed for etcd2 to start :

```yaml
coreos:
  etcd2:
    # You NEED to define a discovery url. To get this one you have to GET this url :
    #    https://discovery.etcd.io/new?size=1 WITH the size feined !
    #    This size will be able to change after but it's a prerequisite for etcd to
    #    config
    # This discoveryUrl HAVE TO BE RENEWED for each new cluster. So if you vagrant destroy, you need
    #    to get a new token. CoreOS will not directly complain of that but etcd won't config
    discovery: https://discovery.etcd.io/<TOKEN>

    # The $private_ipv4 and public_ipv4 are understood by some provider like GCE, AWS or Vagrant.
    #    you cannot use them with baremetal for example
    # The port 2379/2380 correspond to etcd2, port 4001/7001 correspond to etcd1. We define them
    #    for backward availability
    advertise-client-urls: http://$private_ipv4:2379, http://$private_ipv4:4001
    initial-advertise-peer-urls: http://$private_ipv4:2380
    listen-client-urls: http://0.0.0.0:2379, http://0.0.0.0:4001
    listen-peer-urls: http://$private_ipv4:2380

  # And finally you need to run it
  units:
    - name: etcd2.service
      command: start
```
