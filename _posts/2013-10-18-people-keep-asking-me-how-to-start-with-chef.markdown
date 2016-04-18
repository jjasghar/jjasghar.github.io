---
layout: post
title: "People keep asking me how to start with chef"
date: 2013-10-18 10:19
comments: true
categories: chef sysadmin
---

So as the title says, people keep asking me "How to start with chef?" This an outline of what, if I could go back in time, I would do from the beginning. I completely acknowledge that chef can be extremely confusing to start with. If you really want to learn it you'll have to stick with it, and do it. Good god, nothing is better than running it on a vagrant box and seeing what you expect happen happen.

chef-solo is your best friend (step 1)
--------------------------------------
A lot of people can start here, and end here believe it or not.  chef-solo is unbelievably powerful and can full-fill 90% of all requirements for basic usage. I spent some time looking around for a good tutorial (doing all of them that I could find), and [this](http://www.opinionatedprogrammer.com/2011/06/chef-solo-tutorial-managing-a-single-server-with-chef/) on was the best "I have no idea what the fuck I'm doing." situation.  Modern chef installs are a tad bit different than this guy, so the "install.sh" changes I suggest are this:
```bash
#!/bin/bash

# This run as root on the machine
chef_binary=/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/bin/chef-solo

# are we not bootstrapped?
if ! test -f "$chef_binary"; then
  export DEBIAN_FRONTEND=noniteractive
  apt-get update &&
  apt-get dist-upgrade -y &&
  apt-get install ruby1.9.1 ruby1.9.1-dev make curl -y &&
  curl -L https://www.opscode.com/chef/install.sh | sudo bash
fi &&

chef-solo -c solo.rb -j solo.json
```
As you can see I use the omnibus installer, not the gem, and choose the ruby version you want ;), and yes if you use those lame-ass rpm-based distros, `apt-get` won't work for you.

There's something clever here (step 2)
--------------------------------------
So awesome, you walked through the suggested tutorial, you can now run your `install.sh` and get your changes in. Grats! If you think about it, hit up the [Opscode Docs](http://docs.opscode.com) and you'll discover that there's a resource to provision every which way. As I said at the beginning this might be all you need, if it is then use it; no real need to go any farther.

Ah, you're still here. So you DO want to farther, awesome. In step two I suggested going to the docs, that's cool, but sometimes you need more verbose help, that's my second point, it's time to start asking questions. Your first stop is [#chef](http://webchat.freenode.net/?channels=chef); it's manned basically 24x7, and _normally_ extremely helpful. Don't be a douche, if you have to paste something use [gist](http://gist.github.com) or something.  After that the main ~~Opscode mailing list~~ [discourse](https://discourse.chef.io) is great. It's slower, but you get much more in depth questions and conversations. Finally the third sub step is speak up, ask questions the only way to learn this is to be like "I don't understand it, help!"

The only book worth a damn as of 2013/10/18 (step 3)
----------------------------------------------------
Step three of the journey is probably the one that most people jump to initially, and this is usually where the confusion starts.  There's a handful of books out there on chef, this [one](http://www.packtpub.com/chef-starter/book) is the only one worth any money.  With a strong understanding of how to provision a simple box, and where to ask questions this book will be extremely straight forward and build upon those building blocks. I'm constantly looking for another chef bible, but most nuggets of how-to things are spread all over the internet in blog form.

The fun starts here (step 4)
----------------------------
Step four you need a chef server, you need to be able to provision multiple boxes, you understand/can find out what a role or environment is, and you need different `run_lists`.Good for you.  From here you should look at the open source [chef server](http://www.opscode.com/chef/install/) and spin it up on another box. I should say you can use the [hosted chef](https://getchef.opscode.com/signup), you get up to 5 nodes with it for free, which is cool, but if you want to see everything work from the ground up, open source chef server is the way to go. (NOTE: if you are doing it in AWS/`$cloudprovider` you'll need at least a 4 gig box, and that's pushing it. You've been warned.) Now spin up another box, a machine that can talk to the server that you want to provision. Start playing with `knife` add a knife plug-in for you `$cloudprovider` see if you can spin up another box using the [knife bootstrap](http://docs.opscode.com/knife_bootstrap.html).  Start using [minitest-handler-cookbook](http://community.opscode.com/cookbooks/minitest-handler), [test-kitchen](https://github.com/opscode/test-kitchen), and even [chef-spec](https://github.com/acrmp/chefspec) if your feeling sassy. If you've made it this far, you've probably been exposed to a myriad of other tools, run with them. Trust me if someone built if for chef the chance of being helpful is extremely high.

D'oh why did I do it this way? (step 5)
---------------------------------------
Step five is pretty straight forward. GOTO 10. With everything you now have on your tool belt, you'll want to go back to your original chef-solo recipes and refactor everything. You'll want to add your minitests for integration testing to confirm everything is what you expect and much much more, that I'm at a loss of listing out here. The only way to get good with chef is to do it, hack at it and wait for that converge to work. You'll probably love test-kitchen probably by this point.

This has been my cycle of working with chef, it's hard, confusing and honestly sometimes extremely annoying; though on the other hand the community is great, it's constantly changing, and adding great tools to make your life easier. When you finally get that recipe that builds that box exactly how you want it, theres nothing better to know it's always there and you never have to think about it again.
