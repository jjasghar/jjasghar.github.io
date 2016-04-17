---
layout: post
title: "How to fix https://chef defaulting running chef client on open source chef server"
date: 2013-10-05 16:59
comments: true
categories: linux chef sysadmin
---


I went to #chef on freenode and maek helped me out.  Here’s a run down of how to fix it. Alas the gists are gone; sorry. :(

If you play with open source chef you might run into this.

```
18:42 j^2 so i’m having trouble with chef-client
18:42 j^2 why does it default to https://chef/blahblah?
18:43 j^2 example:
18:43 j^2 https://gist.github.com/jjasghar/5873e421a7f8365194e3
18:44 j^2 any advice?
18:44 j^2 i guess i could add it to /etc/hosts, but i’d like it to use the chef_server_url is that the point of it?
18:45 maek j^2:  I just got hit with this also
18:45 maek i think becuase chef 11 is now fronted with nginx
18:45 maek its doing a rewrite
18:45 maek for the name configured in nginx
18:45 maek in this case its hostname
18:45 maek but i assume that box cant resolve chef
18:46 j^2 lame
18:46 maek j^2:  I had to end up adding a hosts entry
18:46 maek while I wait for dns
18:46 maek I think you could do
18:46 j^2 yeah that seems like the only option but it’s stuff dumb :(
18:47 maek you could reconfigure chef
18:47 maek to use its ip
18:47 maek instead of its hostname
18:47 j^2 tried the ip in chef_server_url didnt work either; wait you mean nginx?
18:48 @ssd7 maek: Re your question above. I believe you should be able to edit bookshelf[‘vip’] in your config and the run a reconfigure
18:48 maek i dont see it htought
18:48 j^2 oh!
18:48 maek default[‘chef_server’][‘bookshelf’][‘vip’] = node[‘fqdn’]
18:48 j^2 nice looking
18:48 maek yeah
18:48 maek there it is
```

Or you can do this also, I believe this is how I fixed it myself.:

```
18:48 maek so you can do
18:49 maek in /etc/chef-server/chef-server.rb
18:49 maek bookshelf[‘vip’] = ‘192.168.1.1’
18:50 j^2 yeah i dont thave that file. :(
18:50 maek and run chef-server-ctl reconfigure
18:50 j^2 ah cool
18:50 maek its for overrides
18:50 maek just make it
```
