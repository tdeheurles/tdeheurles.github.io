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
