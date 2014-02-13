---
layout: post
title: "chef handler why did it take a year for me to use you"
date: 2014-02-07 15:24:54 -0600
comments: true
categories: chef sysadmin
---

Something that I never seemed to grok was [chef handlers](http://docs.opscode.com/essentials_handlers.html). I read that site like 50 times, got the jist, but never fully understood them.

Then I learned it in a nutshell: you know when you run `chef-client` or `chef-solo` and that error that pops up? A handler can export it to another location so you can have a centralized stream of information.

This isn't just limited to error reporting, you can report a bunch of stuff, but errors are something that admins should visualize ASAP.

This is AMAZINGLY useful if you have more than a couple nodes. Lets say you have 100-200ish nodes (like me) and you decided that you wanted to run `knife ssh 'chef_environment:staging' 'sudo chef-client'`.


Awesome, so it kicks off, you have a few changes but you had a typo in your `role[db_slave]`. Well, if you've ever used `knife ssh` before you'll know that it streams everything real-time 
and you'll probably miss the nasty red text that says you have an error. If you have your handler; it takes it says "Hey error here, lets push it up to [email|campfire|growl|hipchat|etc]" and boom everyone knows that 
there's a problem with the converge. If you are lucky enough to have a regular checkin with the chef-server you'll get the error pop up each run till you have it fixed. Annoying yes, but it makes sure you have all you machines 
in a good state.

If this has peaked your interest, there is a cookbook called [chef_handler](https://github.com/opscode-cookbooks/chef_handler) that does most if not all of your heavy lifting. It leverages an LWRP that needs to be close if not the first in your `run_list`.

Here's an example for [chef-handler-mail](https://github.com/kisoku/chef-handler-mail):

```ruby
chef_gem "chef-handler-mail"
gem "chef-handler-mail"

chef_handler "MailHandler" do
   source 'chef/handler/mail'
   arguments :to_address => "root"
   action :nothing
end.run_action(:enable)
```
As you can see it's pretty straight forward.

My company uses [campfire](http://campfirenow.com/) and there's a [chef-handler-campfire](https://github.com/ampledata/chef-handler-campfire) gem for it. The LWRP stanza that you put in your recipe requires something to this effect, I put this in the
first main internal recipe:

```ruby
include_recipe 'chef_handler'

chef_gem "chef-handler-campfire" do
  action :install
end

chef_handler 'Chef::Handler::Campfire' do
  action :enable
  arguments [ 'SUBDOMAIN', 'TOKEN' , 'ROOM' ]
  source File.join(Gem.all_load_paths.grep(/chef-handler-campfire/).first,
                   'chef', 'handler', 'campfire.rb')
end
```

Note: Unfortunately the newest [tinder](https://github.com/collectiveidea/tinder) requires `json ~> '1.8.0'` and chef `11.8.2` requires `json ~> '1.7.7'`.  Last night (2014-02-06) chef [released 11.10](http://www.getchef.com/blog/2014/02/06/chef-client-11-10-0-release/) which fixes this, the embedded chef binary now uses 1.8.0. I have a [PR](https://github.com/ampledata/chef-handler-campfire/pull/2) to fix up the README.

There's a lot more to handlers, but for now this is huge win for me, I'll post more as I start learning the reporting side of it.
