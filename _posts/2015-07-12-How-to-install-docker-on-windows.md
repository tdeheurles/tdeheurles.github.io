---
published: true
layout: post
title: How to install a good docker environment on windows
categories:
  - Tutorial
tags:
  - docker
  - windows
  - babun
  - cygwin
  - Oh-My-Zsh
---

In this post we will see how to install docker on windows. We will also look at how to work around to the cygwin `cannot enable tty mode on non tty input`.

### Updated

- 13/07: Added volume informations
- 14/07: Added issue with cygwin
- 20/07: Added Virtual Box 5 informations
- 04/09: Toolbox has replaced boot2docker. The post will be updated soon. The extraction of the env into cmd or powershell still works with toolbox.
- 10/09: fix to bash under windows issue
- 25/09: Added babun and removed the cygwin/ConEMU solution

### Some informations before starting

- Virtual Box 4 doesn't work on windows10 (May 2015), Virtual Box 5 works but non officially
- These tests are done on Windows 7
- Toolbox works on VirtualBox 5
- Linux and windows does not use the same file system format. This result in differences with file ownership. It can make some problems running code.
- Links are written at the end of the post to help for some issues

### installing Toolbox

- Go to [docker toolbox download](https://www.docker.com/toolbox) and download the installer (316Mo). Version was 1.8.1c at the writing time.
- Run the installer
- Run a terminal (cmd/ConEMU/...)
- Start the VM with `docker-machine start default`
- SSH in with `docker-machine ssh default`
- Make a test

 ```bash
C:\Users\username>docker-machine ssh default                                                                        
                        ##         .                                                                               
                  ## ## ##        ==                                                                               
               ## ## ## ## ##    ===                                                                               
           /"""""""""""""""""\___/ ===                                                                             
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~                                                                      
           \______ o           __/                                                                                 
             \    \         __/                                                                                    
              \____\_______/                                                                                       
 _                 _   ____     _            _                                                                     
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __                                                         
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|                                                        
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |                                                           
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|                                                           
Boot2Docker version 1.8.1, build master : 7f12e95 - Thu Aug 13 03:24:56 UTC 2015                                   
Docker version 1.8.1, build d12ea79                                                                                
docker@default:~$ docker ps                                                                                        
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS          
  NAMES                                                                                                            
docker@default:~$ docker run hello-world                                                                           
Unable to find image 'hello-world:latest' locally                                                                  
latest: Pulling from library/hello-world                                                                           
                                                                                                                   
535020c3e8ad: Pull complete                                                                                        
af340544ed62: Pull complete                                                                                        
Digest: sha256:a68868bfe696c00866942e8f5ca39e3e31b79c1e50feaee4ce5e28df2f051d5c                                    
Status: Downloaded newer image for hello-world:latest                                                              
                                                                                                                   
Hello from Docker.                                                                                                 
This message shows that your installation appears to be working correctly.                                         
                                                                                                                   
To generate this message, Docker took the following steps:                                                         
 1. The Docker client contacted the Docker daemon.                                                                 
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.                                          
 3. The Docker daemon created a new container from that image which runs the                                       
    executable that produces the output you are currently reading.                                                 
 4. The Docker daemon streamed that output to the Docker client, which sent it                                     
    to your terminal.                                                          

                                                                                                                   
To try something more ambitious, you can run an Ubuntu container with:                                             
 $ docker run -it ubuntu bash                                                                                      
                                                                                                                   
Share images, automate workflows, and more with a free Docker Hub account:                                         
 https://hub.docker.com                                                                                            
                                                                                                                   
For more examples and ideas, visit:                                                                                
 https://docs.docker.com/userguide/                                                                                
                                                                                                                   
docker@default:~$                                   
```

### Compatibility

Most of the script written with docker suppose that there is a `linux environment` and are written in `bash`.  
When you run `bash` from windows, if you run docker commands, it will fail with a `"cannot enable tty mode on non tty input"` error.  

Here is a simple workaround :

- [`install babun`](http://babun.github.io/)
- start `babun`
- run [the docker fix from tianglo](https://github.com/tiangolo/babun-docker): `curl -s https://raw.githubusercontent.com/tiangolo/babun-docker/master/setup.sh | source /dev/stdin`
- run `docker run -ti busybox sh` to control everything is fine
