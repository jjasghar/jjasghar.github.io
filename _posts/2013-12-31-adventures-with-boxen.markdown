---
layout: post
title: "Adventures with boxen"
date: 2013-12-31 15:54
comments: true
categories: linux puppet sysadmin
---

Note: Thanks to [Greg Kitson](https://github.com/awaxa/) as my sounding board and inspiration here to get this done.

So I started watching a couple videos on [boxen](http://boxen.github.com) both are done by [@wfarr](https://twitter.com/wfarr) and he's convinced me to give it a shot. Here is the "first" part of me getting up a dev env to play around with it.

I think the thing that got me was the "one line cURL" that you can run which is from [boxen-web](https://github.com/boxen/boxen-web) so that's my ultimate goal here; ok lets start this off.

First thing, read the [README.md](https://github.com/boxen/our-boxen/blob/master/README.md) and something jumps out at me straight off the bat:

```
Boxen requires at least the Xcode Command Line Tools installed.
Boxen will not work with an existing rvm install.
Boxen may not play nice with a GitHub username that includes dash(-)
Boxen may not play nice with an existing rbenv install.
Boxen may not play nice with an existing chruby install.
Boxen may not play nice with an existing homebrew install.
Boxen may not play nice with an existing nvm install.
Boxen recommends installing the full Xcode.
```

*sigh* So this means I need a vm to start playing with this. I chose virtualbox as my hypervisor. The trick is; you can only virtualize OS X on OS X so get it installed on your mac ;).

It seems there is a Mavericks network issue with [virtualbox 4.2](https://forums.virtualbox.org/viewtopic.php?f=8&t=58036) so I suggest updating to the latest before going any farther. I ran into a problem with the network adapter and the install wizard wouldn't get farther than Manual/DHCP network setup. *grrr*

Ok, so I'm writing this real time...yeah this blew up. So I converted to [VMware Fusion 6](https://my.vmware.com/web/vmware/info/slug/desktop_end_user_computing/vmware_fusion/6_0) to attempt to build the box. Lets see how that goes.

So tricky VMware Fusion, it says it's designed for Windows, but you can use the Install.App you got from the Mac App store. When it asks for an image to install from, just point it there!

Spin up a machine like it suggests and get a base image installed. Go ahead shut it down here, take a snapshot it as "base install or something" so you can roll back here because you're using a provisioning software and you need a clean box right? RIGHT? (yes yes vagrant would be perfect here, but that's another cost got get it working with fusion)

After you got that done boot up the vm open terminal run the following:

`git --version`

This should prompt you to install the "mac cli tools" without needing to install Xcode. Badass.

Go ahead and fork [our-boxen](https://github.com/boxen/our-boxen) like it suggests. Now if you want this to be private, for your company..(I'm not doing because I'm trying to learn this), you have to go through the following:

```bash
~% sudo mkdir -p /opt/boxen
~% sudo chown ${USER}:staff /opt/boxen
~% git clone https://github.com/boxen/our-boxen /opt/boxen/repo
~% cd /opt/boxen/repo
/opt/boxen/repo% git remote rm origin
/opt/boxen/repo% git remote add origin <the location of my new git repository>
/opt/boxen/repo% git push -u origin master
```

This deploys the default boxen setup like the README.md says, customizing is in a following post.

Ok, now try out boxen:

```bash
/opt/boxen/repo% ./script/boxen
```

Alas this will probably bitch about no Full Disk Encryption, so you'll need to run (good for developing this):

```bash
/opt/boxen/repo% ./script/boxen --no-fde
```

It should ask you for your github credentials (yes it works with two factor authentication), then you should start seeing a bunch of stuff on the screen!

NOTE: you might see it get stuck at `Dnsmasq` for a default run. If you check activity monitor it is running and talking to Greg, it seems that's gcc/make compiling something in the background. This might take a long time, wait it out.

Grats you have successfully started up boxen and got it running. Now lets customize it, that should be the following post...
