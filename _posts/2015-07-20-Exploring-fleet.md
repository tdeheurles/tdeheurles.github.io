---
published: false
layout: post
title: Exploring Fleet
categories:
  - Tutorial
tags:
  - CoreOS
  - fleet
---

fleet is a cluster manager that controls systemd at the cluster level. To run your services in the cluster, you must submit regular systemd units combined with a few [fleet-specific properties](https://coreos.com/fleet/docs/latest/unit-files-and-scheduling.html).

As for etcd, this post attempt is a quick go through, look at the official doc for deep understanding.

## Summary

## Links

- [CoreOS-fleet official documentation](https://coreos.com/fleet/docs/latest/launching-containers-fleet.html)

## start fleet

Just add fleet.service to your coreos.units :

```yaml
- name: fleet.service
  command: start
```

After boot, look to sytemctl status fleet, it should be running.

```bash
systemctl status fleet
● fleet.service - fleet daemon
   Loaded: loaded (/usr/lib64/systemd/system/fleet.service; static; vendor preset: disabled)
   Active: active (running)
   # [...]
```

## Simple control commands
