---
layout: post
title: "hub workflow"
date: 2013-10-31 11:01
comments: true
categories: git sysadmin
---

I recently discovered [hub](https://github.com/github/hub) and man this is really really cool. I'm starting to fork out projects and making PRs, but it seems that when I attempt to use `hub pull-request` I get this error:

```bash
~/repo/learning_git/fakedir1% hub pull-request
Error creating pull request: Unprocessable Entity (HTTP 422)
Missing field: "head_sha"
Missing field: "base_sha"
No commits between jjasghar:master and jjasghar:blah
```

WTF man? What the hell does this mean? It seems I wasn't the only one that had this [problem](https://github.com/github/hub/issues/189).  It seems that the work flow (at least in my case) was the problem. Here's a quick fix to be able to open up a PR off a forked branch:

```bash
~/repo/learning_git/fakedir1% git add some_file
~/repo/learning_git/fakedir1% git commit -m "blah"
~/repo/learning_git/fakedir1% git push -u origin blah
Counting objects: 7, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 351 bytes | 0 bytes/s, done.
Total 4 (delta 2), reused 0 (delta 0)
To git@github.com:jjasghar/learning_git.git
   5224edf..88ad1af  blah -> blah
Branch blah set up to track remote branch blah from origin.
~/repo/learning_git/fakedir1% hub pull-request
https://github.com/jjasghar/learning_git/pull/1
```

Pretty slick eh?

Bonus round you can attach your PR to an issue!

```bash
~/repo/learning_git/fakedir1% hub pull-request -i 3
```
