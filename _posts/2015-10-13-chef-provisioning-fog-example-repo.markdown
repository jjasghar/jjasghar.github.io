---
layout: post
title: "chef-provisioning-fog-example repo"
date: 2015-10-13 09:55:21 -0700
comments: true
categories: chef fog sysadmin
---

After my [post](http://jjasghar.github.io/blog/2015/09/25/chef-provisioning-fog-step-by-step-day-by-day/) about how to build/use
[chef-provisioning-fog](https://github.com/chef/chef-provisioning-fog) I realized that sometimes people don't want how to build
the plane, they just want to fly.

So I spent some time building a "skeleton" repo for for chef-provisioning-fog and called it
[chef-provisioning-fog-example](https://github.com/jjasghar/chef-provisioning-fog-example). Turns out I wasn't the first person to
think of this. [Martin Smith](https://github.com/martinb3/) had the exact same idea with
[chef-provisioning-rackspace](https://github.com/martinb3/chef-provisioning-rackspace-example). I tip my hat to him for coming up
with this idea before me and allowing me to steal some of his ideas.

My main goal with this repo is, if you have an OpenStack cloud, and you need to provision a couple machines and have only a couple
cookbooks to help bootstrap them this should get you off ground. There are much more advanced things you can do, but this is enough
to show you the power that chef-provisioning-fog has.

An example use case is disposable 'dev' environments. There are certain work flows that need to build infrastructures from scratch,
and leveraging chef-provisioning-fog can help provision the cluster in a repeatable effective manner. There is no reason why you couldn't
use a real chef server, or hosted chef with this also. I picked [chef-zero](https://github.com/chef/chef-zero) for this example
so you don't have to worry about the overhead of chef server to start off.

Taken from the [README](https://github.com/jjasghar/chef-provisioning-fog-example/blob/master/README.md) the usage is pretty straight
forward.

1. Copy/download whatever you want in the `cookbooks/` directory. (there is an example cookbook already there)
1. Edit the [.chef/knife.rb](https://github.com/jjasghar/chef-provisioning-fog-example/blob/master/.chef/knife.rb) for your settings. Including the recipes that you want to run.
1. Run `chef-client -z demo.rb` or if you've renamed the file that file name.
1. When you're done, you can use the `chef-client -z destroy.rb` to blow everything away.
1. Edit/save this for what you want, this should be enough to bootstrap you now :D.
