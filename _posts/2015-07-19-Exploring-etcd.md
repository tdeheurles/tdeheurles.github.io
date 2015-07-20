---
published: true
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


You can find :

- The [getting started page](https://coreos.com/etcd/)
- The [official github documentation](https://github.com/coreos/etcd).
- The [api documentation](https://coreos.com/etcd/docs/latest/api.html)

I will implicitly name etcd the etcd2 version for all the post.

If you want to deep in etcd, go to the official getting started page, else if you want just a quick look at most of the informations, you are at the good place.

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

When you define CoreOS cloud-config, here are the minimal informations needed for etcd2 to start :

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

## using etcd

You can use etcd with etcdctl (CLI) or with the HTTP API.  
NOte that the -L option is needed for CURL as there is redirection in etcd.

### Reading/writing from CoreOS

```bash
# etcdctl
➜ etcdctl set /message Hello
Hello
➜ etcdctl get /message
Hello

# curl
➜ curl -L -X PUT http://127.0.0.1:2379/v2/keys/message -d value="HelloFromCurl"
{"action":"set","node":{"key":"/message","value":"HelloFromCurl","modifiedIndex":5,"createdIndex":5},"prevNode":{"key":"/message","value":"Hello","modifiedIndex":4,"createdIndex":4}}
➜  curl -L http://127.0.0.1:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"HelloFromCurl","modifiedIndex":5,"createdIndex":5}}
```

You can add directory just adding a path to the POST :

```bash
➜ curl -L -X PUT http://127.0.0.1:2379/v2/keys/path/to/the/message -d value="HelloFromCurl"
{"action":"set","node":{"key":"/message","value":"HelloFromCurl","modifiedIndex":5,"createdIndex":5},"prevNode":{"key":"/message","value":"Hello","modifiedIndex":4,"createdIndex":4}}
```

You can just create a directory :

```bash
curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d dir=true
```

And see what is in a directory with ls :

```bash
# etcdctl
etcdctl ls

# curl
curl http://127.0.0.1:2379/v2/keys/foo_dir/foo -XPUT -d value=bar

# recursive ls
curl http://127.0.0.1:2379/v2/keys/?recursive=true
```

### Removing

```bash
# etcdctl
etcdctl rm /foo-service

# curl
curl -L -X DELETE http://127.0.0.1:2379/v2/keys/message

# empty directory
curl 'http://127.0.0.1:2379/v2/keys/foo_dir?dir=true' -XDELETE

# directory with value
curl http://127.0.0.1:2379/v2/keys/dir?recursive=true -XDELETE
```

### Reading/writing from container

The ETCD IP can't be 127.0.0.1, so you have to know it from host. The good one is the HOST address on docker0 interface.

You can programmatically grab it with :

```bash
ETCD_ENDPOINT="$(ifconfig docker0 | awk '/\<inet\>/ { print $2}'):2379"
```

### Index, PrevNode

- When you PUT a value on a key, an `index` is incremented each time.
- When you GET a key, if index is more than 1, the response will be with :
  - `node` : the current value
  - `prevNode` : the value just before this change

```bash
# to see a history of the messages
curl -s 'http://127.0.0.1:2379/v2/keys/queue?recursive=true&sorted=true'
```

### Watching and triggering from changes

```bash
# waiting
curl http://127.0.0.1:2379/v2/keys/foo?wait=true

# waiting from an index
curl 'http://127.0.0.1:2379/v2/keys/foo?wait=true&waitIndex=7'

# triggering
etcdctl exec-watch --recursive /foo-service -- sh -c 'echo "\"$ETCD_WATCH_KEY\" key was updated to \"$ETCD_WATCH_VALUE\" value by \"$ETCD_WATCH_ACTION\" action"' &
etcdctl set /foo-service/container3 localhost:2222
```

### TTL

```bash
# etcdctl
etcdctl set /foo "Expiring Soon" --ttl 20

# curl
curl -L -X PUT http://127.0.0.1:2379/v2/keys/foo?ttl=20 -d value=bar

# directory ttl
curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d ttl=30 -d dir=true

# update directory ttl
curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d ttl=30 -d dir=true -d prevExist=true
```

Watcher will receive a message with `action` set to `expire`

### Hidden node

The key here is the `_`

```bash
curl http://127.0.0.1:2379/v2/keys/_message -XPUT -d value="Hello hidden world"
```

hidden keys won't be listed with GET, you need to explicitely ask for.

### files

You can store small files with etcd (configuration for example)

```bash
curl http://127.0.0.1:2379/v2/keys/afile -XPUT --data-urlencode value@afile.txt
```

### statistics

etcd store statistics about what is going on, look at the end of the API for more details.

Just an example :

```bash
curl http://127.0.0.1:2379/v2/stats/store
```
