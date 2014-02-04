---
layout: post
title: "ResourceNotFound and splunk> fun...gotta love that error"
date: 2014-02-03 16:18:51 -0600
comments: true
categories: sysadmin splunk 
---

My company uses [splunk](http://splunk.com) yes yes, I know, I should be using [kibana](https://github.com/rashidkpc/Kibana) and [logstash](http://logstash.net/) but I got the budget for splunk> so I'm using splunk.

I attempted to install an updated license the other day and got an interesting error after restarting splunk and attempting to log in:
```
500 Internal Server Error

Return to Splunk home page

ResourceNotFound: [HTTP 404] https://127.0.0.1:8089/services/data/user-prefs; [{'type': 'ERROR', 'text': 'Application does not exist: user-prefs', 'code': None}]

This page was linked to from http://<mycompanyname>:8000/en-US/account/login?return_to=%2Fen-US%2F.



You are using <mycompanyname>:8000, which is connected to splunkd @182037 at https://127.0.0.1:8089 on Mon Feb 3 16:17:37 2014.
```

Wow, what the hell does that mean?

So after some investigation it seems that I had my `/opt/splunk/bin/splunk` symlinked to `/opt/splunkforwarder/bin/splunk`. After some finagling with splunk support it seems it was a complete
oversight.

Having both the `splunkforwarder` and `splunk` on the same box can be dangerous, if you do you'll have to edit the ports that they use. Keep that in mind if you decide to go down this path.
