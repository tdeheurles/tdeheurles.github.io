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

In this post we will see how to install docker on windows. We will also install Cygwin for a better interactive experience with oh-my-zsh.

### Some informations before starting
- This post is an extraction of informations from this documents : [docker](https://tdeheurles/docs/blob/master/docker) and [cygwin](https://tdeheurles/docs/blob/master/cygwin)
- Virtual Box doesn't seem to work on windows10 (May 2015)
- This tests are done on Windows 7
- Linux and windows does not use the same file system format. This result in difference with file ownership. It can make some problem running code.
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

##### cygwin
Personally I use the Cygwin for Oh-My-Zsh. I like to have the same tools on the OS.  
But for now, there is [an opened issue](https://github.com/docker/docker/issues/12469) with it.

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
PS C:\Users\shinmox>
```

### Links
- [error fix](https://x86x64.wordpress.com/2015/05/03/docker-on-windows-fata0021-an-error-occurred-trying-to-connect/) on docker startup
- [this post](https://developer.ibm.com/bluemix/2015/04/16/installing-docker-windows-fixes-common-problems/) fix lots of common issues
- [An opened issue for Cygwin tls](https://github.com/docker/docker/issues/12469)
