---
published: true
layout: post
title: Exploring Systemd
categories:
  - Tutorial
tags:
  - CoreOS
  - systemd
---

systemd is an init system that provides many powerful features for starting, stopping and managing processes. Within the CoreOS world, you will almost exclusively use systemd to manage the lifecycle of your Docker containers.

## Unit files

Systemd read from `unit files` in `/etc/systemd/system`.

Here is an example :

```ini
[Unit]
Description=MyApp
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill busybox1
ExecStartPre=-/usr/bin/docker rm busybox1
ExecStartPre=/usr/bin/docker pull busybox
ExecStart=/usr/bin/docker run --name busybox1 busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"

[Install]
WantedBy=multi-user.target
```

- `Description` help to store informations
- `After/Requires` tells to wait until
- `ExecStart` is the process that will be monitored by systemd, often, docker run go there.
- `WantedBy` is the target that this unit is a part of.
- `=-` is used to ignore the error and continue the process

So we can write it to `/etc/systemd/system/hello.service`.

We start it :

```bash
➜ sudo systemctl enable /etc/systemd/system/hello.service
Created symlink from /etc/systemd/system/multi-user.target.wants/hello.service to /etc/systemd/system/hello.service.
➜ sudo systemctl start hello.service
```

Control that it's running :

```bash
➜ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
ac697ef16657        busybox             "/bin/sh -c 'while t   19 seconds ago      Up 19 seconds                           busybox1
➜ systemctl status hello
● hello.service - MyApp
   Loaded: loaded (/etc/systemd/system/hello.service; enabled; vendor preset: disabled)
   Active: active (running)
   # [...]
➜ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
ac697ef16657        busybox             "/bin/sh -c 'while t   43 seconds ago      Up 42 seconds                           busybox1
➜ journalctl -f -u hello.service
-- Logs begin at Mon 2015-07-20 11:53:08 UTC. --
Jul 20 13:08:17 core docker[1552]: Hello World
```

## Some systemctl commands :

- `systemctl` to see all services
- `systemctl status` to see a summary of all services status
- `systemctl status name.service` to te see a service
- `systemctl enable name.service` to make it run at startup
- `systemctl start name.service`
- `systemctl stop name.service`
- `systemctl reload name.service`

## Some [Unit] directives :

```
ExecStartPre 	 Commands that will run before ExecStart.
ExecStart 	   Main commands to run for this unit.
ExecStartPost  Commands that will run after all ExecStart commands have completed.
ExecReload 	   Commands that will run when this unit is reloaded via systemctl
                  reload foo.service
ExecStop 	     Commands that will run when this unit is considered failed or if
                  it is stopped via systemctl stop foo.service
ExecStopPost 	 Commands that will run after ExecStop has completed.
RestartSec 	   The amount of time to sleep before restarting a service.
                  Useful to prevent your failed service from attempting to
                  restart itself every 100ms.
```

[Full list here](http://www.freedesktop.org/software/systemd/man/systemd.service.html)

## Variables

### Unit specifiers

You can find a list of the variables [here](http://www.freedesktop.org/software/systemd/man/systemd.unit.html#Specifiers).

Unit variables starts with the `%` character

Just for an example `"%u"` is the `User name`

### Instantiated units

There is two specials variables that can refer to the file name of the unit when this one is written with a `@`, for example foo@bar.service

Then, %p will be foo and %i will be bar.

It seems coreos teams use this for port :

foo@123.service where 123 is the port.
