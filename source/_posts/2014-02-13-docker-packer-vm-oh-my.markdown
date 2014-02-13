---
layout: post
title: "docker packer VM oh my"
date: 2014-02-13 12:17:09 -0600
comments: true
categories: sysadmin
---

So I've been caught up in the [docker](http://docker.io), [packer](http://packer.io), and VM discussion a handful of times. I've been defending and discussing*cough*attacking*cough* different
versions and use cases. Then something happened, something I thought docker is wasn't what it was, and something I thought packer was isn't what it is. Welp, there's some egg on my face eh?
So here's a cheatsheet type post about the difference and __POSSIBLE__ use cases.

### docker
As the docker says in their [intro slide deck](http://www.slideshare.net/dotCloud/docker-intro-november), it's designed to "ship code." This is important distinction between this and what a VM does.
docker is a souped–up version of [lxc](http://linuxcontainers.org/) or to a lesser extent [freebsd jails](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/jails.html). It allows you to
run an application in it's own little bubble, but has a _shared_ underlying OS/kernel. (in this case linux) It allows for a quick provisioning of an application disposable and transient applications
which is great for development work. You can spin up a "base" build extremely quickly and hack away and it and rollback to a know good spot, without having to reimage/build the machine. In the new [TDD](http://en.wikipedia.org/wiki/Test-driven_development)
world this is great for iterations. I guess a good simile is like you have a pad of paper with a base story printed on each page, you tack on the writing you want at the bottom, and if you don't like it just tear it
away and start again.

### VM
So docker is designed for an application? Isn't that what a VM is designed for? Not exactly. docker shares the kernel and base os, while a VM is it's own entity. VMs also require a hypervisor of some type
that sits on another os; so if you can imagine it, VM = os ontop of hypervisor (that controls the VMs) on top of another os. This has it's merits, but at the same time it can be cumbersome, for development. It seems
VMs use case, compared to something like docker, is more for production deployments, more static content that updates through a deployment system.  This can bring in security uses and the like, keep that in mind.

### packer
packer in a couple words is this: a way to create base or gold VM images. It's designed to create a typical/transferable VM image that works with multiple platforms and hypervisors. An example, you have a beloved mail server, you create
an image of this box, but you're boss now says you're moving cloud providers and  have to get it working on [digital ocean](http://digitalocean.com/). packer makes that conversion "simple," which packs up the image to be portable and transferable.
I say simple mainly because honestly I haven't tried it yet, but I have watched a few youtube tutorials on it.

This post is spurred from my miss understanding of the scope of docker and VMs, and in the process of groking them, rediscovering packer.

So in a nutshell: docker is a wrapper for lxc/jails for linux running an app sharing a common kernel, VMs are separate entities and kernels that more production focused, and packer is a way to create transferable VMs between hypervisors.


