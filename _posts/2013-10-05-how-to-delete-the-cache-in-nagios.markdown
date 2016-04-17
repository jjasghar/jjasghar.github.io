---
layout: post
title: "How to delete the cache in nagios"
date: 2013-10-05 16:06
comments: true
categories: nagios sysadmin linux
---

If you use the [nagios cookbook](https://github.com/opscode-cookbooks/nagios), and you have multiple disposable machines checking in and out of your chef-server, you might notice that running chef-client to pull new machines into nagios doesn't always work.  For a time my company had anything ranging from 4 to 8 machines spinning up and down, so I had to basically run chef-client on a 30 min interval to make sure I got them all. I discovered that my new machines weren't showing up, so I had to figure out how to "clear the cache" if you will.

Now this this is specific to  the nagios setup I have, but the `objects.cache` is what controls this.

```bash
root@boe:/tmp# rm /var/cache/nagios3/objects.cache
root@boe:/tmp# service nagios restart
Running configuration check…done.
Stopping nagios: .done.
Starting nagios: done.
root@boe:/tmp# cd /var/cache/nagios3
root@boe:/var/cache/nagios3# ls
objects.cache  status.dat
```

