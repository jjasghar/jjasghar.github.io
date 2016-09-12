---
layout: post
title: "Why aren't my machines showing up in vCenter"
date: 2016-09-09 16:09:14
categories: vmware
---

I've started using [knife-vsphere][knife] and vCenter by [VMware][vmware]. I was
attempting to build machines, and noticed that the `knife vsphere` command showed
my machines, but vCenter didn't.

For example:

```sh
16:04:56 JJs-MacBook-Pro ~ > ce knife vsphere vm list
Folder:
       	VM Name: ubuntu-3
       		IP: 172.16.20.105
       		RAM: 4096
       		State: on
       	VM Name: pfsense
       		IP:
       		RAM: 1024
       		State: on
       	VM Name: vCenter
       		IP:
       		RAM: 8192
       		State: on
```

I looked and looked for `ubuntu-3` and to no avail, clicking around assuming that
the website as I clicked was refreshing in the background. I was wrong.

Then I noticed a "refresh" circle in the top right corner.

![vCenter Refresh][refresh]

Gotta love how you assume one thing and trust software to do the right thing
and you're off base.

[knife]: https://github.com/chef-partners/knife-vsphere
[vmware]: http://www.vmware.com/
[refresh]: ../../../../../pics/vCenter_refresh.png
