---
layout: post
title: "Using the cookbook generator as soon as possible"
date: 2017-08-08 11:54:39
categories: vmware chef sysadmin
---

Something I've started to discuss more often then not is the idea of using a
[cookbook-generator][docscookbook] sooner then later. Unfortunately, if you start
too soon, it can cause some significant confusion to a newbie Chef user. My post
here is going to do the best I can to explain why you should and give you a clear
understanding why you need to put it in your workflow as soon as possible.

## Why?

This spurs from my conversations with VMware based companies. If you have used
[chef-provisioning-vsphere][cpvsphere] to do any [test-kitchen][kitchen] work,
you'll quickly realize that getting a `.kitchen.yml` that works is hard. (If you're
curious here's an [example kitchen.yml][examplekitchen] for one of my development
back-ends.) As you can see, figuring out all of those options can be overwhelming
and frustrating. I actually found myself at one company posting it to their internal
wiki, and quickly realizing that that was the wrong way to share that gem of an
incantation.

Here is another possible situation. You have grown your team that writes Chef cookbooks,
but you start noticing that your documentation starts to become lackluster like
most projects. ;) Using a standard cookbook generator with specific `TODO`s in it,
will not only force good practice but encourage it.
Here is an [example][cpcptodo] of this for the Chef Partner Cookbook Program that I wrote up for the
same type of situation from a generic community standardization stand point.
Also, you can take it to the next step too, set up a simple CI job
that looks for `TODO:` and fails when it greps out that line. As I like to say let the bots do the work
for you and shame people into documenting their code correctly.

## How?

There are a couple ways to get a generator to be built. You can use the [chefdk][chefdk]
to build it for you with the following command:

```bash
$ chef generate generator <cookbook-name>
```

This gives you a generic generator, that you can start from scratch. Personally, I think it's
a tad bit too stripped down for most, so lets start with something that is a bit more complete.

```bash
~ $ git clone https://github.com/chef-partners/cookbook-guide-generator.git
~ $ mv cookbook-guide-generator <cookbook-name-generator>
~ $ cd <cookbook-name-generator>
~/<cookbook-name-generator> $ rm -rf .git/
~/<cookbook-name-generator> $ git init
```

This will pull down the Chef Partner Cookbook-guide generator, remove the git history, and initialize
a new git repository, giving you a directory with many more options.

Lets start with what the generator gives you, run the following command to see what you get:

```bash
~ $ chef generate cookbook -g ~/<cookbook-name-generator>/cookbooks tempcookbook
~ $ cd tempcookbook
~/tempcookbook $ ls
Berksfile       Gemfile         LICENSE         Rakefile        chefignore      recipes         test
CONTRIBUTING.md Guardfile       README.md       TESTING.md      metadata.rb     spec
~/tempcookbook $ ls
```

As you can see, it's a pretty large framework for a typical community cookbook. Everything ranging
from a `TESTING.md` to a `test/` directory, to even two `.kitchen.yml`s! Now, lets start playing with it.
Go ahead and open up the following:

```bash
~ $ vi ~/<cookbook-name-generator>/cookbooks/code_generator/recipes/cookbook.rb
~ $ # Change the line for .kitchen-docker.yml to .kitchen.docker.yml
~ $ chef generate cookbook -g ~/<cookbook-name-generator>/cookbooks tempcookbook2
~ $ ls -a tempcookbook2/.kitchen*
.kitchen.docker.yml .kitchen.yml
```

Pretty slick eh? Now lets edit the contents of a file. Go ahead and do the following:

```bash
~ $ vi ~/<cookbook-name-generator>/cookbooks/code_generator/cookbooks/code_generator/templates/default/kitchen.docker.yml.erb
~ $ # Edit the line under privileged to: socket: <%= ENV['DOCKER_HOST'] || "localhost" %>
~ $ chef generate cookbook -g ~/<cookbook-name-generator>/cookbooks tempcookbook3
~ $ head -5 tempcookbook3/.kitchen.docker.yml
driver:
  name: dokken
  chef_version: latest
  privileged: true # because Docker and SystemD/Upstart
  socket: localhost
~ $ DOCKER_HOST=tcp://192.168.1.1:2375 chef generate cookbook -g ~/<cookbook-name-generator>/cookbooks tempcookbook4
~ $ head -5 tempcookbook4/.kitchen.docker.yml
driver:
  name: dokken
  chef_version: latest
  privileged: true # because Docker and SystemD/Upstart
  socket: tcp://192.168.1.1:2375
```

As you can see if you have a remote `DOCKER_HOST` or for that matter _any_ `ENV` variables you can have a
sane default and an override if needed. This can take effect if you have a remote `DOCKER_HOST` already
enabled so you know your cookbooks point to the correct location.

As a final example I'd like to point you to the `README.md.erb`, this file is probably one of the most
neglected yet powerful of the generator. The first thing anyone does when looking at a cookbook is read
the README.md. This allows you to fill out and automatically have sections in the README.md to force
required documentation. One of the sections that has slowly become a best practice for cookbook
development is the **SCOPE** section. As you can see from the following it's right there at the
top to make sure it's "scoped" properly.

```bash
~ $ > head -13 tempcookbook/README.md
# tempcookbook

TODO: Enter one line description of the cookbook here.

## SCOPE

TODO: Enter a description of the scope of this cookbook, if you
need an example the [mysql](https://github.com/chef-cookbooks/mysql) cookbook
is a good place to start.

## Requirements

TODO: Enter any requirements for the coobook.
```

Awesome, now that you've played around with your cookbook understood some of the advantages,
the next step is to push this generator to a shared repository. However you share your git
repositories create a repo there and push it.

After that, as time progresses you can add change things to make it more delightful, but
be sure that before people use the `generate` command, they do a `git pull` to get the
most up to date.

[chefdk]: https://chefdk.io
[cpcptodo]: https://github.com/chef-partners/cookbook-guide-generator/blob/master/cookbooks/code_generator/templates/default/README.md.erb
[docscookbook]: https://docs.chef.io/ctl_chef.html#chef-generate-cookbook
[cpvsphere]: https://github.com/chef-partners/chef-provisioning-vsphere
[examplekitchen]: https://github.com/jjasghar/vsphere_testing/blob/master/.kitchen.yml
[kitchen]: http://kitchen.ci/
