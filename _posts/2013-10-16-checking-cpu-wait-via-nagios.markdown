---
layout: post
title: "Checking CPU WAIT via nagios"
date: 2013-10-16 11:11
comments: true
categories: linux sysadmin nagios
---

So our machines swap....swap alot. There are the  typical nagios system checks, you can have the `check_swap` call, but at my company the < 20 % hits so often we needed to figure out a way to get warned about crippling proformance. We discovered that when a machine has CPU WAIT spikes, the proformance for the machine tanks, so I tried to find a check for specificly monitor the CPU WAIT.  I should mention also that most/all of the boxes we run are VMs, and waiting on IO for swap makes a sad sysadmin.

I took a stab at creating a check myself and the following is a simple nagios check that you can run via NRPE.  The `PERCENTAGE` line is changeable, I'd make sure your alarm reflects your error cases. We choose 10 % because that's when we started seeing real prefomance issues, but as all sysadmins know any CPU WAIT is bad in general.

```bash
#!/bin/bash
stateid=
CPUWAIT=`top -b -n 1| grep 'wa,' | awk -F ',' {'print $5'} | cut -d . -f 1 | tr -d ' ' `
PERCENTAGE=10

if [ ${CPUWAIT} -le ${PERCENTAGE} ]
 then
   echo Your CPU WAIT is under ${PERCENTAGE}%
   stateid=0
elif [ ${CPUWAIT} -ge ${PERCENTAGE} ]
 then
   echo Your CPU WAIT is at ${CPUWAIT}%, your machine aint happy
   stateid=2
 fi
exit $stateid
```

You are probably wondering "There are alot of great systems checks on [Nagios Exchange](http://exchange.nagios.org/)," but the site is so hard to use and useally much more than I need, like in this case.
