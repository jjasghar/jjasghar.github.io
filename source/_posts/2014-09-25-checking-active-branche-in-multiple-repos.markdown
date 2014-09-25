---
layout: post
title: "Checking active branch in multiple repos"
date: 2014-09-25 14:35:07 -0500
comments: true
categories: git openstack sysadmin
---

I have a bunch of [repos](https://github.com/stackforge/?query=cookbook-openstack) that I have to watch over and make patches against.
Trying to figure out what branch I left checked out can be...well frustrating.
I put all my [openstack](http://www.openstack.org) repos in a sub directory from my standard `~/repo/` directory. I wanted to figure out a way to check all the active branches that I had  checked out; and it turns out that it was hellva a lot more challanging that expected.

As a sysadmin, you probably say ok, `git branch | awk '/\*/ { print $2; }'` seems ok, but man that's a bitch to write.

Leveraging a little bit of googling `git rev-parse --abbrev-ref HEAD` appeared which does the exact same. Odd that there
isn't easier command.

So putting it all together I come up with something like the following to tell me what the active branch is for each of
my repos. This is a quick one liner that does the trick for me.

```shell
for i in *; do cd $i ; pwd; git rev-parse --abbrev-ref HEAD ; cd ../ ; done
```
