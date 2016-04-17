---
layout: post
title: "Lets build a Minecraft Server with Chef"
date: 2015-08-23 17:15:07 -0500
comments: true
categories: linux gaming chef minecraft
---

These are some steps to creating a [Minecraft](https://minecraft.net/) server on [Digital Ocean](https://www.digitalocean.com/). The initial setup is only done the first time, then you'll be able to do just the `create` command when you want a new Minecraft Server.

You can do the following steps with something like [Rackspace](http://www.rackspace.com/) or [AWS](http://aws.amazon.com/) you just need to switch out the `knife-digital_ocean` commands with either `knife-rackspace` or `knife-ec2`. There are different configurations you need to do with either `knife` plugins, and slightly different commands to `create` servers, so please read the documentation.

1. Download the [ChefDK](https://downloads.chef.io/chef-dk/), and install it.
1. Install the Digital Ocean knife plugin `chef exec gem install knife-digital_ocean`
1. Sign up for [hosted Chef](https://manage.chef.io/signup), and log in to your org you create.
1. Pull down the "starter kit," which is located at `https://manage.chef.io/organizations/<THEORGYOUCREATED>/getting_started`.
1. [Configure](https://github.com/rmoriz/knife-digital_ocean#configuration) the `knife.rb` for Digital Ocean you'll find it in the "starter kit" in the `.chef/` directory.
1. Git clone the [minecraft-basic](https://github.com/jjasghar/minecraft-basic) into the `chef-repo/cookbooks/` directory.
1. In your main `chef-repo/` directory run the following steps:
  1. Run `chef exec knife status` to verify you can talk to your hosted Chef instance.
  1. Run `chef exec knife digital_ocean sshkey list` to verify you can talk to Digital Ocean, and figure out your `SSHKEYNUMBER` for a following step.
  1. Run `chef exec knife cookbook upload minecraft-basic` to upload `minecraft-basic` cookbook to your hosted Chef instance.
  1. Run `chef exec knife cookbook list` to verify you successfully uploaded the cookbook. You should see the cookbook name and version number output.
1. You should be ready to run something like the following: `chef exec knife digital_ocean droplet create --server-name minecraft --image ubuntu-14-04-x64 --location sfo1 --size 4gb --ssh-keys <YOURSSHKEYNUMBER> --bootstrap --run-list "recipe[minecraft-basic]"` You may want to tweak this for your usage, I picked 4 gig because it seems `java` seems to play nice with this size. A 4 gig box will run you 40 bucks a month, or $0.06 cents an hour, while a 2 gig instance will cost you $0.03 per hour or 20 bucks a month. Don't forget to blow the machine away, if you aren't planing on running the Digital Ocean server 24x7. _I take no responsibility for your forgetfulness_ ;).  If you would like to use [CentOS 7](https://www.centos.org/) you can change the `--image ubuntu-14-04-x64` to `--image centos-7-0-x64`. I haven't tested this build with anything other then CentOS or Ubuntu, but I'd be interested if you got it working on other distributions.
1. I haven't automated the saving or exporting the worlds that is created with this cookbook _yet_. But here is a link to a place that tells you how to do it by [hand](http://www.howtogeek.com/202591/how-to-backup-your-minecraft-worlds-mods-and-more/), in case you need a refresher.
1. When you are done, you can blow it all away with these commands:

```
~$ SERVER=`chef exec knife digital_ocean droplet list | grep minecraft | awk -F ' ' {'print $1'}`
~$ chef exec knife digital_ocean droplet destroy -S $SERVER # This destroys the machine on Digital Ocean
~$ chef exec knife node delete minecraft -y && chef exec knife client delete minecraft -y # This deletes it from Hosted Chef
```

If you've completed these steps, you won't need to login to the Host Chef instance unless you want to check the cookbook. You should be able to just spin up your Minecraft server with just `chef exec knife digital_ocean droplet create --server-name minecraft --image ubuntu-14-04-x64 --location sfo1 --size 4gb --ssh-keys <YOURSSHKEYNUMBER> --bootstrap --run-list "recipe[minecraft-basic]"`. And use the above commands to blow everything away.

I'll post another article when I figure out a good/chefie/automated way to backup, export, and share your worlds you create.
