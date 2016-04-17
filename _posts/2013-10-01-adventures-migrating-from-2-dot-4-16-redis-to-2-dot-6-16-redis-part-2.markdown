---
layout: post
title: "Adventures migrating from 2.4.16 redis to 2.6.16 redis (Part 2)"
date: 2013-10-01 18:11
comments: true
categories: linux sysadmin redis monitoring nagios
---

Monitoring
=========

Everyone loves monitoring. Ok, that's a lie, only [DevOps|Sysadmins] love monitoring. Ok, that's probably a lie too, hell I love monitoring, and I love nagios.  I've followed used nagios in some shape or another since nagios-1.0b6. My first real paid job as an "IT guy" was setting up nagios for a company.  I've actually gotten in the habit of walking into a new company using [serverspec](http://jjasghar.github.io/blog/2013/07/12/serverspec-the-new-best-way-to-learn-and-audit-your-infrastructure/), to learn what each machine does, then implementing nagios behind the serverspec scripts for a proactive monitoring on error conditions. This might seem redundant, but I don't think so. serverspec is great for spot checking, while nagios gives the best "state of the union" snapshot. Granted a lot of the time creating these checks are reactive, but sometimes like here, I wrote some proactive monitoring because I know there are certain error conditions on the slaves I've spun up.

I use nrpe to monitor my machines, and I set up my `nrpe.cfg` to something like the following. I use chef, so the `<%= node['hostname'] %>` is from [ohai](https://github.com/opscode/ohai).

```ini
command[check_redisserver]=/usr/lib/nagios/plugins/check_procs -c 1:1 -C redis-server
command[check_redis_replication]=/usr/lib/nagios/plugins/check_redis_replication -h <%= node['hostname'] %> -w 20 -c 45
command[check_redis_rdb_age]=/usr/lib/nagios/plugins/check_redis_rdb_age <%= node['hostname'] %>
```

The first line is pretty self explanatory.  Obviously I'd like to check that redis is running, and that's what it does.

The second line is where I started creating the new monitoring.  I have all my monitors right now at a 3 strike out rule, I have played with the idea of tweaking it, but it seems 3 strikes then an email seems to cut down on the noise I get.

```ruby
#!/usr/bin/env ruby
require 'optparse'
options  = {}
required = [:warning, :critical, :host]

parser   = OptionParser.new do |opts|
    opts.banner = "Usage: check_redis_replication [options]"
      opts.on("-h", "--host redishost", "The hostname of the redis slave") do |h|
        options[:host] = h
      end
      opts.on("-w", "--warning percentage", "Warning threshold") do |w|
        options[:warning] = w
      end
      opts.on("-c", "--critical critical", "Critical threshold") do |c|
        options[:critical] = c
     end
end
parser.parse!
abort parser.to_s if !required.all? { |k| options.has_key?(k) }

master_last_io_seconds_ago = `/opt/redis/bin/redis-cli info | grep master_last_io_seconds_ago`.split(':').last.to_i rescue -1

status = "A-Ok!"
if master_last_io_seconds_ago < 0 || master_last_io_seconds_ago >= options[:critical].to_i
  status = :critical
elsif master_last_io_seconds_ago >= options[:warning].to_i
  status = :warning
end

status_detail = master_last_io_seconds_ago == -1 ? 'ERROR' : "#{ master_last_io_seconds_ago.to_s }s"
puts "#{status.to_s.upcase} - replication lag: #{status_detail}"

if status == :critical
  exit(2)
elsif status == :warning
  exit(1)
end
```

I stole this monitor from [here](http://blog.winfieldpeterson.com/2012/12/13/monitoring-redis-replication-in-nagios/) but as you can tell the `redis-cli` is edited and the default status too.  [Miah](https://twitter.com/miah_)'s [cookbook](https://github.com/miah/chef-redis) is great, works like a charm, but it puts the `redis-cli` in `/opt/redis/bin/`.

As a side note: I even put in a [PR](https://github.com/miah/chef-redis/pull/55) to update to 2.6.16, because of this project.  I ran all the [test-kitchen](https://github.com/opscode/test-kitchen) tests and it passed like a charm.  I actually learned about [bats](https://github.com/sstephenson/bats) in the process, and honestly I'm pretty damn impressed with something so straight forward.  Because serverspec fulfills my requirements at the moment, I don't have to use it; but I like the idea of having it in my back pocket.  The replication between master and slave seems extremely important, so this monitor has been a god sent.  I never have seen it spike over 45 seconds, but then the next check pop back to <3 seconds.

The second monitor I wrote was this:
```bash
#!/bin/bash

stateid=
LAST_SYNC_HUMAN=`/opt/redis/bin/redis-cli -h $1 info | grep rdb_last_save_time | awk -F : {'print $2'} | xargs -I {} date -d @{}`
LAST_SYNC=`/opt/redis/bin/redis-cli -h $1 info | grep rdb_last_save_time | awk -F : {'print $2'} | tr -d '\r'`
LAST_15MIN=`date -d '15 minutes ago' +%s`
LAST_20MIN=`date -d '20 minutes ago' +%s`

if [ "${LAST_15MIN}" -lt "${LAST_SYNC}" ];
  then
  echo Sync has happened in the last 15 mins or less at ${LAST_SYNC_HUMAN}
  stateid=0
elif [ "${LAST_15MIN}" -ge "${LAST_SYNC}" ] && [ "${LAST_20MIN}" -lt "${LAST_SYNC}" ];
  then
  echo Sync happened a little longer than we would like, the last at ${LAST_SYNC_HUMAN}
  stateid=1
elif [ "${LAST_20MIN}" -ge "${LAST_SYNC}" ];
  then
  echo Sync took over 20 mins, something might be wrong, the last sync happened at ${LAST_SYNC_HUMAN}
  stateid=2
fi
exit $stateid
```
This monitor I took from my typical [bash nagios template](https://github.com/jjasghar/scripts/blob/master/nagios_check_generic_template.sh) and added the error states.  It seems like the .rdb file should always be in "close" proximity to the last write of to slave, and it would be nice to be alarmed about it if it went out of wack.  From what I've seen it's never more than 1 or 2 minutes behind; so this worked out.  Writing a monitor for something and never seeing it fire till there is a real problem, that's a great place to be in.

Non-nagios monitoring
---------------------

I started looking around for some type of dashboard to get some visual data about the redis instance.  It turns out there is a TON of these bastards, from a paid product like [Scout](https://scoutapp.com/plugin_urls/271-redis-monitoring) to [redis-stat](https://github.com/junegunn/redis-stat).  I picked redis-stat because well, it's un-intrusive and fast.  I have it polling against my master slave on two instances of the gem running, and it's really nice to have a easily digestible view of the health of it.  I have noticed that it has doubled up the info because of the syncing, so be warned, you probably should run it via just the master and then maybe put all the slaves on one? (I'm still trying to figure this one out.)

If you do read this and have good suggestions please through them my [way](http://jjasghar.github.io/about/), the `INFO` command gives you a lot of data, and redis-stat doesn't store anything so treading over time isn't an option. I was thinking about putting some of this data in ganglia, (another favorite of mine) but to have something just redis focused seems like the correct move here.

---------------------
So, so far these is the monitors that I'm using for my new master slave set up.  My next challenge is creating the VIP in front of the master slave, and have a "hot fail over." This is going to require some pretty interesting engineering; it's fun, but man the more I think about it the more it hurts my brain.  So I haven't actually talked about the migration yet either.  From what I've understood it seems that 2.4.16 is actually _inside_ 2.6.16, so if I get this VIP in front of this master slave then fail over to the slave turning off the read-only, this should be seamless. I need to do some testing; but that seems like the best course of action. For just the migration though, the "hot fail over" isn't required, but I want it to be in the fore-front of the design because I'd rather be additive than have to rip out everything after we get up dated.
