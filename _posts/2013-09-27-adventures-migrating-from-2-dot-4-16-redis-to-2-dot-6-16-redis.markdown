---
layout: post
title: "Adventures migrating from 2.4.16 redis to 2.6.16 redis (part 1)"
date: 2013-09-30 20:15
comments: true
categories: linux sysadmin redis
---

Like most sysadmins out there, I read the blogs and posts about new technologies that are relevant to my product(s).  There has been a lot of buzz about NoSQL and the key value store type db's out there, yet I have never really focused directly on it.  I have paid passing attention, until I realized that we use one heavily at my company.

[Redis](http://redis.io) is a key value store that runs completely in memory. It's fast, REALLY fast, and after spending some time with a few tutorials like [The Little Redis Book](http://openmymind.net/2012/1/23/The-Little-Redis-Book/) by [Karl Seguin](https://twitter.com/karlseguin), it makes a ton of sense, and an extremely neat backend piece of software.

I started to learn more about it, and I started talking to a coworker and he said that he really wanted to upgrade from 2.4 redis to 2.6 redis. I took this as a challenge, and ran with it.

The first part of the migration is creating a slave.  We had .aof and .rdb snapshotting, but we didn't have a slave. This post is be going to be chronicling my adventures getting from 2.4.16 to 2.6.16.

Another coworker of mine, [Lance Woodson](https://github.com/lwoodson), created a script called "[bloater](https://gist.github.com/jjasghar/423d040444a4f4cbea1d)" to start putting trash keys in the redis db.  As you can see it's pretty straight forward, I do strongly suggest using `bloater.clear!` if you are playing with it.  I leveraged this to simulate creating a slave already in production, the theory being I'll have data inside the redis db, things talking to db, and the sync happening.  I understand that a lot of people have tested this situation, but I couldn't seem to find any direct examples.  Hopefully this will be useful to someone.

The first step after getting my 2.4.16 redis machine running, I ran `bloater.bloat! 2295000000` to get an .rdb file that was ~2 gigs.  I configured the snapshotting at the default values so it created it without a hitch.  I copied it off called it something like `2gig-dump.rdb` or something.  Then I ran `bloater.bloat! 4295000000` to create the 4gig dump file.  I copied that off next and then I had my two dumpfiles that I could play with.  You are probably wondering why 2 and 4 gigs, and also "Dang, that's a huge redis db." Yes, yes it is, I focused on the "oh crap, our db is too big" situation, so if redis handles that, then I know it'll deal with the 100 meg db just fine.

Secondly I did my `FLUSHALL` command, then shutdown my redis instance then copied the 2 gig db, in the `/var/lib/redis/` location.  I spun up redis, and ran the `INFO` command, like all redis admins learn to love to do.  I saw that it was 2 gigs, and yay, now I had my foundation to start my testing.

I went through quite a few different iterations, but I'll just explain the basic trouble shooting that I did, and maybe you can leverage off it.

So with my redis 2.4 instance with a 2 gig db, and my 2.6 blank db instance it was time to start the fun.  I ran the following commands in a tmux session on my main workstation.  I liked the idea of a 3rd party machine actually looking at all this stuff instead of running directly on one of the machines.
``` bash
watch "redis-cli -h redis-slave info | grep master_sync_left_bytes "
redis-cli -h redis-slave keys "*" | wc -l
redis-cli -h redis-master keys "*" | wc -l
redis-cli -h redis-master info | grep used_memory
redis-cli -h redis-slave info | grep used_memory
redis-cli -h redis-master --latency
```

Most, if not all are pretty self explanatory.  The most interesting one of them all is probably the last one, the `--latency`.  It used it as a way to emulate something poking my redis instance during the inital clone. The thing we, [where|are|still] worried about is customer impact, and as long as the number didn't sky rocket then stay high it was an acceptable risk.

After I got the tmux session all up and happy, i  ran these two commands to start the `slaveof`:
```bash
# on the master
redis-master:6379> slaveof no one
# on the slave
redis-slave:6379> slaveof redis-master 6379
```

It worked like a charm. The `watch "redis-cli -h redis-slave info | grep master_sync_left_bytes "` started showing me the amount of data left for the transfer, and before I knew it I had a working master slave configuration.  This was a lot of cruft to basically say `slaveof redis-master 6379` but hell it's sometimes nice to see the thought process around it.  Oh!, the latency, did spike to 800 from .05, but I discovered that was in milliseconds and only ONE sample.  So yeah one call to the master with a spike of something still sub-second, was still in the risk params.
Now I have a working master slave configuration.  Next (part 2) I'll post about my monitoring of this, it's not _that_ exciting, but it is useful.
