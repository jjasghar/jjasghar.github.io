---
layout: post
title: "Nagios-status-rackspace"
date: 2013-07-09 15:05
comments: true
categories: rackspace nagios monitoring
---

I posted a [this](http://redd.it/1gefnf) on [/r/sysadmin/](http://www.reddit.com/r/sysadmin) back on 6/15/13.  I wanted to put some context around why I created the nagios checks and why the [RSS](https://status.rackspace.com/index/rss) after all that discussion still didn't fulfill my requirements.

The biggest problem I have with the RSS feed is that it is everything inside the rackspace status page.  As I said to [/u/rackerkev](http://www.reddit.com/user/rackerkev); as a customer in ORD I don't want to be paged or alerted when something in LON happens.  This seems be the default for the RSS feed, a water hose of what that status page says.

After our account manager suggested us to check http://status.rackspace.com a developer on my team asked if there was an automated way to check this.  I ran a quick `curl http://status.rackspace.com` and discovered it's flat html.  Boom, I had a way to get the data, I just needed a way to parse it.

I had a default nagios check template and all I had to do was put that `curl` in to parse it out.

```bash
#!/bin/bash

stateid=
SOMECMD=`echo "something"
SOMETHING=something

if [ "${SOMECMD}" == ${SOMETHING} ];
 then
   echo ${SOMETHING} is something
   stateid=0
#elif [  "${SOMECMD}" == ${SOMETHING} ];
#  then
#  echo This is a Warning
#  stateid=1
elif [ "${SOMECMD}" != ${SOMETHING} ];
 then
   echo Something is broken!
   stateid=2
 fi
exit $stateid
```

I don't think this actually works, but hell you get the point.

So with some patience using `sed` and `grep` I was able to start parsing out the data for my back ends specifically.  I ended up parsing out each section so you could have London Next-Gen and Rackconnect General.  I had a couple posts about "You should write it as one script and then add flags or variables at the end." I disagree, I wrote this so it could be dropped in nagios, straight forward install with little to no overhead.  As long as you have `curl` installed and write the command stanza, like my suggestion in the [README.md](https://github.com/jjasghar/nagios-status-rackspace/blob/master/README.md) you should be off to the races.

Now something interesting I learned posting these checks.  The quote that says "rackspace only updates the status page if enough customers complain about some problem some place" is true.  There is no automated update to the status page, it requires human intervention to update.  When I first heard of the status page, I like most though it was programmatic way to see into rackspace infrastructure.  This makes me a sad panda; and as I have learned the best way to get anything done with rackspace is to call yell and yell more.
