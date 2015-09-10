---
published: true
layout: post
title: How to install a good docker environment on windows
categories:
  - Tutorial
tags:
  - docker
  - windows
  - cygwin
  - ConEMU
  - Oh-My-Zsh
---

In this post we will see how to install docker on windows. We will also look at how to work around to the cygwin `cannot enable tty mode on non tty input`.

### Updated

- 13/07 : Added volume informations
- 14/07 : Added issue with cygwin
- 20/07 : Added Virtual Box 5 informations
- 04/09 : Toolbox has replaced boot2docker. The post will be updated soon. The extraction of the env into cmd or powershell still works with toolbox.
- 10/09 : fix to bash under windows issue

### Some informations before starting

- Virtual Box 4 doesn't work on windows10 (May 2015), Virtual Box 5 works but non officially
- These tests are done on Windows 7
- Toolbox works on VirtualBox 5
- Linux and windows does not use the same file system format. This result in difference with file ownership. It can make some problems running code.
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

- [install cygwin](#install-cygwin)
- start a cmd
- update env using docker-machine
- start bash/zsh

That script will do the work (you still need cygwin installed), I call it `docker-bash.bat` for that document :  

```bash
@ECHO off
docker-machine start default >> garbagefile
docker-machine env default --shell cmd > somefile.bat
call somefile.bat 2> garbagefile
rm garbagefile somefile.bat
bash
@ECHO on
```

### Install Cygwin

#### Installation

- First [download](https://www.cygwin.com/), I have used x86 for this tutorial.
- Then install. When it ask for internet / server, you just accept.
- shoes this **Packages** (and other if you want) : `tree (utils), openssh (net), rsync (net), zsh (shells), curl (net), wget (Web), git (Devel), ncurses (Utils), vi, nano`. It will make your life easier

go directly to [install ConEMU](#install-conemu) or [run a bash docker from cmd](#run-a-bash-docker-from-cmd) if you don't want to configure a better cygwin environment.

#### Oh-My-Zsh && apt-cyg (optional)

Oh-My-Zsh is a booster for CLI. It gives lots of usefull shortcut and ui. It's an recommended optionnal ^^.  
To be installed with cygwin installer (just rerun installer to update if you've not chosen this ones) :

- zsh
- wget
- git

[install](https://github.com/haithembelhaj/oh-my-cygwin) with :  

```bash
wget --no-check-certificate https://raw.github.com/haithembelhaj/oh-my-cygwin/master/oh-my-cygwin.sh -O - | sh
```

We have to change Cygwin default shell. I just had zsh at the end of my `.  bashrc` that is in `/c/cygwin/home/username/`. It will run bash and then a zsh inside. There should be a better way for Cygwin to start directly zsh, but I didn't find for now.  

Finally reload .bashrc with `exec -l $SHELL`.  

Some Oh-My-Zsh shortcut :

```bash
gst              # git status
gaa              # git add --all
gcmsg "blabla"   # git commit -m "blabla"
gl               # git pull
gp               # git push
```

There is lots of plugin for most of the tools. Look [here](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins). For [git](https://github.com/robbyrussell/oh-my-zsh/blob/master/plugins/git/git.plugin.zsh)


#### rc files (optional)

In linux, rcfiles are a config files for terminals. When the terminal start, it execute what is in his corresponding rcfile. Bash -> .bashrc, Zsh -> .zshrc.

This files are located in the user folder : `/home/username/`. You can access it in Cygwin with `cd`. (remember that files starting with `.` are hidden (use ls -la to see them))

We can use it to add alias (shortcut for commands), functions, configuration, environment variable, etc ...

#### Some alias examples (optional)

alias are shortcut for CLI. Here are some we can use. Just copy past them in your rcfile.  
Install with cygwin installer if not already done (or update using the same installer file):  

- ncurses
- tree
- clear

```bash
# update shell
alias uprc="exec -l $SHELL"

# showing files
alias l="ls -lthg --color"
alias la="l -A"
alias ct="clear && pwd && tree"

# Edit .zshrc
alias edz="nano ~/.zshrc && uprc"
alias edb="nano ~/.bashrc && uprc"

# docker
alias dps="docker ps"
alias dpsa="docker ps -a"
alias dk="docker kill"
alias dgarbage="docker kill \`docker ps -q\` && docker rm \`docker ps -a -q\` && docker rmi \`docker images -q -f dangling=true\`"

# path (here we add docker to our bash PATH)
export PATH="/cygdrive/c/Program Files/Docker Toolbox:$PATH"
```

### install ConEMU

Here I just point to ConEMU installer. The `cmd` and `powershell` UI are unusable if you are not in Windows 10. And good way to go is `ConEMU`.

The installer is [here](http://conemu.github.io/).

Just to have a minimal experience :

- Win+W : start new terminal (with the configuration)
- Win+Q/Win+Shift+Q : switch from terminal
- Win+X : start a cmd.exe without configuration

Ask ConEMU to run our docker start script when you start it :

- `start ConEMU`
- `Win+Alt+P` or click on the 3 bars in the upper-right corner then click on settings
- Click on the `Startup` on the left
- Click `Command line` and enter that cmd : `cmd.exe /k "docker-bash.bat"` where docker-bash.bat is the docker script written before. And don't forget that it need to be callable from the PATH !


### run a bash docker from cmd

- Start a cmd (with ConEMU if installed)
- `enable docker` in your cmd.exe :  

```bash
C:\Users\username>docker ps                                                                       
Get http://127.0.0.1:2375/v1.19/containers/json: dial tcp 127.0.0.1:2375: ConnectEx tcp: No connection could be made b
ecause the target machine actively refused it.. Are you trying to connect to a TLS-enabled daemon without TLS?        
                                                                                                                      
C:\Users\username>docker-machine env default --shell cmd                                                               
set DOCKER_TLS_VERIFY=1                                                                                               
set DOCKER_HOST=tcp://192.168.99.100:2376                                                                             
set DOCKER_CERT_PATH=C:\Users\username\.docker\machine\machines\default                                                
set DOCKER_MACHINE_NAME=default                                                                                       
# Run this command to configure your shell:                                                                           
# copy and paste the above values into your command prompt                                                            
                                                                                                                      
C:\Users\username>set DOCKER_TLS_VERIFY=1
C:\Users\username>set DOCKER_HOST=tcp://192.168.99.100:2376                                        
C:\Users\username>set DOCKER_CERT_PATH=C:\Users\username\.docker\machine\machines\default
C:\Users\username>set DOCKER_MACHINE_NAME=default                                        
C:\Users\username>docker ps                                                                                            
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS             
  NAMES                                                            
```

- start yout bash/zsh :

```bash
C:\Users\username>bash
➜  username  docker run -ti busybox sh
/ # echo run from zsh !!
run from zsh !!
/ # %
➜  username  docker run busybox sh -c "echo run from zsh"
run from zsh
➜  username
```
