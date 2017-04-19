---
layout: post
title: "Docker Squid Cache and MacOS"
date: 2017-04-19 15:44:06
categories: macos squid docker
---

I pull a lot of things down from the internet. I seem to pull the same thing
weather it be a `gem` or `iso` over and over. I finally decided to create a
local Squid proxy, to cache my downloads so I don't have to keep going out
to the internet to pull things down.

First thing first. Install Docker. There are a ton of how-tos, I'd google
for em.

Awesome, lets start the next part. Pull down the defacto squid container:

```shell
$ docker pull sameersbn/squid:3.3.8-23
```

You can use `:latest` if you want, but this was the version I picked for no
specific reason.

Next, you'll need to find a place for your cache. By default Docker file-shares
the `/Users` directory on MacOS so this is what I did:

```shell
$ mkdir -p ~/squid/cache/
```

Start up the container!

```shell
$  docker run --name squid -d --restart=always \
>   --publish 3128:3128 \
>   --volume /Users/jjasghar/squid/cache:/var/spool/squid3 \
>   sameersbn/squid:3.3.8-23
```

Check that the container is running:

```shell
$ docker ps
CONTAINER ID        IMAGE                      COMMAND                 CREATED             STATUS              PORTS                    NAMES
6d245aedb504        sameersbn/squid:3.3.8-23   "/sbin/entrypoint.sh"   3 minutes ago       Up 3 minutes        0.0.0.0:3128->3128/tcp   squid
```

Next, figure out the IP, for me it was `utun4`, then export out some settings to bash:

```shell
$ ifconfig
[-- snip --]
utun4: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1500
	inet 172.25.0.14 --> 172.25.0.14 netmask 0xffff0000
	inet6 fe80::f65c:89ff:feca:6527%utun4 prefixlen 64 scopeid 0x12
	inet6 2001:470:bb7b:ffff::14 prefixlen 64
	nd6 options=201<PERFORMNUD,DAD>
$ export ftp_proxy=http://172.25.0.14:3128
$ export http_proxy=http://172.25.0.14:3128
$ export https_proxy=http://172.25.0.14:3128
```

With this, you can now run, in a different terminal:

```shell
$ docker exec -it squid tail -f /var/log/squid3/access.log
```

And `curl` google:

```shell
$ curl https://google.com
```

You should see something like the following in the `access.log` window:

```shell
1492635161.351    529 172.17.0.1 TCP_MISS/200 15223 CONNECT www.google.com:443 - HIER_DIRECT/216.58.194.132 -
```

Congrats, you have a working proxy now. Lets do some configuration, stop your container:

```shell
$ docker stop squid
```

Now go to your `~/squid` directory, and pull down the main `squid.conf` to it, and open it up
in your text editor:

```shell
$ cd squid
~/squid $ wget https://raw.githubusercontent.com/sameersbn/docker-squid/master/squid.conf
~/squid $ emacs squid.conf
```

Make any changes you think you might need, I'm still learning them myself. After your done,
start it back up!

```shell
$  docker run --name squid -d --restart=always \
>   --publish 3128:3128 \
>   --volume /Users/jjasghar/squid/cache:/var/spool/squid3 \
>   --volume /Users/jjasghar/squid.conf:/etc/squid3/squid.conf \
>   sameersbn/squid:3.3.8-23
```
