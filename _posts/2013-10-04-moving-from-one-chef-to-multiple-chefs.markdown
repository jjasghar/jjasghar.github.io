---
layout: post
title: "Moving from one chef to multiple chefs"
date: 2013-10-04 14:00
comments: true
categories: linux chef sysadmin
---

There's a organic growth of a Ops team using chef goes through.  You start out with a `chef_repo` then you post it to github/some DVCS.  From there you tell people to clone it down and put PRs against it. From there you attempt to be a gate keeper, looking at the commits in the log, realizing that there is no match ups. From there you say to yourself "Hey, ok, these guys are pretty smart, as long as I spot check, they should be able to merge things in and I can trust them right?" NO you are lying to yourself you just don't realize it yet.

Wait for that one day that you think you have the newest cookbook and you bump the version, add your changes and you upload, and push...and nothing happens. Oh I'm 2x behind where I thought great, bump and push. Oh...it seems there was a critical fix in that one that I didnt get for that exact cookbook I uploaded...crap. (And other situations like this can pop up.)

So, I pinged [Nathen Harvey](https://twitter.com/nathenharvey) at Opscode asking for some guidance, he suggested [knife-spork](https://github.com/jonlives/knife-spork).  So far with my initial tests it looks like it is the correct answer,  so I'm also writing this as a HOWTO for my company so this is just a run down of how to use it. (A cheatsheet to the README.md if you will.)

Installation
------------
Obviously the first thing you need to do is install it. Luckily it's a gem so you can just do the following. If you read the docs there are a bunch of places that you `.yml` gets read from, but I chose this because I like having all my chef stuff in `.chef` so I don't have to think about pulling anything other than `.chef` if I want to move boxes.

```bash
gem install knife-spork
touch ~/.chef/spork-config.yml
```

After installing the gem and touching the file, you can run `knife spork info`, it should say everything is disabled.  If so, then you are read to create the config file.

The example [config](https://raw.github.com/jonlives/knife-spork/master/README.md) is on the main site, but I copied the demo one here too.

```yaml
default_environments:
  - development
  - production
environment_groups:
  qa_group:
    - quality_assurance
    - staging
  test_group:
    - user_testing
    - acceptance_testing
version_change_threshold: 2
environment_path: "/home/me/environments"
plugins:
  campfire:
    account: myaccount
    token: a1b2c3d4...
  hipchat:
    api_token: ABC123
    rooms:
      - General
      - Web Operations
    notify: true
    color: yellow
  jabber:
    username: YOURUSER
    password: YOURPASSWORD
    nickname: Chef Bot
    server_name: your.jabberserver.com
    server_port: 5222
    rooms:
      - engineering@your.conference.com/spork
      - systems@your.conference.com/spork
  git:
    enabled: true
  irccat:
    server: irccat.mydomain.com
    port: 12345
    gist: "/usr/bin/gist"
    channel: ["chef-annoucements"]
  graphite:
    server: graphite.mydomain.com
    port: 2003
  eventinator:
    url: http://eventinator.mydomain.com/events/oneshot
```

All in all this seems pretty self [explanatory](https://github.com/jonlives/knife-spork#default-environments) but the most important things to change are `environment_path` and disabling the plugins (by removing them) here.  For my company I only used the git plugin and...well that was it. :)

By the way there are only a few plugins, [here](https://github.com/jonlives/knife-spork/tree/master/plugins) is a link to the different .md files on each.

Ok, so you have everything set up, what do you do now?

Usage
-----

The first step is to run `knife spork check COOKBOOK --all` where COOKBOOK is one of your commonly updated/tweaked cookbooks.  Spork checks against what you have locally compared to what's in the server, like this:

```
knife spork check COOKBOOK --all
```

Here's an example:
```bash
Checking versions for cookbook nagios...

Local Version:
  5.1.5

Remote Versions: (* indicates frozen)
  5.1.5
  5.1.4
  5.1.3
  5.1.2

ERROR: The version 5.1.5 exists on the server and is not frozen. Uploading will overwrite!
```

As you can see with the error, it's pretty self explaintory.  

The second step is to bump the version:
```bash
knife spork bump nagios patch
Git: Pulling latest changes from /Users/jasghar/repo/chef_repo/environments
Pulling latest changes from git submodules (if any)
Git: Pulling latest changes from /Users/jasghar/repo/chef_repo/cookbooks/nagios
Pulling latest changes from git submodules (if any)
Successfully bumped nagios to v5.1.6!
```

Now as you can see I have the git plugin working, and it without thinking about it, updates the metadata.rb so you don't have to. (I HATE that part of chef, I always forget.) Now you can go off make your changes.

From here you're happy, you've commited your new `chef_repo` back to the DVCS that you use.  This is where the magic happens: `knife spork upload COOKBOOK`.  This thing is great, it (to quote the `README.md`) _This function works mostly the same as normal knife cookbook upload COOKBOOK except that this automatically freezes cookbooks when you upload them._ Which is the bread and butter of spork.  The freezing is crazy important, by freezing the upload you take "ownership" of that version of the cookbook.  Your changes are yours, and no-one can mess with them.  So in turn you don't step on your coworkers toes and he doesen't step on yours.

Ok, so this was the cheatsheet, we have implymented this at my company now, so hopefully this goes great. If not...well that's a different conversation.
