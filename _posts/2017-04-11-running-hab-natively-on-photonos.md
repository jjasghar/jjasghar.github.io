---
layout: post
title: "Running hab natively on PhotonOS"
date: 2017-04-11 12:29:30
categories: photon habitat
---

I've successfully run [habitat][habitat] *inside* of [PhotonOS][photon] as a docker container, here's my notes on how I got `hab` to start naively **ON** PhotonOS.

First SSH into the photon box and pull down `hab`:

```shell
root@photon-iso [ ~ ]# wget -O hab.tar.gz https://api.bintray.com/content/habitat/stable/linux/x86_64/hab-%24latest-x86_64-linux.tar.gz?bt_package=hab-x86_64-linux
```

Untar the tarball and move it into `/usr/bin/local/`:

```shell
root@photon-iso [ ~ ]# tar xzf hab.tar.gz
root@photon-iso [ ~ ]# mv hab-<VERSIONNUMBER>-x86_64-linux/hab /usr/bin/local/
root@photon-iso [ ~ ]# chmod a+x /usr/bin/local/hab
```

Confirm it works:

```shell
root@photon-iso [ ~ ]# hab
hab <VERSIONNUMBER>

Authors: The Habitat Maintainers <humans@habitat.sh>

"A Habitat is the natural environment for your services" - Alan Turing
[-- snip --]
```

Create a hab user and group and change the ownership `/hab` directory:

```shell
root@photon-iso [ ~ ]# useradd hab
root@photon-iso [ ~ ]# groupadd hab
root@photon-iso [ ~ ]# chown -R hab.hab /hab
```

Start a process. Here's `redis` as an example.

```shell
root@photon-iso [ ~ ]# # it seems that iptables needs to be opened :(
root@photon-iso [ ~ ]# iptables -A INPUT -p tcp --dport 6379 -j ACCEPT
root@photon-iso [ ~ ]# hab start core/redis

[-- snip --]

★ Install of core/hab-sup/<VERSIONNUMBER> complete with 13 new packages installed.
hab-sup(MR): Butterfly Member ID 6912f94b808e47a09a09e218f0e3614d
hab-sup(SR): Package core/redis not found locally, installing from https://willem.habitat.sh/v1/depot
» Installing core/redis
↓ Downloading core/redis/3.2.4/20170215222111
    569.30 KB / 569.30 KB \ [==========================================================================] 100.00 % 2.52 MB/s
→ Using core/glibc/2.22/20160612063629
→ Using core/linux-headers/4.3/20160612063537
✓ Installed core/redis/3.2.4/20170215222111
★ Install of core/redis/3.2.4/20170215222111 complete with 1 new packages installed.
hab-sup(MR): Butterfly Member ID 6912f94b808e47a09a09e218f0e3614d
hab-sup(SR): Adding core/redis/3.2.4/20170215222111
hab-sup(MR): Starting butterfly on 0.0.0.0:9638
Error persisting rumors to disk, Error reading or writing to DatFile, /hab/sup/default/data/6912f94b808e47a09a09e218f0e3614d.rst, Invalid cross-device link (os error 18)
hab-sup(MR): Starting http-gateway on 0.0.0.0:9631
redis.default(SR): Initializing
redis.default(SV): Starting process as user=hab, group=hab
redis.default(O): 31771:M 11 Apr 17:42:57.943 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
redis.default(O): 31771:M 11 Apr 17:42:57.943 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
redis.default(O): 31771:M 11 Apr 17:42:57.943 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
redis.default(O):                 _._
redis.default(O):            _.-``__ ''-._
redis.default(O):       _.-``    `.  `_.  ''-._           Redis 3.2.4 (00000000/0) 64 bit

[-- snip --]
```

[habitat]: https://www.habitat.sh/
[photon]: https://vmware.github.io/photon/
