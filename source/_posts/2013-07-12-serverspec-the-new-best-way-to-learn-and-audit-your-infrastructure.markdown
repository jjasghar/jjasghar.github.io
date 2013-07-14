---
layout: post
title: "serverspec the new best way to learn and audit your infrastructure"
date: 2013-07-12 15:06
comments: true
categories: linux chef puppet ruby sysadmin
---

Listening to the [Opscode](http://www.opscode.com) comunity pod cast, the [foodfight](https://twitter.com/foodfightshow) show, [bryanwb](https://twitter.com/bryanwb) suggested more love for something called [serverspec](http://serverspec.org).  A couple days later he wrote a [tweet](https://twitter.com/bryanwb/status/340155947634794496) and I figured I'd give it a shot.  I've been blown away by it; and I'm going to sell/leverage it where ever I can.

serverspec is basically [rspec](http://rspec.info/) for sysadmins, from what I've tested basic rspec (and some advanced rspec) assertions work like a charm. Here's a [cheat sheet](https://gist.github.com/byplayer/965857) if you want.  You can write tests like ruby developers do, but leverage them for your infrastructure.  As I've been learning more about TDD, I see the advantage, but to put that style in sysadmining, might seem initially crazy.

As a new sysadmin, you've probably had an experience something like this:
You come in at a new job, you have the initial "Hey newguy, here's your new network.  You have web machines, databases, app servers, caching servers, configuration management and other boxes. We have process monitoring *ahem*nagios*ahem* and yes we have configuration management where you have to leverage code reviews to get anything out to production." As a new guy I'm excited this is great, but here comes the kicker.  Your new trainer continues, "Ok, lets walk through each machine.  There are 10 web servers, 9 of them are called www1 through www8, and one called paco." You quickly grab a piece of paper, "Paco?", "Yes the admin before you didn't like standard names, he wanted to give machines 'personality', and we had a management change after him, so we have that one off."


This is where serverspec comes into play.  You know that list that starts to grow, learning about your app servers, your one off web machines? If you use serverspec, you can keep the pseudo-english list and be able to check against what should be on there. It  gives you a way to confirm everything is exactly what you expect.

serverspec only requires ruby to run, the following command will get it installed if you have ruby 1.9.x+ installed. ```gem install serverspec``` and ssh access to the boxes you want to check.  The example of the webserver on the main page is priceless and speaks volumes. The following is that example with a small edit from me.
``` ruby
require 'spec_helper'

describe package('httpd') do
  it { should be_installed }
end

describe service('httpd') do
  it { should be_enabled   }
  it { should be_running   }
end

describe port(80) do
  it { should be_listening }
end

describe file('/etc/httpd/conf/httpd.conf') do
  it { should be_file }
  it { should contain "ServerName my-server-name" }
end
```
That's it, now all you have to do is `rake spec` and you have a test that checks for all the basic apache web server stuff, and instead of sshing out to each machine you can make the computer do the work for you, and this is just the tip of the iceberg!

nagios (or automated process monitoring) vs serverspec
-----------------------------------------
I understand the hesitation about _another_ monitoring solution, but every sysadmin goes through this situation like this when learning a new network.  Like the story above, there should be at least one rudimentary monitoring solution, a way to  make sure machines are up and maybe processes aren't erring.  With the low overhead of serverspec you can leverage this in an ad-hoc way and, I'd like to state that this isn't designed to take place of something like nagios, but to augment it. 

The advantage of something like serverspec over nagios or, a polling based monitoring solution, is that there is a lag between when you deploy something and the monitoring systems reporting it.  You can run serverspec in an ad-hoc manner you'll get a much faster feedback loop.  I have 1034 tests on about 40 boxes, and it only takes about 2 minutes 30 seconds to run, needless to say that's a small price to pay for a piece of mind.  The default polling for nagios is 5 minutes, so cutting it in half that's already a great turn around to see the state of the machines.

I've also able to get serverspec working with jenkins, and run the `rake` task every 30 minutes, and if it exits on anything other than a 0, it'll email out the configuration or error that it finds. I'm still experimenting with jenkins and serverspec, but so far it's nice to have something other than cron running this.

Conclusion
----------
serverspec is a tool/gem that I'll be leveraging from now on.  I've been wanting to learn rspec and this gives me an avenue to use it day to day; and get a huge benefit from it.  I even bought [The RSpec Book: Behaviour-Driven Development with RSpec, Cucumber, and Friends](http://pragprog.com/book/achbd/the-rspec-book) by The Pragmatic Programmers, so hopefully I'll be able to leverage it to get to the next level. Give it a shot, and when you get all your green dots come back, you'll know you've found a winner.
