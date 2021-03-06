---
layout: post
title: "PhotonOS setup for remote access"
date: 2017-04-04 10:56:13
categories: chef photon vmware kitchen
---


I've had to set up a couple instances of [PhotonOS][photon] in my testing remote [docker][docker]
with test-kitchen. Here are my notes on how to get the PhotonOS prep'd and allowing for remote
connections.

First you need to enable `PermitRootLogin yes` at the bottom of the file:

```shell
root@photon-iso [ ~ ]# vi /etc/ssh/sshd_config
```

After that you need to allow for `iptables` to open the port for the docker process:

```shell
root@photon-iso [ ~ ]# iptables -A INPUT -p tcp --dport 2375 -j ACCEPT
```

You'll need to add `-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock` to the end of `ExecStart`
```shell
root@photon-iso [ /usr/lib/systemd/system ]# vi docker.service.d/10-dockerd.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
root@photon-iso [ /usr/lib/systemd/system ]#
```

Then start the process!

```shell
root@photon-iso [ ~ ]# systemctl start docker
```

You can verify your connection with a remote machine via:

```shell
$ DOCKER_HOST=tcp://IPOFMACHINE:2375 docker info
```

[photon]: https://vmware.github.io/photon/
[docker]: https://www.docker.com/
