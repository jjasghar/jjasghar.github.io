---
layout: post
title: "Running habitat in PhotonOS"
date: 2017-04-10 11:50:49
categories: photon habitat chef
---

![](https://camo.githubusercontent.com/4d4cc8eedee941e882dc521eb37dd354c6beca20/687474703a2f2f73746f726167652e676f6f676c65617069732e636f6d2f70726f6a6563742d70686f746f6e2f766d772d6c6f676f2d70686f746f6e2e737667)

![](https://avatars1.githubusercontent.com/u/18171698?v=3&s=200)


So you got [PhotonOS][photon] running in your DC! Awesome. Now you've been playing around with Cloud
Native Apps, and came across [habitat][habitat] and want to give it a shot. Here's my notes
on how I got a habitat `.hart` package, as a docker container, running in a Photon end point.

I'm going to use my [habitat-jenkins][jenkins] as the demo.

First go ahead and clone the repo and build the `.hart` locally in your `hab studio`:

```shell
$ git clone https://github.com/jjasghar/habitat-jenkins
$ cd habitat-jenkins
$ hab studio enter
[1][default:/src:0]# build
[2][default:/src:0]# hab start YOURORIGIN/jenkins-war # verify everything comes up
[3][default:/src:0]# hab pkg export docker YOURORIGIN/jenkins-war # export to a container
[4][default:/src:0]# logout # leave the studio
$ docker run -p 8080:8080 -it YOURORIGIN/jenkins-war:latest # run container
```

Now lets get the container over to the PhotonOS box:

```shell
$ docker save jjasghar/jenkins-war:latest > jenkins.tar
$ scp jenkins.tar root@PHOTONHOST://root/ # this can be where ever, but demowise...
$ ssh root@PHOTONHOST
root@PHOTONHOST [ ~ ]# docker load < jenkins.tar
de4244ee79bb: Loading layer [==================================================>] 431.6 MB/431.6 MB
Loaded image: YOURORIGIN/jenkins-war:latest
root@PHOTONHOST [ ~ ]# logout
```

Ok, lets verify everything is what we expect.

```shell
$ export DOCKER_HOST=tcp://PHOTONHOST:2375
$ docker images --all
REPOSITORY                                  TAG                 IMAGE ID            CREATED                  SIZE
YOURORIGIN/jeknins-war                        latest              1a29c0686c54        Less than a second ago   427 MB
[-- snip --]
$ docker run -p 8080:8080 -it YOURORIGIN/jenkins-war:latest
hab-sup(MR): Butterfly Member ID ad83fdf99b7d4a6fa719c60fedb2fa3f
hab-sup(SR): Adding jjasghar/jenkins-war/2.9/20170410195441
hab-sup(MR): Starting butterfly on 0.0.0.0:9638
hab-sup(MR): Starting http-gateway on 0.0.0.0:9631
hab-sup(SC): Updated config.xml 14987bc61b9df77b5ff24736af6c1b6b3301240bef151cb85c0057c29ee02500
jenkins-war.default(SR): Initializing
jenkins-war.default(SV): Starting process as user=hab, group=hab
jenkins-war.default(O): + hab pkg path core/jre8
jenkins-war.default(O): + export JAVA_HOME=/hab/pkgs/core/jre8/8u111/20161214012044
jenkins-war.default(O): + export JENKINS_CONFIG=/hab/svc/jenkins-war/config
jenkins-war.default(O): + export JENKINS_HOME=/hab/svc/jenkins-war/data
jenkins-war.default(O): + hab pkg path core/gcc-libs
jenkins-war.default(O): + LD_LIBRARY_PATH=/hab/pkgs/core/gcc-libs/5.2.0/20161208223920/lib

[-- snip --]

jenkins-war.default(O): Apr 10, 2017 9:41:31 PM hudson.WebAppMain$3 run
jenkins-war.default(O): INFO: Jenkins is fully up and running
```

After this, you can run something like:

```shell
$ docker run -d -p 8080:8080 YOURORIGIN/jenkins-war:latest
dbd600327638f247fd5851c8b7cdbf08a2c09b12908cc38bdeb51b0060d37b7c
$ docker logs dbd600327638f247fd5851c8b7cdbf08a2c09b12908cc38bdeb51b0060d37b7c # to get the API key
```

[habitat]: https://www.habitat.sh/
[photon]: https://vmware.github.io/photon/
[jenkins]: https://github.com/jjasghar/habitat-jenkins
