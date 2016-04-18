---
layout: post
title: "Using Carina to mimic IRCcloud"
date: 2015-11-15 18:41:18 -0600
comments: true
categories: sysadmin linux docker
---

Using Open Source software you get used to being on IRC [freenode][freenode] be exact,
more or less 24 hours a day 7 days a week. People talk/collaborate/work together
there and it's important to stay informed.

I've used [IRCcloud][irccloud] for this lifestyle for the past year, love it, but having
to do everything through a web browser has been frustrating. I've used [Nimbus][nimbus]
and even [fluid][fluidapp] to make a fake desktop application. None of these options seemed to
ever feel right, I still felt like I was looking at a web browser. Before
IRCcloud I used [weechat][weechat] and before that [irssi][irssi] I really miss the
chance to use these amazing IRC clients. Though I think the main reason why
I'd stuck with IRCcloud for so long was due to the push notifications to my phone.
No matter what the ability to get ping'd when a community member needed some help
was invaluable, but I still missed my clients.

I chewed on my options for a while, then talking to [John Vrbanac][John_Vrbanac]
on a train from NRT to the [Tokyo OpenStack summit][tokyo-2015] he mentioned he uses
a Docker container as his IRC bouncer. This idea was brilliant, but there were
quite a few options for IRC bouncers out there. I came to the conclusion,
which was to spin up a Docker container with [znc][znc] inside of it. As one of my
main points for using  IRCcloud was push notifications so I had to have a way
to send pushes.

In a previous $JOB I had purchased [Pushover][pushover] and the
application was still on my phone, so I choose it as my main notification layer.
Luckily there is a [znc-push][znc-push] plugin, and it has [support for Pushover][pushover-md].
Now all I need to do was create the container and find a place to run it.

I extended the code from [docker-znc][docker-znc] after discovering that, as I
was trying to build znc, [Jim Myhrberg][Jim_Myhrberg] had already done it.
He hadn't created an easy way to get a plugin installed _inside_ the container
so that's where I could extend it. I wrote up [docker-znc-pushover][docker-znc-pushover]
which build in the `push.so` which was all this container was missing.

I tested the container, deployed it to my development docker host, worked as I
suspected. I even pushed [the container to hub.docker.com][hub-container]
which was good luck, because where I decided to run it required a place to pull
from.

I spent some time looking for place to run this container from, I thought my
development docker host, but it didn't seem right. I wanted a place that wouldn't
rely on my 2 gig laptop having power. So I decided to look into [Carina][carina]
which was announced at the Tokyo Summit. I went through [the setup][carina-setup]
and had a working Docker Swarm cluster. Now I needed to get my container on this cluster.
I did some head desk movements, some debugging, but eventually it came out to
these steps. These are the actual commands I ran, and now, I have a working znc
bouncer in Carina now.

```bash
~$ mkdir ~/carina
~$ carina create znc-pushover --wait --nodes=1
~$ carina credentials --path=/Users/jasghar/carina/znc-pushover znc-pushover
~$ source ~/carina/znc-pushover/docker.env
~$ docker info # to confirm everything is talking correctly
~$ docker create -v /zncdata --name zncdata training/postgres /bin/true # creating a Volume Container
~$ docker run -d --volumes-from zncdata --name znc-pushover -p 36667:6667 jjasghar/znc-pushover
```

The `/zncdata` container was the most challenging part for me, but as soon
as I got my head wrapped around the concept of a Volume Container, the rest
was easy.

I went to the IP address that came back with `docker info` on port `36667` and
I saw the glorious znc login. I set everything up from the web browser, and spun
up irssi. I connected to the `IP:36667` and saw the glorious znc welcome MOTD.

I set up push notifications via these commands:

```
/msg *push set service pushover
/msg *push set username your-username-token
/msg *push set secret your-secret-token
/msg *push set target your-device-name-if-you-have-multiple
/msg *push send test
/msg *status connect
```

I got my test message on my phone and smiled.

[freenode]: http://freenode.net
[irccloud]: http://irccloud.com
[nimbus]: https://github.com/jnordberg/irccloudapp
[fluidapp]: http://fluidapp.com
[weechat]: http://weechat.org/
[irssi]: http://irssi.org/
[John_Vrbanac]: https://github.com/jmvrbanac
[tokyo-2015]: https://www.openstack.org/summit/tokyo-2015/
[znc]: http://wiki.znc.in/ZNC
[pushover]: http://pushover.net/
[znc-push]: https://github.com/jreese/znc-push
[pushover-md]: https://github.com/jreese/znc-push/blob/master/doc/pushover.md
[docker-znc]: https://github.com/jimeh/docker-znc
[Jim_Myhrberg]: https://github.com/jimeh
[docker-znc-pushover]: https://github.com/jjasghar/docker-znc-pushover
[hub-container]: https://hub.docker.com/r/jjasghar/znc-pushover/
[carina]: https://getcarina.com
[carina-setup]: https://getcarina.com/docs/getting-started/getting-started-carina-cli/
