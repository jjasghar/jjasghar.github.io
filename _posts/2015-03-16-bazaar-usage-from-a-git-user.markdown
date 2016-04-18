---
layout: post
title: "bazaar usage from a git user"
date: 2015-03-16 12:16:07 -0500
comments: true
categories: git linux bzr sysadmin
---

I love git. Honestly, I really do, hell I even have a sticker on my laptop.
Unfortunately [Canonical](http://canonical.com) choose something different
called [bazaar](http://bazaar.canonical.com/en/). It's supposed to be easier,
the jury is still out on that for my opinion; but it really is a tad bit different.

I'm putting a collection of commands here, so if you decided you'd like to put in
a "PR" or in [launchpad](https://launchpad.net/)'s term "Propose for merging" merge.

Lets say you've found a typo and you'd like to fix it; something like [this](https://code.launchpad.net/~d-jj/ironic/ironic-conductor/+merge/253085).

These steps assume you have a [launchpad id](https://login.launchpad.net/+login) and account set up correctly, and all you
want to do is push up a change to an issue you've found.

First be sure to login to launchpad:

```bash
~$ bzr launchpad-login <username>
```

Then you clone down the code locally with this, NOTE: ubuntu/ironic can be anything on launchpad.

```bash
~$ bzr branch lp:ubuntu/ironic
```

Then `checkout` the repo into a branch you can work on, NOTE: ironic-conductor will create a directory you will work in

```bash
~$ bzr checkout lp:ubuntu/ironic ironic-conductor
```

Then you need to 'link' the local working directory with your launchpad account, NOTE: this'll push up the initial branch to launchpad.

```bash
~$ cd ironic-conductor
ironic-conductor > bzr branch --stacked --switch lp:ubuntu/ironic lp:~d-jj/ironic/ironic-conductor
```

Make your changes!

```bash
ironic-conductor > # I'm doing awesome changes now, rm -rf my_file and blah blah blah
```

Sanity check your changes:

```bash
ironic-conductor > bzr diff
```

Yep, looks good, lets commit:

```bash
ironic-conductor > bzr commit -m "I did some amazing changes and this is that commit"
```

If everything goes to plan you should see something like:

```bash
Most recent Ubuntu version: 2015.1~b2-0ubuntu1
Packaging branch status: CURRENT
Committing to: bzr+ssh://bazaar.launchpad.net/~d-jj/ironic/ironic-conductor/
modified debian/ironic-conductor.init.in
Committed revision 14.
```

Ok, so now if you go to your launchpad you should see the push, now go to the project and click that
`Propose for merging` link.

Under "Target Branch" place the active branch under development, also add a note to the "Description of Change"
so you have a blurb on what it does. Click "Propose Merge."

I've found that I constantly get it backwards, so be wary of that; you may have to re-propose the merge if the
diffs look wrong. You can do this from the page after the "Propose Merge." If someone figures out how to get it
right the first time please ping me: @jjasghar on twitter, I'd love to see the "correct" way to do this.

Also for that matter, I couldn't figure out a way to do this without using the site, there's gotta be a CLI
only version of this.

Here are some links I looked at I was figuring this out:

1. http://stackoverflow.com/questions/5043104/step-by-step-bazaar-workflow
2. https://oxygene.sk/2009/10/working-with-branches-in-bazaar/
3. http://askubuntu.com/questions/13547/how-to-create-a-personal-branch-in-launchpad
4. http://askubuntu.com/questions/93859/what-is-bazaar-and-how-do-i-use-it
