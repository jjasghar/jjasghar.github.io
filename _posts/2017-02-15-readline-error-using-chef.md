---
layout: post
title: "readline error using chef"
date: 2017-02-15 08:31:11
categories:
---

There are times where I don't use the [chefdk][chefdk]. Yes, I know this is bad,
but I do `gem` development! I use `chruby` and it seems I've run into this issue
a couple times now. I was working on the `knife-rackspace` gem, and hit this:

```bash
08:26:46 JJs-MacBook-Pro ruby-tmp > be knife rackspace server list
bundler: failed to load command: knife (/Users/jjasghar/ruby-tmp/.direnv/ruby/bin/knife)
LoadError: dlopen(/Users/jjasghar/.rubies/ruby-2.3.1/lib/ruby/2.3.0/x86_64-darwin15/readline.bundle, 9): Library not loaded: /usr/local/opt/readline/lib/libreadline.6.dylib
  Referenced from: /Users/jjasghar/.rubies/ruby-2.3.1/lib/ruby/2.3.0/x86_64-darwin15/readline.bundle
  Reason: image not found - /Users/jjasghar/.rubies/ruby-2.3.1/lib/ruby/2.3.0/x86_64-darwin15/readline.bundle
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/knife-rackspace-1.0.3/lib/chef/knife/rackspace_base.rb:35:in `require'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/knife-rackspace-1.0.3/lib/chef/knife/rackspace_base.rb:35:in `block (2 levels) in included'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/chef-12.18.31/lib/chef/knife.rb:232:in `block in load_deps'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/chef-12.18.31/lib/chef/knife.rb:231:in `each'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/chef-12.18.31/lib/chef/knife.rb:231:in `load_deps'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/chef-12.18.31/lib/chef/knife.rb:216:in `run'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/chef-12.18.31/lib/chef/application/knife.rb:156:in `run'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/gems/chef-12.18.31/bin/knife:25:in `<top (required)>'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/bin/knife:23:in `load'
  /Users/jjasghar/ruby-tmp/.direnv/ruby/bin/knife:23:in `<top (required)>'
```

The most important part of it is this:

```
LoadError: dlopen(/Users/USERNAME/.rubies/ruby-2.3.1/lib/ruby/2.3.0/x86_64-darwin15/readline.bundle, 9): Library not loaded: /usr/local/opt/readline/lib/libreadline.6.dylib
```

It seems I rebuilt something that leverages `libreadline` and it broke. There's
a couple ways to fix this, but the way I've landed on is adding a `gem` to my Gemfiles.

For instance:

```ruby
source 'https://www.rubygems.org'

gem 'yard'
gem 'rake'
gem 'rb-readline'
gem 'knife-rackspace'
```

The `rb-readline` seems to do the trick!

[chefdk]: https://downloads.chef.io/chef-dk
