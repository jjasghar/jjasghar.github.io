---
layout: post
title: "Adventures with chef-solo"
date: 2013-08-02 14:36
comments: true
categories: chef linux
---

I started my chef career with inheriting a chef infrastructure with an opensource chef server.  I read all of the blog posts, and the tutorials, tried to wrap my head around what a cookbook, recipe, or data bag was.  I know now, this was the _wrong_ way to learn chef.  chef with chef-client/server adds a level of complexity that just is unfounded.  I decided to spend a copy hours to master chef-solo, and damn I wish I had started there.

My hands on learning project
----------------------------

I had a requirement of setting up a unicorn nginx instance on a VPS.  I thought hell, there's gotta be a way to do this with chef. (I normally use passenger+apache, so I had to learn nginx and unicorn too, but that's a different post...well kinda) So I looked around for a nice [tutorial](http://www.opinionatedprogrammer.com/2011/06/chef-solo-tutorial-managing-a-single-server-with-chef/) and came across that guy.  I started reading it, realized holy crap, this is awesome.  I stole the `solo.json` and the `solo.rb` and `install.sh` because I was lazy.  
I changed the `install.sh` around a little bit:
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
As you can tell, its ubuntu focused, but basically if I want to provision a box after making a change to one of the cookbooks I just had to run `./install.sh`.  Interestingly, I had heard bad things about the 1.9.1 ruby binaries for ubuntu, but so far, it is a ton better than doing that compile from src. ;)

So, basically with those three files, running that curl in the script, and creating a `cookbooks/` directory, I had my framework to make this project happen.

Now I made a mistake here.  I started off with a simple "generic" cookbook, with a simple `default.rb` file that called a `packages.rb` file to add some pkgs I love and run on a daily basis.

```ruby
%w{build-essential git-core libgdbm-dev libreadline-dev libssl-dev libyaml-dev s3cmd tmux tk-dev vim wget zlib1g-dev zsh}.each do |pkg|
  package pkg do
    action [:install]
  end
end
```

Now to an passive onlooker there seems nothing wrong with this, and 3 weeks ago JJ would agree with you.  JJ now says, "Stop that, start with the webserver and the deps for that instead of the convince of something easy like package management."  LEARN [berkshelf](http://berkshelf.com/) and don't make excuses. (at this writing, 8/2/13, I still actually haven't learned it, the best analogy is this, dling cookbooks is like building from source, and berkshelf is like using apt-get) Anyway I digress.   

Back to the original story.

I made the `cookbooks/` directory, and when to github to pull down the newest [nginx](https://github.com/opscode-cookbooks/nginx) cookbook.  `git clone https://github.com/opscode-cookbooks/nginx.git` in the directory and then because  I hate submodules, I ran `rm -rf .git/`. I `cd ~/chef-solo` then ran `install.sh`. Crossed my fingers and no, didn't work. Turns out I needed a bunch of dependences (hence the berkself statement). I resolved them all, and then ran the beautiful `./install.sh`. BOOM, it worked.  I took a moment, realized what I had just done.  I created a git repo off this work, created a branch "nginx" and committed my changes. Now whenever I need to spin up an nginx box, all I have to do is clone my chef-solo repo and checkout nginx and run `install.sh`. 

Dang, that's awesome. To get that to work with chef-server...is a ton of work.  You have to download each cookbook, upload each to the chef-server, change the role or add the recipe, then run chef-client.  This would be great if I had to do it over and over, but being I was just setting up a dev box, chef-server overhead just isn't worth it.

Now all in all, this has only been one adventure with chef-solo, but going through this academic exercise I realized the potential of this unbelievable powerful application.  From this project, and my "generic" cookbook, I've ran with it, so now I have a boostrap cookbook with my packages, some of my dotfiles, and other tidbits.  It's nice, and I strongly suggest checking it out.
