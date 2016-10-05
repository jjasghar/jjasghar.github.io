---
layout: post
title: "Signing commits in git"
date: 2016-10-04 14:22:13
categories: git sysadmin chef
---

With Chef's new policy to use the [DCO signing process][dcosigning], I found
myself googling how to set it up a couple times. Here are my notes on how to
set up `git` so you are now in compliance.

I should make it clear that the difference between `signoff` and `gpg-sign`
are significant but subtle. `signoff`  doesn't require the GPG key and is just
the line: "Signed-off by" which is covered in the following quote.

Our DCO process only requires:

> Moving forward all commits to Chef-maintained open source software projects must include a "Signed-off-by" line indicating the name and email address of the contributor signing off on the change.

For a [detailed description][detailed] or you can just do the following:

```
$ git commit -s
$ # Or
$ git commit --signoff
```
If you already have the commit, use `git commit --amend` or `git rebase -i` to
edit your commit message, and add the above signoff line.

## .gitconfig

If you would like something more permanent, you can add to your `.gitconfig`
something like the following:

```
[alias]
  amend = commit -s --amend
  cm = commit -s -m
  commit = commit -s
```

Or you can do something like [Dave Parfitt][dave] suggests; which is supprisingly
easy. You can add this to your ~/.gitconfig: (be sure to change your name and email address)

```
[commit]
    template = ~/.gitmessage
```

followed by:

```bash
~$ cat ~/.gitmessage

Signed-off-by: Your Name <email@addre.ss>
```

## magit

If you are attempting to do it in emacs with [magit][magit] you need to
the following:

`c - s RET` and you'll notice the `(--signoff)` switch become highlighted.


[dcosigning]: https://discourse.chef.io/t/a-developer-certificate-of-origin-dco-is-now-required-with-code-contributions/9579
[detailed]: http://stackoverflow.com/questions/13457203/how-to-add-the-signed-off-by-field-in-the-git-patch
[magit]: https://magit.vc/
[gpg]: https://harryrschwartz.com/2014/11/01/automatically-signing-your-git-commits.html
[dave]: https://twitter.com/metadave
