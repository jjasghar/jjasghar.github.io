---
layout: post
title: "installing nvidia drivers on centos 9 stream"
date: 2024-08-15 12:55:21
categories: linux gpu centos
---

I had some trouble getting the Official Drivers from Nvidia working on a GPU "Tesla V100" headless
machine in a Datacenter running CentOS 9 Stream.

Here are my commands/notes on how I got it working, future me will be happy this is here.

First install "newer" release of the Nvidia drivers. 550 was the newest as of writing this,
but look [here](https://www.nvidia.com/Download/Find.aspx?lang=en-us) for the newest ones.

```bash
wget https://us.download.nvidia.com/tesla/550.54.15/NVIDIA-Linux-x86_64-550.54.15.run
chmod +x NVIDIA-Linux-x86_64-550.54.15.run
```

Then install the `development tools` group, and run the `.run`. You'll have to "yes"
through the wizard.

```bash
dnf install @"development tools"
./NVIDIA-Linux-x86_64-550.54.15.run
reboot
```

Reboot and run `nvidia-smi` to make sure that the Nvidia drivers can see the GPU.
Install `nvtop` so you can track usage of your GPU.

```bash
nvidia-smi
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %{rhel}).noarch.rpm
dnf install -y nvtop
```

