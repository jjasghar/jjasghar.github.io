---
layout: post
title: "Building a factorio multiplayer server on Digital Ocean"
date: 2016-03-20 16:57:57 -0500
comments: true
categories: linux gaming chef factorio
---


These are the steps to create a [factorio](http://factorio.com) server on
[Digital Ocean](http://digialocean.com). The initial setup is only done the first time, then you’ll be
able to do just the create command when you want a new Factorio Server.

You can do the following steps with something like Rackspace or AWS you just need
to switch out the [knife-digital_ocean](https://github.com/rmoriz/knife-digital_ocean) commands with either [knife-rackspace](http://github.com/chef/knife-rackspace) or
[knife-ec2](http://github.com/chef/knife-ec2). There are different configurations you need to do with either knife
plugins, and slightly different commands to create servers, so please read the documentation.

-   Download the ChefDK, and install it.
-   Open up a terminal, or command prompt.
-   Install the Digital Ocean knife plugin `chef exec gem install knife-digital_ocean`
-   Sign up for [hosted Chef](https://manage.chef.io/login), and log in to your org you create.
-   Pull down the “starter kit,” which is located at `https://manage.chef.io/organizations/<THEORGYOUCREATED>/getting_started`.
-   [Configure](https://github.com/rmoriz/knife-digital_ocean#configuration) the `knife.rb` for Digital Ocean you’ll find it in the “starter kit” in the .chef/ directory.
-   Git clone the `factorio-cookbook` into the `chef-repo/cookbooks/` directory.
-   In your main `chef-repo/` directory run the following steps:
-   Run `chef exec knife status` to verify you can talk to your hosted Chef instance.
-   Run `chef exec knife digital_ocean sshkey list` to verify you can talk to Digital Ocean, and figure out your SSHKEYNUMBER for a following step.
-   Run `chef exec knife cookbook upload factorio-cookbook` to upload factorio-cookbook to your hosted Chef instance.
-   Run `chef exec knife cookbook list` to verify you successfully uploaded the cookbook. You should see the cookbook name and version number output.
-   You should be ready to run something like the following: `chef exec knife digital_ocean droplet create --server-name factorio --image ubuntu-14-04-x64 --location sfo1 --size 4gb --ssh-keys <YOURSSHKEYNUMBER> --bootstrap --run-list "recipe[factorio]"` You may want to tweak this for your usage, I picked 4 gig. A 4 gig box will run you 40 bucks a month, or $0.06 cents an hour, while a 2 gig instance will cost you $0.03 per hour or 20 bucks a month. Don’t forget to blow the machine away, if you aren’t planing on running the Digital Ocean server 24x7. I take no responsibility for your forgetfulness ;). I haven’t tested this build with anything other then Ubuntu, but I’d be interested if you got it working on other distributions.

When you are done, you can blow it all away with these commands:
```
~$ SERVER=`chef exec knife digital_ocean droplet list | grep factorio | awk -F ' ' {'print $1'}`
~$ chef exec knife digital_ocean droplet destroy -S $SERVER # This destroys the machine on Digital Ocean
~$ chef exec knife node delete factorio -y && chef exec knife client delete factorio -y # This deletes it from Hosted Chef
```

If you’ve completed these steps, you won’t need to login to the Host Chef instance unless you want to check the cookbook.
You should be able to just spin up your factorio server with just `chef exec knife digital_ocean droplet create --server-name factorio --image ubuntu-14-04-x64 --location sfo1 --size 4gb --ssh-keys <YOURSSHKEYNUMBER> --bootstrap --run-list "recipe[factorio]"`.
And use the above commands to blow everything away.
