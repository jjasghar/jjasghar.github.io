---
layout: post
title: "Fedora CoreOS working on VirtualBox"
date: 2020-05-26 16:05:45
categories: sysadmin, linux, tutorial
---

> Note, a lot of this was inspired from [this](https://gist.github.com/noonat/9fc170ea0c6ddea69c58) GitHub gist.

Hi! I wanted to build up a VM for Fedora CoreOS, and these were my steps to get
it to work. It's just a basic installation, and enough to play with it. I have a
romatic goal of moving all my servers and applications to an immutable OSs, and
getting my hands on a disposable Fedora CoreOS is the first learning step.

## Prereqs

- Download and install VirtualBox.
- Download the CoreOS ISO from [here](https://getfedora.org/en/coreos/download)

## Create a new VM in VirtualBox

- Select Fedora 64-bit
- Give the VM at least 4 Gigs, I gave mine 8 Gigs
- 16 Gigs for the hard drive to test
- Set the network to "NAT" and add the following to the port forwarding:
  - Host IP: 127.0.0.1
  - Host Port: 2222
  - Guest IP: 10.0.2.15
  - Guest Port: 22
- Mount the LiveCD as an Optical Device
- Confirm that you'll boot off it as your first device

## Boot the machine

- Go ahead and boot the machine, and you should boot into something like:

```bash
[core@localhost ~]$
```

## Configure your ignition file

There are a ton of options for your file, see [here](https://docs.fedoraproject.org/en-US/fedora-coreos/fcct-config/) but for this basic installation take the following `yaml` file.

```yaml
variant: fcos
version: 1.0.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAA...
```

Put it on your local file system, and add your `id_rsa.pub` as the `- ssh-rsa AAA`
line. Save the file as something like `example.yaml`. Next, you want to convert it to the `json` for the `.ign`
file. Run the following on your local machine, (this is assuming you have Docker installed):

```bash
docker run -i --rm quay.io/coreos/fcct:release --pretty --strict < example.yaml > example.ign
```

This will create the `json` friendly version from your `yaml`.

## Get the `ign` file onto your VM

Next you need to get your file onto your VM. The easiest way to do this is
in your directory with the `example.ign` run the following to start a simple `python3` webserver.:

```bash
python3 -m http.server
```

Now you can go to your booted Fedora CoreOS machine and run a `curl` to bring it 
to the box: (10.0.2.2 is the default host machine, check yours by `route -n` whatever
the Gateway is)

```bash
curl -LO 10.0.2.2:8000/example.ign
```
*Note*: by default the python server runs on port `8000`.
You should now have your `example.ign` on your Fedora CoreOS machine.

## Install the base image

Now that you have your `example.ign` on the machine you can run the installer, there
are multiple options but the basic command is the following:

```bash
sudo coreos-installer install /dev/sda --ignition-file example.ign
```
You should see a default image being pulled down, and it should inject your
`example.ign` file. It will be done when you see:

```console
Install complete.
```

Now that it's done, you need to `sudo shutdown -h now` and pull the ISO
from the Virtual Drive. Power up the machine again.

This will start going through the first boot, and then you should see: 

```console
localhost login:
```

Now go to another terminal on your local machine and SSH in:

```bash
ssh 127.0.0.1 -p 2222 -l core
Fedora CoreOS 31.20200505.3.0
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/c/server/coreos/

[core@localhost ~]$
```

Congratulations! You now have a Fedora CoreOS machine ready to play with.

## Verifying your `docker` installation

Just as a quick sanity check, you can run the following to make sure you're machine
can pull from the `docker` registry and also run the containers:

```bash
[core@localhost ~]$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Hopefully this'll help you get your hands on Fedora CoreOS to play around with.
