---
layout: post
title: "chef-jenkins-vagrant"
date: 2013-06-25 11:43
comments: true
categories: chef vagrant
---

To quote the top of my [README.md](https://github.com/jjasghar/chef-jenkins-vagrant/blob/master/README.md):

> This is my "recipe" on how I got Chef Jenkins and Vagrant to play nice. After watching this (http://www.youtube.com/watch?v=h_IEfNklzW4) video, I realized I could do this. I searched high and low for a tutorial on how to bulid this stack, and nothing. So I decided to create a github doc to centralize my setup and hopefully let the community suggest additions.

I want to thank [Brian Scott](https://github.com/bscott), because after his video it really opened my eyes.  I've now had 3 people run through this tutorial and all come out with a working [chef-jenkins-vagrant](https://github.com/jjasghar/chef-jenkins-vagrant) set ups and learned to extend it to where I had never thought of.  

I understand and that a lot of people wonder why someone would want to build this from scratch, instead of using, for instance [Fletcher Nichol's](https://github.com/fnichol) *ahem*[jamie-ci](https://github.com/jamie-ci/jamie)*ahem* test-kitchen, and this is what I'm going to write about.

I see [test-kitchen](https://github.com/opscode/test-kitchen) is more for writing cookbooks and recipes for the general public.  You can easily create faux VMs, per [Seth Vergo's](https://github.com/sethvargo) awesome [fauxhai](https://github.com/customink/fauxhai) to test cookbooks across multiple setups and multiple provisioned systems to see if they work.  This is great for developing a cookbooks but I can't see it working in a sysadmins workflow.  

Sysadmins have a select few boxes; they have their known quantities, they know they aren't going to have RHEL/DEB/tar based machines.  They build infrastructures to have uniformity, they stick with one of them.  Granted, some of the sysadmnis out there are unlucky and DO have these environments, but I truly believe that's the minority not the majority.

A sysadmin needs a way to provision his machines, in a safe manner before pushing changes out to production. This sounds obvious, but how many places I've been where have I heard "Oh, it's just an /etc/hosts file update, just push it out."  Just yesterday, (06/25/2013) as I was victim if that short cut and caused an outage to some our backend machines.  If you are curious, it seems, we had a hardcoded IP to DNS name in /etc/hosts (not DNS) that was required for our application to work.  If I held true to my statements earlier I would have noticed this change from my CI builds and not looked like the dumbass I was.

I built this [chef-jenkins-vagrant](https://github.com/jjasghar/chef-jenkins-vagrant) howto so I don't have an excuse anymore, I have a proven way to build and implement good practices in an quick manner.  I hope you find it useful, and if you have suggestions or additions PRs are always welcome.

Mind your `rm -rf *`,

JJ Asghar
