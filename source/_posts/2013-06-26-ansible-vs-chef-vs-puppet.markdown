---
layout: post
title: "ansible vs chef vs puppet"
date: 2013-06-26 13:58
comments: true
categories: chef puppet ansible
---

Intro
----

First off, I'm going to say I'm a chef zealot, but I'm attempting to put together an intelligent discussion of the battle that has popped up in sysadmin discussion groups now-a-days.  Secondly, I'm only focusing on the three systems I have exposure to, ansible the least, puppet the second, and chef the most.  I could also probably throw salt, cfengine, and others in there, but that's superfluous, this is my discussion so I'm going to keep it in my scope.

I'm going focus on the pluses of each, and use the "Overall" section to combine them.  In my conclusion my true thoughts will come out, so if you want the tl;dr I'd skip down there.


ansible
-------

Links:
[homepage](http://www.ansibleworks.com/), a great [tutorial](https://github.com/leucos/ansible-tuto)

- Low overhead
- python based
- playbook based

To quote Wikipedia:
> Ansible is an open-source software platform for configuring and managing computers. It combines multi-node software deployment, ad-hoc task execution, and configuration management. It manages nodes over SSH and does not require any additional remote software to be installed on them. Modules work over JSON and standard output and can be written in any programming language. The system uses YAML to express reusable descriptions of systems.

Probably the neatest thing about ansible is that it only requires ssh to work.  You do the work on a local box, and run the "play book" and it walks through the steps.  its designed to be something fast and quick to spin up boxes.  The disadvantage of this is that creep can happen; you can make sure the box is what you expect but you don't have the focused control of something like chef or puppet.  If you need just a webserver that is configured one way and is disposable then this is the way to go.  If you have to support something for long term though, it seems its out of scope of this product. I could be wrong but that's how I interperat it. 

chef
----

Links:
[homepage](http://www.opscode.com/), a great [tutorial](https://learnchef.opscode.com/)

- gem or [omnibus](http://www.opscode.com/blog/2012/06/29/omnibus-chef-packaging/) installer
- server based or [chef-solo](http://docs.opscode.com/chef_solo.html)
- cookbook or recipe based

To quote Wikipedia:
> chef is a configuration management tool written in Ruby and Erlang. It uses a pure-Ruby, domain-specific language (DSL) for writing system configuration "recipes" or "cookbooks". chef was written by Opscode and is released as open source under the Apache License 2.0. chef is a DevOps tool used for configuring cloud services or to streamline the task of configuring a company's internal servers. chef automatically sets up and tweaks the operating systems and programs that run in massive data centers.

As I said in the intro, I love chef.  Hell if I could I'd probably work for opscode.  As soon as I got my head wrapped around chef (which is chef's main problem), its quick and efficient, and its slick.  I've only ever used chef with a chef server, never really touched chef-solo.  I also leveraged [chef-zero](https://github.com/jkeiser/chef-zero) in my testing because I like the idea of uploading the cookbooks and pulling them down.  Having cookbooks on a centralized chef server, its a true "system of record" that doesn't rely on your laptop or workstation, if you do a `knife server create 'role[web]'` you know what you are going to get. 

Something I would like to say though, a huge disadvantage of chef is the way cookbooks are controlled.  It requires constant babying, or plugins like [knife-spork](https://github.com/jonlives/knife-spork) to make sure people don't mess with each others work.  I am lucky enough to work in an environment with only 2 others that have any real use for chef, so I am able to be a good gatekeeper.  I could only imagine the cluster with larger teams.  As of 6/26/13 I know this is a conversation topic and pain point, I'd be interested to see how the workflow matures.

puppet
------

Links:
[homepage](http://www.puppetlabs.com/), a great [cheat sheet](http://docs.puppetlabs.com/puppet_core_types_cheatsheet.pdf)

- an official [learning](http://info.puppetlabs.com/download-learning-puppet-VM.html) vm
- [package](https://puppetlabs.com/puppet/puppet-open-source/) based
- server based or local puppet-apply
- manifest based

To quote Wikipedia:
> puppet is a tool designed to manage the configuration of Unix-like and Microsoft Windows systems declaratively. The user describes system resources and their state, either using puppet's declarative language or a Ruby DSL (domain-specific language). This information is stored in files called "puppet manifests". puppet discovers the system information via a utility called Facter, and compiles the puppet manifests into a system-specific catalog containing resources and resource dependency, which are applied against the target systems. Any actions taken by puppet are then reported.

Something I've always loved about puppet is the trifecta. You can easily create a manifest to control the package/file/service in only a few lines.  (check that cheat sheet above for an example) I was able to take over an infrastructure in an extremely short period of time with just that.  I didn't have that long to control every facet of the groupings but the investment to the reward was mind boggling fast. 


Overall
------

Ok, now that my little tl;dr of the pluses are done, I can start my views overall.  puppet is a very "eins, zwei, eins, zwei," type application.  With marrying it with something like [cobbler](http://www.cobblerd.org/) you can in theory rack mount a machine, hit the power button and have a machine exactly how you want it to be in "x" about of time.  This is great for big companies that always want machines to cookie cutter. It could be that that's how I saw puppet being used, but comparing them to the two other config management systems; it really fits. Speaking of cobbler, yes, I do understand you can leverage cobbler with each of these tools, but as you can [see](https://github.com/cobbler/cobbler/wiki/Using%20cobbler%20with%20a%20configuration%20management%20system) it just seems that cobbler and puppet are a match made in heaven.  chef on the other hand seems like it's designed for smaller companies with smaller teams.  It looks for cooperation, nay, requires cooperation, and its to its strength and its weakness.  Unlike puppet, chef doesn't default to an "active" mode (Ok, that's not fully true about puppet either, but the better practice is to have puppet in active mode from the get go).  puppet runs by default every 30 mins, to make sure its what you want to be, chef on the other hand is a pull based config-management that they even suggest "putting chef-client" in your crontab.  This originally bothered me, coming from the puppet world, but as i learned more about chef i realized that isn't its bread and butter.  chef is a way to spin up boxes for an elastic environment like AWS.  You have a massive blast of traffic, chef will trigger throw your machines in the load balancer, install apache or whatever, and you have what you expect, and 2-3 hours later you blow it away.  This is different than ansible because it is not "automated" per se. Remember the play books site in your local machine, you have interact with the provisioning, which for a startup seems like the correct path. In a startup you have to know exactly what machines are running, not only because of OPEX but because its your baby, you want focus and tweak; you *probably* don't have the budget to just blow away the boxes.  


Conclusion
---------

So I'll say this because I believe this: each of these tools are for different "size" infrastructures.  You can argue as much as you want; saying that "yes well you can use each in of the different sizes", but they were all seem to have a natural fit.  puppet for the corporate/enterprise space, chef for the the small corporate/budding-enterprise space, and ansible for the start ups.  
