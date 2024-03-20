---
layout: post
title: "installing nvidia drivers on centos 8 stream"
date: 2024-03-20 10:55:21
categories: linux gpu centos
---

I had some trouble getting the Official Drivers from Nvidia working on a GPU "Tesla T4" headless
machine in a Datacenter running CentOS 8 Stream.

Here are my commands/notes on how I got it working, future me will be happy this is here.

First install "newer" release of the Nvidia drivers. 550 was the newest as of writing this,
but look [here](https://www.nvidia.com/Download/Find.aspx?lang=en-us) for the newest ones.

```bash
wget https://us.download.nvidia.com/tesla/550.54.15/NVIDIA-Linux-x86_64-550.54.15.run
chmod +x NVIDIA-Linux-x86_64-550.54.15.run
```

Then install the [main line](https://elrepo.org/wiki/doku.php?id=kernel-ml) kernel, because
RHEL 8 removed some needed headers for nvidia.

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml
yum --enablerepo=elrepo-kernel install kernel-ml-devel
uname -r
reboot
```

Validate that you are running the newest kernal, then run the binary from Nvidia.

```bash
uname -r
./NVIDIA-Linux-x86_64-550.54.15.run
reboot
```

Reboot and run `nvidia-smi` to make sure that the Nvidia drivers can see the GPU.
Install `nvtop` so you can track usage of your GPU.

```bash
nvidia-smi
dnf install -y epel-release
dnf install nvtop
```
