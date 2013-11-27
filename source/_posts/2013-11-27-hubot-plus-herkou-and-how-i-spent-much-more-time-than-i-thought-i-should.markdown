---
layout: post
title: "Hubot+Herkou+Campfire and how I spent much more time than I thought I should"
date: 2013-11-27 11:41
comments: true
categories: sysadmin linux
---

So I recently watched [ChatOps](http://www.youtube.com/watch?v=NST3u-GjjFw) and got sold on [Hubot](http://hubot.github.com/).  We had him at my company for a while, but
with a migration we lost him; so I took it upon myself to learn how to use him again.  All in all getting him running inside my company was pretty straight forward, hell even getting
node to work on my local box for local development ([this gist](https://gist.github.com/isaacs/579814) sums it up nicely) wasn't hard at all.  I found myself wanting my own hubot for [memes](https://github.com/github/hubot-scripts/blob/master/src/scripts/meme_captain.coffee) generation, or just to play around with.

So I decided hell, lets try out the [heroku](https://github.com/github/hubot/blob/master/docs/deploying/heroku.md) install. It was pretty straight forward, I got my repo running, got it deployed; but I ran into an error: 
```
ERROR Campfire request error: Error: getaddrinfo ENOTFOUND
```
This is what the doc says....
```
% heroku config:add HUBOT_CAMPFIRE_ACCOUNT=yourcampfireaccount
% heroku config:add HUBOT_CAMPFIRE_TOKEN=yourcampfiretoken
```
I put my `HUBOT_CAMPEFIRE_ACCOUNT` name as my hubot's account name with the token which seems logical...wth man? 
```
HUBOT_CAMPFIRE_ACCOUNT=hubotsemail@myaccountname.com
```
I did some searching, googling, (it seems a few people had this [issue](https://www.google.com/search?q=ERROR+Campfire+request+error%3A+Error%3A+getaddrinfo+ENOTFOUND&oq=ERROR+Campfire+request+error%3A+Error%3A+getaddrinfo+ENOTFOUND&aqs=chrome..69i57j69i59.2689j0j9&sourceid=chrome&espv=210&es_sm=91&ie=UTF-8), but no fixes were posted), more reading, and it something clicked with me, maybe it's the actual consumer account that it connects to, not the hubot account? So I tried this:
```
HUBOT_CAMPFIRE_ACCOUNT=myaccountname.campfirenow.com
```
Damn, nope.
```
ERROR Campfire request error: Error: getaddrinfo ENOTFOUND
```
I figured I'd try with `https://`.....
```
HUBOT_CAMPFIRE_ACCOUNT=https://myaccountname.campfirenow.com
```
Damnit.
```
ERROR Campfire request error: Error: getaddrinfo ENOTFOUND
```
Finally I started searching around actual hubot [issues](https://github.com/github/hubot/issues/329) and came across that guy. The `HUBOT_CAMPFIRE_ACCOUNT` just needed to be the subdomain. Do'h!
```
HUBOT_CAMPFIRE_ACCOUNT=myaccountname
```
Boom, it worked.
