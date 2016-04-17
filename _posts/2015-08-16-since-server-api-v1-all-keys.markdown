---
layout: post
title: "Since Server API v1 all keys"
date: 2015-08-16 15:10:37 -0500
comments: true
categories:  sysadmin chef
---

I was playing around with [Chef Server 12](https://downloads.chef.io/chef-server/) and attempted to do a `knife bootsrap` a machine. My bootstrap failed due to a timeout with a `remote_file` download. I attempted to run a second `knife bootstrap` against the same machine, and I saw the `first-boot.json` start doing it's thing.

About 30 seconds late I saw this error:

```
Server Response:
----------------
Since Server API v1, all keys must be updated via the keys endpoint.
```

Needless to say I got frustrated extremely quickly. I wasn't doing anything due to keys, and now I need to update? :sad_panda: I re-ran the `knife bootstrap` again, and it bombed out with that same error. Double :sad_panda:

I did some searching around, asked a few people around Chef, and ended up at [this blog post](https://www.chef.io/blog/2015/07/14/chef-server-12-1-1-released/).

I did what was suggested in the comments `knife client delete <machine>` and to be safe `knife node delete <machine>` and re-ran `knife bootstrap`. It continued on and bootstrapped the box. I haven't seen the error since.
