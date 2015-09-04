---
published: true
layout: post
title: How to install docker on windows
categories:
  - Tutorial
tags:
  - docker
  - windows
  - cygwin
---

In this post we will see how to install docker on windows. We will also configure PowerShell as Cygwin have an issue for the moment.

### Updated
- 13/07 : Added volume informations
- 14/07 : Added issue with cygwin
- 20/07 : Added Virtual Box 5 informations
- 04/09 : Toolbox has replaced boot2docker. The post will be updated soon. The extraction of the env into cmd or powershell still works with toolbox.

### Problem with cygwin

- **cygwin** : there is [an opened issue](https://github.com/docker/docker/issues/12469) with it, for now you cannot use tls. So you can start container but not exec/-ti with it.

### Some informations before starting

- This post is an extraction of informations from these documents : [docker](https://tdeheurles/docs/blob/master/docker) and [cygwin](https://tdeheurles/docs/blob/master/cygwin)
- Virtual Box 4 doesn't work on windows10 (May 2015), Virtual Box 5 works but non officially
- These tests are done on Windows 7
- boot2docker works on VirtualBox 5
- Linux and windows does not use the same file system format. This result in difference with file ownership. It can make some problems running code.
- Links are written at the end of the post to help for some issues

### installing boot2docker

- Go to [boot2docker](http://boot2docker.io/) and download the installer (127Mo). Version was 1.7 at the writing time.
- Run the installer
- You should have a `Boot2Docker Start` shortcut on desktop, run it.
- It will create SSH keys, start the VM and give a prompt
- now just test that all is fine with :

 ```bash
 docker ps
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
 ```

##### PowerShell

A `docker` command should return the help and a `docker ps` should fail. The first tells that docker client (boot2docker) is correctly installed, the second shows that the docker daemon (inside the VM) is not reachable from here (VM is not running or client cannot access for other reason).

To run the VM you can :

- clic on `boot2docker start`
- launch VirtualBox and start the VM.
- run `boot2docker up` in a terminal (boot2docker need to be in the PATH)

To tell where is the VM, run the following :

```
boot2docker shellinit | Invoke-Expression
```

To enable docker on each instance of PowerShell :

```
# create your power shell profile
new-item -itemtype file -path $profile -force
```

and add this code to it :

```
boot2docker shellinit | Invoke-Expression
```

### Make a try

```
PS C:\Users\username> docker run hello-world
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
PS C:\Users\username> docker run -ti debian bash
```

And now run a debian (~ 80 mb) :

```
root@6fc97235ee57:/# exit
PS C:\Users\username>
```

### Volume sharing

Volume sharing is a little special with boot2docker because we use a docker installed on a VM. So files shared by the run volume option are from VM to container : `-v VirtualMachine:Container`.

boot2docker already share your c:\Users folder to the VM path `/c/Users`

```bash
docker run -ti -v /c/Users/username/repository:/repository debian bash
root@dc5c39b4a0c9:/# cd /repository/
root@dc5c39b4a0c9:/repository# ls
CLI-tools       coreos-vagrant         jenkins-cluster          kubernetes-vagrant-coreos-cluster  ws-backend
Epsilon-Web-UI  docs                   jvm-tools                nginx                              ws-front
HomeKube        edge                   k8s-base                 packer-example                     ws-wssproxy
NonGithub       galliasphere           kiwi                     packer-qemu-templates
Ubuntu          gcloud-packer-example  kubeos                   packer-templates
backend         gcloud-tools           kubernetes               tdeheurles.github.io
cloud-rnd       jekyll-now             kubernetes-docker-files  weave-kubernetes
root@dc5c39b4a0c9:/repository# exit
```

The sync is a to way automatically updated. So you can work with your editor in windows and run your containers with updated code.

Look at the [official boot2docker documentation](https://github.com/boot2docker/boot2docker/blob/master/README.md#folder-sharing). Personnaly I don't like the samba solution for now as the VM sharing seems so easy like that.  
The documentation also say that is subject to change, so I won't go further.

### Links

- [error fix](https://x86x64.wordpress.com/2015/05/03/docker-on-windows-fata0021-an-error-occurred-trying-to-connect/) on docker startup
- [this post](https://developer.ibm.com/bluemix/2015/04/16/installing-docker-windows-fixes-common-problems/) fix lots of common issues
- [An opened issue for Cygwin tls](https://github.com/docker/docker/issues/12469)
