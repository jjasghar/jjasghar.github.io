---
layout: post
title: "Getting VASSAL working on Fedora 30+"
date: 2020-01-10 15:55:13
categories: linux fedora boardgaming
---

If you don't know, [VASSAL](http://www.vassalengine.org/) is an Open Source
Java based board game engine. I recently converted to Fedora, and it seems can't
just "Get Vassal" and run it.

If you pull the code down and run `./VASSAL.sh` and nothing happens, run this:

```bash
$ rpm -qa | grep openjdk
java-1.8.0-openjdk-headless-1.8.0.232.b09-0.fc31.x86_64
```

As you can see the default installation doesn't have the libraries like mentioned
[here](http://www.vassalengine.org/forum/viewtopic.php?f=3&t=10120). Go ahead
and run the following and then you should be able to run `./VASSAL.sh`, and
see the application start.

```bash
sudo dnf install java-1.8.0-openjdk
```
