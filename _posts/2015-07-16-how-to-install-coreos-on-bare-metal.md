---
published: true
layout: post
title: How to install coreos on baremetal
categories:
  - Tutorial
tags:
  - CoreOS
  - baremetal
---

We will look how to install CoreOS on a machine. I try to go in the simplest way. I won't configure ssh, just install CoreOS with a root password. From that, you should easily evolve.  
I wont explain CoreOS mecanism in this post.

## Usefull Links :

- [CoreOS official installing to disk page](https://coreos.com/os/docs/latest/installing-to-disk.html)
- [CoreOS cloud-config page](https://coreos.com/os/docs/latest/cloud-config.html)

## Specific installation

CoreOS is a bit special to install because you need to run installer from a different linux distribution on the machine you want to install.  
CoreOS also comes with cloud-config files to define the way it will be running. You will have to prepare what you want him to be before triggering the installation.

## cloud-config

Here is a minimalist config-file to run CoreOS :

```yaml
#cloud-config

users:
  - name: core
    passwd: hashed_password
coreos:
  etcd:
    name: node01
    discovery: https://discovery.etcd.io/token

hostname: node01
```

- The `#cloud-config` at the start of the file is needed for coreos to understand the file.
- We configure the core user by adding an encrypted password (see [next](#how-to-encrypt-password) how to create encrypted password)
- We add the discovery Url

## installation

We need to run a linux on the machine, personally I used a Ubuntu on usb key that I prepared with [UNetBootIn](http://unetbootin.sourceforge.net/).

Boot your bare-metal with linux, start a terminal, go to this address, and copy/paste this to your terminal :

```bash
cat <<'EOF1' > install.sh

if [[ $# != 1 ]]; then
   echo "args usage : "
   echo "  1- password "
   echo " "
   exit 1
fi

# hash password
password=`echo $1 | mkpasswd --method=SHA-512 --rounds=4096 -s`

# add needed package
sudo apt-get install -y curl wget whois

# get discovery url
discoveryUrl=`curl https://discovery.etcd.io/new`

# write cloud-config.yml
cat <<EOF2 > cloud-config.yml
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
EOF2

# get the coreos installation script
wget https://raw.github.com/coreos/init/master/bin/coreos-install

# run installation
chmod 755 coreos-install
sudo ./coreos-install \
      -d /dev/sda \
      -c cloud-config.yml \
      -C alpha
EOF1

chmod 755 install.sh
```

It will create a script, that will :

- control you give a password as an argument
- hash the given password
- get a new discovery url
- write a cloud-config file (with pass and discoveryUrl)
- get the coreos-install script and make it executable

Note that I install the `alpha` channel on `/dev/sda`. You may need to adapt the script to your needs.  
So run the script with a password of your choice :

```bash
./install.sh your_private_password
```

And something like that should appear :

```bash
Downloading the signature for http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2...
2015-07-16 11:44:09 URL:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2.sig [543/543] -> "/tmp/coreos-install.dLdl1JFbNl/coreos_production_image.bin.bz2.sig" [1]
Downloading, writing and verifying coreos_production_image.bin.bz2...
2015-07-16 11:46:52 URL:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2 [178488210/178488210] -> "-" [1]
gpg: Signature made Thu 09 Jul 2015 07:20:35 PM UTC using RSA key ID E5676EFC
gpg: key 93D2DCB4 marked as ultimately trusted
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: Good signature from "CoreOS Buildbot (Offical Builds) <buildbot@coreos.com>"
Installing cloud-config...
Success! CoreOS alpha current is installed on /dev/sda
```

If the last line is `Success! ...`, you can run a `sudo reboot` and connect to your coreos with :

- login : core
- password : `what you have given`

## Update coreos

### cloud-config

You can then update your cloud-config :

- write a new one in a yourfile.yml
- run `coreos-cloudinit --from-file yourfile.yml`

### bashrc

You can update bashrc with this command : `cp --remove-destination bashrc /home/core/.bashrc`

### formatting

You can now rerun the same installation process from coreos.
