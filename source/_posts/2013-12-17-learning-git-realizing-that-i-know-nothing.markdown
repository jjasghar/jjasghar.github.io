---
layout: post
title: "Learning git realizing that I know nothing"
date: 2013-12-17 13:56
comments: true
categories: sysadmin git
---

So as with all modern day "DevOps" guys I use a SCM [[Source Control Management](http://en.wikipedia.org/wiki/Source_Control_Management)] system on a daily if not hourly basis.  I've been attempting to learn it more and more, and I came across this video from the [/r/git](http://reddit.com/r/git) subreddit and I don't think I've ever seen a better explanation. It's called _Git for Ages 4 and Up_.

{% youtube 1ffBJ4sVUb4 %}

I strongly suggest taking the 99ish mins to watch it, it's a great tutorial, with great visual examples.


I've also stolen some aliases that he suggests!
```bash
alias glp="git log --graph --pretty --all"
alias gld="git log --graph --decorate --all"
```

Some useful notes that this video helped clear up for me:

Something else that finally cleared up to me `origin/master` means a label, while `origin master` means the remote origin with the label of master on it's remote side. This works with `origin/bugfix` and `origin bugfix` also, it's just unfortunate that the master is used twice is a very similar context.

`git merge master` merges master into the current branch you have checked out. `git merge origin/master` merges the upstream changes of master into your current branch. `git merge master bugfix` merges the master branch into bugfix explicitly.

`git push --set-upstream (or -u) origin bugfix` sets the upstream branch on origin to bugfix, if from your local repo.

`refs/head/blah` is a fancy way of saying `blah` is the branch.

`git push origin --tags` to push your tags on the local repo to the origin

A detached HEAD isn't a bad thing; it just means there's no branch associated with it. It's good to check things out but you can always `git checkout -b <branch_name>` if needed.

`git rebase -i HEAD^^` go back to rebase 2 commits; rebase goes bottom up, to squash up to the top one.

`rebase` creates a _new_ line of history, not REWRITES history. 

Once you've pushed, never rebase; it's not worth it. :(

