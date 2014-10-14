---
layout: post
title: "git rebase and git detached head cheat sheet"
date: 2014-10-13 15:19:30 -0500
comments: true
categories: git sysadmin
---

## git rebasing or fixing an auto merge conflict

In my adventures with openstack and gerrit reviews I sometimes see something to the effect of: *Patch in Merge Conflict*

This makes me very sad. I hate these things, and to quote Mark Vanderwiel, it's basically like someone snuck in a change before you.

We had a great conversation about it [here](http://youtu.be/wsWj_NQJKI0?t=3m50s) for about 3 mins, but if you want a tl;dw (watch) it's this:
```bash
$ git remote update
$ git checkout master
$ git pull
$ git checkout $BRANCH_THAT_YOUR_FIXING
$ git rebase master
$ # Fix any conflicts that should pop up
$ git add $ANY_CONFLICTS
$ git rebase --continue
$ git commit --amend
$ git review -R
```
It looks like a lot, but all in all not bad at all. The rebasing might look scary but walk through each conflict, edit it and add it back you should be fine.


## Detached head what the hell does that mean?

If you continue watching that video we start to talk about detached heads. If you have played in the openstack/gerrit world you'll notice a copy paste off a review by download, something like:
```bash
$ git fetch https://review.openstack.org/stackforge/cookbook-openstack-client refs/changes/65/126365/9 && git checkout FETCH_HEAD
```
Needless to say this is confusing as all hell. Not to mention when you finish it is says something like:

```bash
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 789cb70... Initial Commit of the cookbook-openstack-client
```

Seriously what the hell does this mean? Luckily it's actually not a bad thing at all. If you watch from [here](http://youtu.be/wsWj_NQJKI0?t=8m10s) for about 2 minutes you'll see us talk about it. The short of it is, you've pulled down the patch, but it's not attached (detached) to your local master/other branch. When you do the `git checkout -b branch_name` off of master/other branch, it connects it up so it's no longer detached.

After saying it out loud, or typing it out as i just did, it makes sense, but surprisingly no one ever told me such a simple concept. I hope this helps someone down the line.
