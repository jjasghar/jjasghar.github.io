---
layout: post
title: Customizing Boxen
date: 2014-01-01 13:27:20 -0600
comments: true
categories: sysadmin linux puppet
---

My goal is to get these few apps on my vm, but first lets get a default build working:

*   iterm2
*   dropbox
*   mysql
*   1password
*   virtualbox
*   vagrant
*   chrome

With my [fork](https://github.com/jjasghar/our-boxen)...now it's time to start messing around with it. Go ahead and click [here](https://github.com/jjasghar/our-boxen/tree/master/modules/people) this is for personal manifests. This is something I'll cover in the second half of this post. As I was walking through this, it seems that "projects" are just a shared collection of manifests also. So if you learn how to make a personal manifest, a project on isn't much more!

Where we want to start is at the default [site.pp](https://github.com/jjasghar/our-boxen/blob/master/manifests/site.pp) this is what will be installed on all your boxes by well, default.

Ok, first thing first, get a base image running again, open up terminal, and checkout boxen from your fork, (this is my fork and the commands I use from a base install):
```
[~] % git --version # click "Install" for the developer tools
[~] % sudo mkdir -p /opt/boxen
[~] % sudo chown ${USER}:staff /opt/boxen
[~] % git clone https://github.com/jjasghar/our-boxen.git /opt/boxen/repo
[~] % cd /opt/boxen/repo
[/opt/boxen/repo] % git checkout -b removing_nodejs_old_ruby
[/opt/boxen/repo] % cd manifests
[/opt/boxen/repo/manifests] % vim site.pp
```
I don't care for nodejs or the older versions of ruby for this example, so I'm going to remove lines 66-69 and 72,73 of that site.pp file. If you noticed I created a branch for this too, so I'll push this up to my repo so I can track my work.

Ok, lets give boxen a shot!
```
[/opt/boxen/repo/manifests] % cd ..
[/opt/boxen/repo/] % script/boxen --no-fde
[/opt/boxen/repo/] % source /opt/boxen/env.sh
[/opt/boxen/repo/] % boxen
```
You'll notice a warning about auto-update, that's fine, you are developing manifests right?

Go ahead and run these commands on the command line, if our changes worked as expected it should say:
```bash
jjasghar-Mac:repo jjasghar$ node
-bash: node: command not found
jjasghar-Mac:repo jjasghar$ ruby --version
ruby 2.0.0p247 (2013-06-27 revision 41674) [universal.x86_64-darwin13]
```
Hell yes! We got what we expected! Ok, go ahead and commit your changes so you can track your work, and push the branch up if you want to be completely safe.

## Per user manifests

Ok, go ahead and go to that `modules/people` and read that [README.md](https://github.com/jjasghar/our-boxen/tree/master/modules/people) so you get some background.

Now:
```bash
[/opt/boxen/repo/] % cd modules/people/manifests
[/opt/boxen/repo/modules/people/manifests] % git checkout master
[/opt/boxen/repo/modules/people/manifests] % git checkout -b jjasghar_manifest
[/opt/boxen/repo/modules/people/manifests] % vim jjasghar.pp # this needs to be your github account name
```
Here's a template from [Greg](https://github.com/awaxa/our-boxen/blob/master/modules/people/manifests/awaxa.pp), if you want to see what he's done.
```ruby awaxa.pp
class people::awaxa {
  include people::awaxa::applications
  include people::awaxa::dotfiles
  include people::awaxa::gitconfig
  include people::awaxa::preferences
  include people::awaxa::puppetlabs
}
```
As you can see he references other files located in that same directory, though in a sub directory as `awaxa/`. Lets use that `include people::awaxa::applications` initially.
```ruby applications.pp
class people::awaxa::applications {
  include caffeine
  include chrome
  #include dropbox
  #include gpgtools
  include onepassword
  include java
  include rdio
  include tunnelblick::beta
  include vagrant
  include virtualbox
  include vlc
  include vmware_fusion

  package { [
    'htop-osx',
    'tmux'
    ]:
  }

  package { 'GoogleVoiceAndVideoSetup':
    source => 'http://dl.google.com/googletalk/googletalkplugin/GoogleVoiceAndVideoSetup.dmg',
    provider => pkgdmg,
  }
}
```
Ok here's the one I created:
```ruby jjasghar.pp
class people::jjasghar {
  include people::jjasghar::applications
}
```
Nice! Ok, now we need to create:
```bash
[/opt/boxen/repo/modules/people/manifests] % mkdir jjasghar
[/opt/boxen/repo/modules/people/manifests] % vim jjasghar/applications.pp
```

This is where we add our apps!
```ruby applications.pp
class people::jjasghar::applications {
  include iterm2::stable
  include dropbox
  include mysql
  include onepassword
  include virtualbox
  include vagrant
  include chrome
}
```

Ok, now you need to add these to the `Puppetfile` located at the root of the boxen repo
```ruby
[-- snip --]

# Optional/custom modules. There are tons available at
# https://github.com/boxen.
github "iterm2", "1.0.0", :repo => "boxen/puppet-iterm2"
github "chrome"
github "dropbox"
github "mysql"
github "onepassword"
github "virtualbox"
github "vagrant"
```

After you update the `Puppetfile` go ahead and run `boxen` again:
```bash
[/opt/boxen/repo/modules/people/manifests] % cd /opt/boxen/repo
[/opt/boxen/repo] % boxen
```

Grats! You got everything installed. This is just the start, but you can see the beauty of this now. Or at least I could.

