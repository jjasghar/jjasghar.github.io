---
layout: post
title: ".kitchen.local.yml and when you want to use it"
date: 2014-02-17 15:32:20 -0600
comments: true
categories: sysadmin chef test-kitchen
---

I recently bought the vagrant [vmware fusion](http://www.vagrantup.com/vmware) plugin to start testing out [test-kitchen for mac](https://gist.github.com/fnichol/8609348). Good ol' [Fletcher](https://twitter.com/fnichol)
made it ridiculously easy to do and I thank him for that.  Interestingly enough though in the process of figuring it out I ran into a problem.  I have a few other `.kitchen.yml` files in different
cookbooks, and I wanted to start leveraging what I just paid for.

So take this `.kitchen.yml` for instance:

```
---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
  - name: ubuntu-12.04

suites:
  - name: default
    run_list:
      - recipe[hubot-solo::default]
    attributes:
```

Yep, it runs off the default of virtualbox.  If you create a `.kitchen.local.yml` file in that directory, something like this:

```
---
driver:
  name: vagrant
  provider: vmware_fusion

provisioner:
  name: chef_solo

platforms:
  - name: ubuntu-12.04

suites:
  - name: default
    run_list:
      - recipe[hubot-solo::default]
    attributes:
```

It'll run it with vmare as the hypervisor or provider, in kitchen lingo.

That's all fine and dandy, but what about an over arching config? I thought you could create something like `~/.kitchen.local.yml` that didn't seem to work.

I pinged [#kitchenci](https://webchat.freenode.net/?channels=%23kitchenci) and __teukka__ gave me
the answer: set/export `VAGRANT_DEFAULT_PROVIDER=vmware_fusion` env var (in your shell's rc file), and boom, it worked.

So, long story short: if you want to have a specific change that overrides the default `.kitchen.yml` make a `.kitchen.local.yml` in the directory,
but if you want to override every hypervisor use `export VAGRANT_DEFAULT_PROVIDER=vmware_fusion` in your bashrc/zshrc.

I hope this helps someone making the conversion from virtualbox to vmware.
