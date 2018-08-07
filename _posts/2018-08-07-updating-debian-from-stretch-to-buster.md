---
layout: post
title: "Updating Debian from stretch to buster aka Debian 9 to 10"
date: 2018-08-07 15:43:32
categories: sysadmin linux
---


I got a PXE server that was able to [netboot](https://wiki.debian.org/PXEBootInstall) Debian 9.5. I wanted to start
playing with Buster/Sid, and completely forgot how to convert the
machine.

For more information on Sid, check out [here](https://wiki.debian.org/DebianUnstable).
But the most relevant thing about it is this:

> The Unstable repositories are updated every 6 hours. You can upgrade with apt-get dist-upgrade, taking all the necessary precautions beforehand of course.

I did some googling and found [this page](https://linuxconfig.org/how-to-upgrade-debian-9-stretch-to-debian-10-buster)
on how to do it, but figured I'd capture it for myself here.

Make sure that your Debian 9.5 machine is as up to date as possible:

_NOTE_: you need to be root for all of this.

```shell
apt-get update
apt-get upgrade
apt-get dist-upgrade
```

Next, convert the `sources.list` from `stretch` to `buster`.

```shell
sed -i 's/stretch/buster/g' /etc/apt/sources.list
```

Update the `apt cache` with the new sources.

```shell
apt-get update
```

Update the machine fully:

```shell
apt-get upgrade
apt-get dist-upgrade
```

Verify that the update has succeeded:

```shell
cat /etc/debian_version
buster/sid
```

I'd reboot now to make sure everything comes up as expected.
