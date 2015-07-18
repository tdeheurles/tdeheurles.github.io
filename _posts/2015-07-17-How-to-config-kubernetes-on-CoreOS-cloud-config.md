---
published: false
layout: post
title: How to config kubernetes on CoreOS
categories:
  - Tutorial
tags:
  - CoreOS
  - kubernetes
---

This post look at the way to configure cloud-config files for automating kubernetes run on the start of the CoreOS.  
We will start with a simple cloud-config and then add everything needed. It will be an interactive go-throught as I show every step and why we need it.

## Initial informations

Here is the initial config file, I took it from [how to install coreos on bare metal]({% post_url 2015-07-16-How-to-install-coreos-on-bare-metal %}):

```yaml
#cloud-config

users:
  - name: core
    passwd: $password
    groups:
      - sudo
      - docker

coreos:
  etcd:
    name: node01
    discovery: $discoveryUrl

hostname: node01
```

We also know that we can run kubernetes with this code from [how to run local kubernetes]({% post_url 2015-07-14-How-to-run-local-kubernetes %}):

```bash
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
```

## cloud-config update

So just ask CoreOS to manage this containers.

[Here](https://coreos.com/fleet/docs/latest/launching-containers-fleet.html) is official CoreOS documentation on how to run containers with fleet.  
As a reminder, `fleet` is the cluster version of `systemd`. Systemd is a services manager for linux. It's used in the Ubuntu 15.04. Here in CoreOS, systemd manage the node services and is managed by fleet.  

First, let resume the official document :

- There is two types of Fleet Units : **Standard** and **Global**
- **Standard units** run on one node, if the node stop for any reason, the Units is restarted on another node
- **Global units** are started on all nodes at the same time.

Just have a look to our fleetctl :

```bash
➜  ~  fleetctl list-unit-files
Error retrieving list of units from repository: Get http://domain-sock/fleet/v1/units?alt=json: dial unix /var/run/fleet.sock: no such file or directory
➜  ~  fleetctl list-units
Error retrieving list of units from repository: Get http://domain-sock/fleet/v1/state?alt=json: dial unix /var/run/fleet.sock: no such file or directory
➜  ~  systemctl status fleet
* fleet.service - fleet daemon
   Loaded: loaded (/usr/lib64/systemd/system/fleet.service; static; vendor preset: disabled)
   Active: inactive (dead)
➜  ~  systemctl status fleetctl
* fleetctl.service
   Loaded: not-found (Reason: No such file or directory)
   Active: inactive (dead)
```

So we need to add it to our cloud-engine.  
Look at [this coreos documentation](https://coreos.com/os/docs/latest/cloud-config.html) on how to run fleet. We have to add the following to the cloud-config :

```yaml
coreos:
  units:
    - name: fleet.service
      command: start
```

So just create a cloud-config with that in it, and ask for update :

```bash
➜  cat <<EOF >> cloud-config.yaml
#cloud-config

coreos:
  units:
    - name: fleet.service
      command: start
EOF
➜  sudo coreos-cloudinit --from-file cloud-config.yml
Checking availability of "local-file"
Fetching user-data from datasource of type "local-file"
Fetching meta-data from datasource of type "local-file"
2015/07/17 09:20:49 Parsing user-data as cloud-config
Merging cloud-config from meta-data and user-data
2015/07/17 09:20:49 Ensuring runtime unit file "etcd.service" is unmasked
2015/07/17 09:20:49 Ensuring runtime unit file "etcd2.service" is unmasked
2015/07/17 09:20:49 Ensuring runtime unit file "fleet.service" is unmasked
2015/07/17 09:20:49 Ensuring runtime unit file "locksmithd.service" is unmasked
2015/07/17 09:20:49 Calling unit command "start" on "fleet.service"
2015/07/17 09:20:49 Result of "start" on "fleet.service": done
➜  cloud-config  systemctl status fleet
* fleet.service - fleet daemon
   Loaded: loaded (/usr/lib64/systemd/system/fleet.service; static; vendor preset: disabled)
   Active: active (running) since Fri 2015-07-17 09:20:49 UTC; 10s ago
 Main PID: 857 (fleetd)
   Memory: 6.8M
      CPU: 25ms
   CGroup: /system.slice/fleet.service
           -857 /usr/bin/fleetd

Jul 17 09:20:57 node01 fleetd[857]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 100ms
Jul 17 09:20:57 node01 fleetd[857]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127....refused
Jul 17 09:20:57 node01 fleetd[857]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127....refused
Jul 17 09:20:57 node01 fleetd[857]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 200ms
Jul 17 09:20:58 node01 fleetd[857]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127....refused
Jul 17 09:20:58 node01 fleetd[857]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127....refused
Jul 17 09:20:58 node01 fleetd[857]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 400ms
Jul 17 09:20:58 node01 fleetd[857]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127....refused
Jul 17 09:20:58 node01 fleetd[857]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127....refused
Jul 17 09:20:58 node01 fleetd[857]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 800ms
Hint: Some lines were ellipsized, use -l to show in full.
➜  cloud-config
```

Our service is on. Try to restart and look if still ok :

```bash
sudo reboot
➜  systemctl status fleet
* fleet.service - fleet daemon
   Loaded: loaded (/usr/lib64/systemd/system/fleet.service; static; vendor preset: disabled)
   Active: inactive (dead)
```

And no ... the cloud-init does not persist after restart.  
So we will try to update the cloud-config in /var/lib/coreos-install/user_data.

```bash
➜  ~  sudo vi /var/lib/coreos-install/user_data
➜  ~  sudo coreos-cloudinit --from-file /var/lib/coreos-install/user_data
Checking availability of "local-file"
Fetching user-data from datasource of type "local-file"
Fetching meta-data from datasource of type "local-file"
2015/07/17 10:00:08 Parsing user-data as cloud-config
Merging cloud-config from meta-data and user-data
2015/07/17 10:00:08 Set hostname to node01
2015/07/17 10:00:08 User 'core' exists, ignoring creation-time fields
2015/07/17 10:00:08 Setting 'core' user's password
2015/07/17 10:00:08 Writing drop-in unit "20-cloudinit.conf" to filesystem
2015/07/17 10:00:08 Writing file to "/run/systemd/system/etcd.service.d/20-cloudinit.conf"
2015/07/17 10:00:08 Wrote file to "/run/systemd/system/etcd.service.d/20-cloudinit.conf"
2015/07/17 10:00:08 Wrote drop-in unit "20-cloudinit.conf"
2015/07/17 10:00:08 Ensuring runtime unit file "etcd.service" is unmasked
2015/07/17 10:00:08 Ensuring runtime unit file "etcd2.service" is unmasked
2015/07/17 10:00:08 Ensuring runtime unit file "fleet.service" is unmasked
2015/07/17 10:00:08 Ensuring runtime unit file "locksmithd.service" is unmasked
2015/07/17 10:00:08 Calling unit command "start" on "fleet.service"'
2015/07/17 10:00:08 Result of "start" on "fleet.service": done
➜  ~  systemctl status fleet
* fleet.service - fleet daemon
   Loaded: loaded (/usr/lib64/systemd/system/fleet.service; static; vendor preset: disabled)
   Active: active (running) since Fri 2015-07-17 10:00:08 UTC; 24s ago
 Main PID: 1006 (fleetd)
   Memory: 6.9M
      CPU: 35ms
   CGroup: /system.slice/fleet.service
           `-1006 /usr/bin/fleetd

Jul 17 10:00:32 node01 fleetd[1006]: ERROR client.go:214: Unable to get result for {Update /_coreos.com/fleet/machines/32d...n 800ms
Jul 17 10:00:32 node01 fleetd[1006]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127...refused
Jul 17 10:00:32 node01 fleetd[1006]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127...refused
Jul 17 10:00:32 node01 fleetd[1006]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32d...n 100ms
Jul 17 10:00:32 node01 fleetd[1006]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127...refused
Jul 17 10:00:32 node01 fleetd[1006]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127...refused
Jul 17 10:00:32 node01 fleetd[1006]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32d...n 200ms
Jul 17 10:00:32 node01 fleetd[1006]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127...refused
Jul 17 10:00:32 node01 fleetd[1006]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127...refused
Jul 17 10:00:32 node01 fleetd[1006]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32d...n 400ms
Hint: Some lines were ellipsized, use -l to show in full.
➜  ~
```

The service have been started. Try after a restart :

```bash
sudo reboot

Last login: Fri Jul 17 09:28:06 2015 from 192.168.1.28
CoreOS alpha (745.1.0)
➜  ~  systemctl status fleet
* fleet.service - fleet daemon
   Loaded: loaded (/usr/lib64/systemd/system/fleet.service; static; vendor preset: disabled)
   Active: active (running) since Fri 2015-07-17 10:02:45 UTC; 18s ago
 Main PID: 618 (fleetd)
   Memory: 8.7M
      CPU: 33ms
   CGroup: /system.slice/fleet.service
           `-618 /usr/bin/fleetd

Jul 17 10:02:59 node01 fleetd[618]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 100ms
Jul 17 10:02:59 node01 fleetd[618]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127....refused
Jul 17 10:02:59 node01 fleetd[618]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127....refused
Jul 17 10:02:59 node01 fleetd[618]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 200ms
Jul 17 10:02:59 node01 fleetd[618]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127....refused
Jul 17 10:02:59 node01 fleetd[618]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127....refused
Jul 17 10:02:59 node01 fleetd[618]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 400ms
Jul 17 10:03:00 node01 fleetd[618]: INFO client.go:292: Failed getting response from http://localhost:4001/: dial tcp 127....refused
Jul 17 10:03:00 node01 fleetd[618]: INFO client.go:292: Failed getting response from http://localhost:2379/: dial tcp 127....refused
Jul 17 10:03:00 node01 fleetd[618]: ERROR client.go:214: Unable to get result for {Create /_coreos.com/fleet/machines/32de...n 800ms
Hint: Some lines were ellipsized, use -l to show in full.
```

Cool, it works.
Next time I will just add a step to control that the cloud-config is good.

Try to look to fleetctl units a new time :

```bash
➜  ~  fleetctl list-unit-files
Error retrieving list of units from repository: googleapi: Error 503: fleet server unable to communicate with etcd
```

Ok. We need etcd too. Just add it to cloud config.

```bash
mkdir -p ~/cloud-config
sudo cp /var/lib/coreos-install/user_data ~/cloud-config/cloud-config.yml
vi ~/cloud-config/cloud-config.yml
```

Add etcd service start to coreos units :

```yaml
- name: etcd.service
  command: start
```

Try to validate :

```bash
➜  ~  coreos-cloudinit -validate --from-file cloud-config/cloud-config.yml
Checking availability of "local-file"
Fetching user-data from datasource of type "local-file"
```

No error message, so lets cp to /var/lib/coreos-install/user_data, restart and look to our fleetctl status :

```bash
➜  ~  sudo cp ~/cloud-config/cloud-config.yml /var/lib/coreos-install/user_data
➜  ~  sudo reboot
➜  ~  fleetctl list-unit-files
UNIT    HASH    DSTATE  STATE   TARGET
➜  ~  fleetctl list-machines
MACHINE         IP              METADATA
32de502a...     192.168.1.13    -
➜  ~  fleetctl list-units
UNIT    MACHINE ACTIVE  SUB
➜  ~
```

We now can add our kubernetes containers. Lets do it.

## Add kubernetes to fleet units

We add the kubernetes Units to our cloud-config without the etcd container that is already running

```yaml
 - name: kubernetes.service
   command: start
   content: |
     [Unit]
     Description=Kubernetes master services
     Documentation=https://github.com/GoogleCloudPlatform/kubernetes
     Requires=etcd.service fleet.service docker.service
     After=etcd.service fleet.service docker.service

     [Service]
     ExecStart=/usr/bin/docker run --net=host -v /var/run/docker.sock:/var/run/docker.sock gcr.io/google_containers/hyperkube:v0.21.2 /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests

 - name: kube-proxy.service
   command: start
   content: |
     [Unit]
     Description=Kubernetes proxy service
     Requires=etcd.service fleet.service docker.service
     After=etcd.service fleet.service docker.service

     [Service]
     ExecStart=/usr/bin/docker run --net=host --privileged gcr.io/google_containers/hyperkube:v0.21.2 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
```

and look our coreos fleet after a reboot :

```bash
➜  ~  sudo cp ~/cloud-config/cloud-config.yml /var/lib/coreos-install/user_data
➜  ~  sudo reboot
```

After a quick look, it fails. The services are not started, complaining about the image not present. So I try manually

```bash
➜  ~ docker pull gcr.io/google_containers/hyperkube:v0.21.2
```

And reboot :

```bash
➜  ~  systemctl status kubernetes
* kubernetes.service - Kubernetes master services
   Loaded: loaded (/etc/systemd/system/kubernetes.service; static; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://github.com/GoogleCloudPlatform/kubernetes

Jul 17 10:57:34 node01 systemd[1]: Started Kubernetes master services.
Jul 17 10:57:34 node01 systemd[1]: Starting Kubernetes master services...
Jul 17 10:57:34 node01 docker[614]: f49526c18096442d74e26c5741941b10fdf2f1ec51972cc67d9599720b1b3ed8
➜  ~  docker ps
CONTAINER ID        IMAGE                                        COMMAND                CREATED              STATUS              PORTS               NAMES
5cc24100fba7        gcr.io/google_containers/hyperkube:v0.21.2   "/hyperkube schedule   About a minute ago   Up About a minute                       k8s_scheduler.b725e775_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_5701e473
01b706edef03        gcr.io/google_containers/hyperkube:v0.21.2   "/hyperkube apiserve   About a minute ago   Up About a minute                       k8s_apiserver.70750283_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_a46f2c2e
7462dc1cb47a        gcr.io/google_containers/hyperkube:v0.21.2   "/hyperkube controll   About a minute ago   Up About a minute                       k8s_controller-manager.aad1ee8f_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_ee3147a3
ba3df63a12d9        gcr.io/google_containers/pause:0.8.0         "/pause"               About a minute ago   Up About a minute                       k8s_POD.e4cc795_k8s-master-127.0.0.1_default_9b44830745c166dfc6d027b0fc2df36d_ea6d7d38
f49526c18096        gcr.io/google_containers/hyperkube:v0.21.2   "/hyperkube kubelet    About a minute ago   Up About a minute                       berserk_lovelace
➜  ~  docker run -ti --net=host tdeheurles/gcloud-tools bash
root@node01:/# kubectl cluster-info
Kubernetes master is running at http://localhost:8080
```

Kubernetes is running but is not managed by fleet/systemctl.  
So first lets resolve this pull problem :
