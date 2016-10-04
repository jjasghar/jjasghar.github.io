---
layout: post
title: "Signing commits in git"
date: 2016-10-04 14:22:13
categories: git sysadmin chef
---

With Chef's new policy to use the [DCO signing process][dcosigning], I found
myself googling how to set it up a couple times. Here are my notes on how to
set up `git` so you are now in compliance.

For a [detailed description][detailed] or you can just do the following:

```
$ git commit -s
$ # Or
$ git commit --signoff
```

If you already have the commit, use `git commit --amend` or `git rebase -i` to
edit your commit message, and add the above signoff line.

Or if you want a more perminant option the following is a way to set it forever:

```bash
$ git config commit.gpgsign true
```

If you're a [magit][magit] user like me, you can do the following:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/jjasghar">@jjasghar</a> Well of course ;-) To sign one commit use &quot;c = S RET c&quot; to sign all commits set the Git variable &quot;commit.gpgSign&quot;.</p>&mdash; Jonas Bernoulli (@magit_emacs) <a href="https://twitter.com/magit_emacs/status/766785033277353984">August 19, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


[dcosigning]: https://discourse.chef.io/t/a-developer-certificate-of-origin-dco-is-now-required-with-code-contributions/9579
[detailed]: http://stackoverflow.com/questions/13457203/how-to-add-the-signed-off-by-field-in-the-git-patch
[magit]: https://magit.vc/
