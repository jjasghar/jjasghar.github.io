---
layout: post
title: "boxen is neat but man chef-solo was so much easier"
date: 2014-01-25 00:24:28 -0600
comments: true
categories: chef linux sysadmin puppet
---

So over the 2013 holiday break I played around with [boxen](http://boxen.github.com/), I even wrote a couple beginner posts on it. If you want go check them out, I'll wait.....

So, yeah, you can tell I was pretty impressed with it right?

Well from a guy that uses [chef](http://getchef.com) day in and day out, using it in a real life scenario it was a nightmare.

First off, I can understand some of the choices they made; like putting everything in `/opt/boxen`. Alas the disadvantage is that well EVERYTHING is in `/opt/boxen`. My company leverages [homebrew](http://brew.sh/) for all our base development software, and it seems that boxen installs the binary at `/opt/boxen/homebrew`. This in turn confuses the hell out of the seasoned admin AND the new person developer trying just do a simple `brew install <someapp>`. No joke, I spent 2-3 hours trying to figure out why boxen said homebrew was installed but I couldn't simplify just use `brew`. I ended up symlinking like crazy not realizing that I hand to `source /opt/boxen/script/env.sh`.  Turns out it seems I completely missed that small note in the docs/wrapper script, saying you need to source it. *sigh*

Along with that nasty turn, one of the selling points was my ability to `git clone` down my companies code from Github. Yeah that didn't work at all. I forgot how hard it was to make a damn directory with puppet, then to use the [puppet-repository](https://github.com/boxen/puppet-repository) and have it fall on it's face only frustrated me even farther. Strike two boxen, strike two.

The final strike came pretty simply but I didn't realize how much of a pain it was. The [puppet-mysql](https://github.com/boxen/puppet-mysql) runs it's socket file in both a completely different location and runs a completely nonstandard port to connect to it. Now to a sysadmin like myself, this seems negligible, neigh, not even negligible,  but put this in front of a Developer that has to change his `database.yml` and all hell breaks loose.   And to top that off, because the database.yml is part of the code base, if said Dev decides to push it back upstream it opens up a can of worms asking why we have to have something like the following for everyone:

```ruby
<%
  socket = [
    ENV["BOXEN_MYSQL_SOCKET"],
    "/var/run/mysql5/mysqld.sock",
    "/tmp/mysql.sock"
  ].detect { |f| f && File.exist?(f) }

  port = ENV["BOXEN_MYSQL_PORT"] || "3306"
%>

development: &development
  adapter: mysql
  database: yourapp_development
  username: root
<% if socket %>
  host: localhost
  socket: <%= socket %>
<% else %>
  host: 127.0.0.1
  port: <%= port %>
<% end %>

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *development
  database: yourapp_test
```

Yeah that ain't worth it AT ALL.  Now that I've got my bitching out of the way, on to the happiness that is chef.

I took what I learned from boxen, and took the plunge into chef-solo to fix this issue. I recalled attempting to use chef-solo on OSX when I first started and it was also complete disaster. I'm betting I really had no idea what I was doing so that's probably a major factor in it. But now I feel have mastered it on "the linuxes", and hell even wrote a sensu cookbook for windows, so I told myself to take the plunge.  Man, I'm really glad I did; TL;DR: easy pezzy.

I started around noon after deciding that boxen was dead, and by 3pm I had a working framework with a [wrapper script](http://bit.ly/jj_mac), [homebrew](https://github.com/opscode-cookbooks/homebrew), and [chef-solo](http://jjasghar.github.io/blog/2013/08/02/adventures-with-chef-solo/), why oh why didn't I do this originally.

(I pinged Fletcher Nichol about getting test-kitchen working with OSX, he sent me a gist on how to get it working, as of writing this I haven't attempted it yet, but hellz yes I'm gonna kitchen this bitch up very very soon.)

I had some trouble with [rvm](http://rvm.io), automating the install was a bit tricky, but breaking it up into different `execute` and `bash` blocks allowed for solidarity. After that hooking up the [dmg](https://github.com/opscode-cookbooks/dmg) cookbook and boom, I have my [mac provisioning](https://github.com/jjasghar/provision_mac) system.

Take a look, I'm honestly curious on what people think. Oh, take out the `repo.rb` though, it's specific to my company.
