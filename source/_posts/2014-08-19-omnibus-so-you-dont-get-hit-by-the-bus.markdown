---
layout: post
title: "Omnibus so you dont get hit by the bus"
date: 2014-08-19 13:34:45 -0500
comments: true
categories: chef sysadmin omnibus
---

I recently took a new job at [Chef](http://getchef.com) Inc. Needless to say I'm challenged and excited. I took a position as the
[openstack](http://openstack.org) guy, which, again needless to say is a mountain of work. Part of my first responsibilities was to
start building an integration framework for the [stackforge](https://github.com/stackforge/) cookbooks. Part of this was to automate
the building and destroying of compute nodes with different hardware configurations, so we needed a ipxe/tftp setup to network
boot machines. We decided on a project from CSC called [hanlon](https://github.com/csc/Hanlon) which does a great deal of this work.

You're probably asking by now, so where's omnibus come into play with this? Well hanlon moves extremely fast and we needed a way to
package it up. There are specific dependencies and other specific things that having a package with "blessed" versions of the other
underlying apps would help support. If I recall correctly, [sensu](http://sensuapp.org) came to this same decision because even as small
as patch numbers for ruby can cause havoc attempting to support someone.

So on with the show/tutorial:

## Omnibus

The first thing you need to know is that [Omnibus](https://github.com/opscode/omnibus) is actually in two pieces. First the framework,
at this repo: [omnibus](https://github.com/opscode/omnibus) and the [omnibus-software](https://github.com/opscode/omnibus-software)
which is the the building blocks for whatever you're trying to package. 

The first thing that got me learning omnibus was the two files that sat inside of `project/` and `software/`. You'll notice that they
probably are both named your package and probably wonder why it's named the same thing. The best description is this; `project/blah.rb` is the
overarching definition of the package where as `software/blah.rb` is the build instructions. You shouldn't repeat the data is both places,
because it can cause issues supporting the pkg in the long run. Keep that in mind.
You might notice a `dependency` line `project/blah.rb`, I suggest commenting it out and putting all the dependencies in `software/blah.rb`
is so it's all in one place.

So let's take a step back. How do you create a new project? Here's a snippet to do it:

```bash
$ gem install omnibus
$ omnibus new $MY_PROJECT_NAME # this will create a directory with omnibus-$MY_PROJECT_NAME
$ cd omnibus-$MY_PROJECT_NAME
$ bundle install
```
Something pretty straight forward. I want to stress though the portion of `$MY_PROJECT_NAME`, that caught me a couple times. Yes, a couple times.

So lets say you are on a Ubuntu box and you're trying to build the pkg for Ubuntu. You can short circuit this by just doing the following,
you won't see too much go by, but it'll tell you when it's done.

```bash
$ cd omnibus-$MY_PROJECT_NAME
$ bundle exec omnibus build $MY_PROJECT_NAME
```

Now if you want to see more things go by,

```bash
$ cd $MY_PROJECT_NAME
$ bundle install
$ bundle exec omnibus build $MY_PROJECT_NAME --log-level=debug
```

I personally like it, I like to see my computer work ;).

Now lets say you're still on that Ubuntu box but your coworker is using Centos, this is where [test-kitchen](http://kitchen.ci) can take part making your
life so much easier.
You should do something like the following:

```bash
$ bundle exec kitchen list
$ bundle exec kitchen converge <os you want to converge to build the pkg>
```

This will set up the box via test-kitchen, you'll want to `kitchen login` the box but then you can do the above commands to build the pkg.

Something I noticed as I was attempting to leverage test-kitchen as my build box, (I'm on a Mac trying to make Ubuntu pkgs) the following made my life
much easier.

I added the following to my `~/.ssh/config`.
```bash
host localhost
     User vagrant
     IdentityFile ~/.ssh/vagrant
```
Where the `~/.ssh/vagrant` is from [here](https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant)

Then you can do something like, then ship it off to some place else. NOTE: check the port number ;)
```bash
$ mkdir ~/tmp && cd ~/tmp
$ scp -P 2201 vagrant@localhost://home/vagrant/$MY_PACKAGE_NAME/pkg/* ./
```

And say you spin up another vagrant box to test. NOTE: check that port number ;)
```bash
$ scp -P 2202 $MY_PACKAGE_NAME_0.1.0+20140819210922-1_amd64.deb vagrant@localhost://home/vagrant/
```

I ran into an error during my adventures with Omnibus, and figured I should capture it here:

If you see an error like the following:
```ruby
              [NullFetcher: libgcc] I | Fetching `libgcc' (nothing to fetch)
              [NetFetcher: cacerts] I | Downloading from `http://curl.haxx.se/ca/cacert.pem'
              [NetFetcher: cacerts] I | Verifying checksum
Verification for cacerts failed due to a checksum mismatch:
 
    expected: fd48275847fa10a8007008379ee902f1
    actual:   c9f4f7f4d6a5ef6633e893577a09865e
 
This added security check is used to prevent MITM attacks when downloading the
remote file. If you have updated the version or URL for the download, you will
also need to update the checksum value. You can find the checksum value on the
software publisher's website.
```
A quick fix would be to blow away your `Gemfile.lock` and run another `bundle install`.

So here's some notes on Omnibus, I'll probably add more as time progresses, but this is at least a start!
