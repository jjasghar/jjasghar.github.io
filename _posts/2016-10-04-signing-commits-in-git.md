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
are significant but subtle.

Our DCO process only requires:

> Moving forward all commits to Chef-maintained open source software projects must include a "Signed-off-by" line indicating the name and email address of the contributor signing off on the change.

Which doesn't require the GPG key. The following is first the `signoff` process
then if you are curious on how to set up GPG signing, it follows.

## Sign off

For a [detailed description][detailed] or you can just do the following:

```
$ git commit -s
$ # Or
$ git commit --signoff
```

If you already have the commit, use `git commit --amend` or `git rebase -i` to
edit your commit message, and add the above signoff line.

If you would like something more perminant, you can add to your `.gitconfig`
something like the following:

```
[alias]
  amend = commit -s --amend
  cm = commit -s -m
  commit = commit -s
```

If you are attempting to do it in emacs with [magit][magit] you need to
the following:

`c - s RET` and you'll notice the `(--signoff)` switch become highlighted.

## GPG signing

Here is a good tutorial on the [setup][gpg] for GPG keys and signing.

If you want a more to leverage the GPG "signoff" the following is a
way to set it forever:

```bash
$ git config commit.gpgsign true
```

or something like the following in `.gitconfig`:

```
[alias]
  amend = commit -S --amend
  cm = commit -S -m
  commit = commit -S
```

If you're a [magit][magit] user like me, you can do the following for `gpg-sign`:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/jjasghar">@jjasghar</a> Well of course ;-) To sign one commit use &quot;c = S RET c&quot; to sign all commits set the Git variable &quot;commit.gpgSign&quot;.</p>&mdash; Jonas Bernoulli (@magit_emacs) <a href="https://twitter.com/magit_emacs/status/766785033277353984">August 19, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


[dcosigning]: https://discourse.chef.io/t/a-developer-certificate-of-origin-dco-is-now-required-with-code-contributions/9579
[detailed]: http://stackoverflow.com/questions/13457203/how-to-add-the-signed-off-by-field-in-the-git-patch
[magit]: https://magit.vc/
[gpg]: https://harryrschwartz.com/2014/11/01/automatically-signing-your-git-commits.html
