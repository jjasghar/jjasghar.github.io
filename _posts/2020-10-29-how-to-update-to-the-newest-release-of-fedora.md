---
layout: post
title: "How to update to the newest release of Fedora"
date: 2020-10-29 10:41:30
categories: sysadmin linux fedora
--- 

The [Fedora][fedora] community just released Fedora [33][33] the other day. I've
been running Fedora 32 for a while, on both servers and my laptop, and loved every
minute of it. Needless to say as soon as I heard that 33 came out I needed upgrade
to it. There's a handful of pages and howto's out there, but for my own sanity
I'm capturing this here. 

> This is also how to fully upgrade your release, so this will work in the future

```bash
sudo su - # yes you need to be root
dnf upgrade --refresh
dnf install dnf-plugin-system-upgrade
dnf system-upgrade download --releasever=33 # put 'next' version here
dnf system-upgrade reboot # this reboot can take a while
cat /etc/redhat-release # confirm your update!
```

And that's it! Pretty straight forward, and from some statements from friends, you
should go one release at a time, so don't jump from Fedora 30 to 33, go 30 to 31
to 32 to 33.

[fedora]: https://getfedora.org/
[33]: https://fedoramagazine.org/announcing-fedora-33/
