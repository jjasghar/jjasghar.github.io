---
layout: post
title: "Using docker and the ibmcloud together"
date: 2019-03-07 13:14:33
categories: sysadmin docker ibm
---


# Starting with Docker and IBM cloud

If you are starting your journey into the contiainerized world the first thing you'll
come across is [docker][dockerweb]. This blog post will give you the jump start you
need to be able to start playing with `docker`, `Dockerfile`s, the IBM Cloud, and our
IBM Container Registry. OK! Lets get started.

## Docker

You need to install Docker, there are quite a few ways do do this, but the first and
best place to is by going to [the main docs site][dockerinstall]. Try to keep the the
most up-to-date, and install it for your operating system.

At the time of this blog post, there are two main editions of Docker, I'd like to take
a moment to discuss the different versions here. First there is Docker Community Edition,
or `docker-ce`, and Docker Enterprise Edition, or `docker-ee`. They both have advantages and
are aimed at different use cases. I strongly suggest after walking through this blog post
revisiting if `docker-ce` or `docker-ee` is the right fit for you.

Which ever you choose, these commands should work on either.

### Docker CLI

After you have `docker` installed, bring up a command prompt. During the installation,
there is a sanity check the documentation sometimes ask you to run. We are going to run
the following again to make sure everything is working as expected.

```bash
docker run hello-world
```

You should see something like the following, if you get an error or something your installation
is not set up correctly and you should resolve this before going any farther.

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Congratulations! You have a working `docker` instance. Lets start playing with it.

The next command I'd like to you try is the following, depending on your internet
access it could take a little time to run, but luckily it'll be cached on your machine
after which we'll visit here in a few.

```bash
docker run -it centos:latest /bin/bash
```

As this runs lets talk about what's happening. Running the `docker` command you tell it
I'd like the `latest` copy from the public docker hub, ran with the shell of `/bin/bash`.
The command runs off to the internet, sees the version you have, checks against either
the SHA of your cached image, or if you don't have it pulls it down from docker hub.
For clarity sake the `-it` runs it as `interactive` and creates a `tty` for the container.
You should see the following now.

```bash
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a02a4930cb5d: Pull complete
Digest: sha256:184e5f35598e333bfa7de10d8fb1cebb5ee4df5bc0f970bf2b1e7c7345136426
Status: Downloaded newer image for centos:latest
[root@2de726a5fcb8 /]#
```

Congratulations! You now have your second docker container running. Go ahead and type `exit`, and
run that `docker` command again.

```bash
[root@2de726a5fcb8 /]# exit
exit
$ docker run -it centos:latest /bin/bash
[root@583c6cec5d41 /]#
```

Notice how the `@2de726a5fcb8` and `@583c6cec5d41` are different? It's because when you spin up
containers this way they are ephemeral. Ephemeral means they only live for the time as they are
running, so as soon as you exit it is now gone. We'll show you how to do longer lived containers
in a little while.

Now lets play around in the container for a second. Being a CentOS machine, you have `yum` so
lets install something.

```bash
[root@583c6cec5d41 /]# yum install vim

[-- snip --]

Transaction Summary
=======================================================================================================
Install  1 Package (+32 Dependent packages)

Total download size: 19 M
Installed size: 63 M
Is this ok [y/d/N]: y

[-- snip --]

  perl-podlators.noarch 0:2.5.1-3.el7                  perl-threads.x86_64 0:1.87-4.el7
  perl-threads-shared.x86_64 0:1.43-6.el7              vim-common.x86_64 2:7.4.160-5.el7
  vim-filesystem.x86_64 2:7.4.160-5.el7                which.x86_64 0:2.20-7.el7

Complete!
[root@583c6cec5d41 /]#
```

Now you can run `vim` in your container! Try it out.

```bash
[root@583c6cec5d41 /]# vim
```

If you don't know you, to exit `vim` you type: `:q` and you should see your command prompt again.

Now type `exit` again.

```bash
[root@583c6cec5d41 /]# exit
exit
$ docker run -it centos:latest /bin/bash
[root@86f1f3872cbe /]# vim
bash: vim: command not found
[root@86f1f3872cbe /]# exit
exit
$
```

Just to reinforce this notion when your container is gone any changes inside are gone too.

### Dockerfile

OK, so we can spin up a container and make changes to it, but how do me keep changes around?
There are a few ways to do this, and I'll like you discover them yourself, but I'll walk you
through the most common way to achive this; using a `Dockerfile`.

Ok, back at your command prompt make a new directory, and change directory into it, and use
your `$EDITOR` of choice to open a new file called `Dockerfile`.

```bash
$ mkdir docker-tutorial
$ cd docker-tutorial
$ $EDITOR Dockerfile
```

I'll hit the highlights of `Dockerfiles` here, but I strongly suggest you take a look at [the
documentation][dockerfile] on Best Practices for them to get a taste of their power.

In your `Dockerfile` write out the following:

```
FROM centos:latest
RUN yum install vim -y && mkdir /vim
WORKDIR /vim
ENTRYPOINT ["vim"]
```

Save the file and make sure it's called `Dockerfile`.

Lets talk about what these three lines mean.

- `FROM`       : creates a layer from the `centos`:`latest` Docker image
- `RUN`        : builds your container by installing `vim` into it and creating a directory called `/vim`
- `WORKDIR`    : informs the container where the working directory should be for it
- `ENTRYPOINT` : is the command that is ran when the container starts, instead of `/bin/bash` like how we did above

Now lets build this locally. Run the following in the directory that the `Dockerfile` is and lets see this come
together.

```bash
$ docker build .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos:latest
 ---> 1e1148e4cc2c
Step 2/4 : RUN yum install vim -y && mkdir /vim
 ---> Running in ebd37633ab31
Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: mirror.fileplanet.com
 * extras: mirror.ash.fastserv.com
 * updates: www.gtlib.gatech.edu
Resolving Dependencies

[-- snip --]

Step 4/4 : ENTRYPOINT ["vim"]
 ---> Running in 82618eb1e891
Removing intermediate container 82618eb1e891
 ---> eda2652aa25e
Successfully built eda2652aa25e
$
```

**NOTE**: You should have a different hash `eda2652aa25e` please keep that in mind.

Congratulations! You have built your first `Dockerfile` and customized docker container.

Lets give it a run now. Go ahead and run the following command.

```bash
$ docker run -it eda2652aa25e
```

You should see `vim` start up! Remember, `:q` is how to quit it, and then you should see your command
prompt again. You can run this as many times as you'd like and you'll have a new instance of `vim`
each time; but you can't actually save anything or read anything because it's a container right? Lets
fix this.

We are going to add a bind mount, and volumes to mount the local directory into the container. If you'd
like to read more about mounts and volumes I suggest starting [here][dockervolumes], it's one of the harder
concepts but worth your time.

On your command line first create a file called `hello`, then save the words `hello world` in it.

```bash
$ EDITOR hello
```

Now, lets mount the local directory into our container.

```bash
$ docker run -it -v ${PWD}:/vim eda2652aa25e
```

Inside of `vim` type `:e hello`. You should see `hello world` come up! As you can see, you now have opened
the file you created on the host machine, created a contiainer with `vim` inside, mounted the directory
and was able to open the file!

If you'd like you can type `i` and type something out, then type `:wq` when your done. The container should
close out, and then you can type the following on your command line:

```bash
$ cat hello
hello world
I added th is line from my container
$
```

**Note**: Obviously `I added th is line from my container` will be what you write not this statement.

Awesome! Now lets start figuring out how to share this container to the world.

### IBM cloud

I'm going to assume you have your `ibmcloud` CLI installed and working, if not take a look [here][ibmclinstall]
and walk through the installation process.

Go ahead and log in to make sure you are authenticated against the back end you'd like.

```bash
$ ibmcloud login --sso

API endpoint: https://api.ng.bluemix.net

Get One Time Code from https://identity-2.us-south.iam.cloud.ibm.com/identity/passcode to proceed.
Open the URL in the default browser? [Y/n]> Y
One Time Code >
Authenticating...
OK

[-- snip --]


Tip: If you are managing Cloud Foundry applications and services
- Use 'ibmcloud target --cf' to target Cloud Foundry org/space interactively, or use 'ibmcloud target --cf-api ENDPOINT -o ORG -s SPACE' to target the org/space.
- Use 'ibmcloud cf' if you want to run the Cloud Foundry CLI with current IBM Cloud CLI context.

$
```

Now lets create a container registry name-space for you. You need to take a moment and think of something
globally unique, and that will be descriptive enough to tell you what it is. If you're lazy like me using
your username or something to that effect is a safe bet.

```bash
$ ibmcloud cr namespace-add jjasghar
Adding namespace 'jjasghar'...

Successfully added namespace 'jjasghar'

OK
```

This command uses the container registry plugin to create the name space of `jjasghar` in my account.

Luckily IBM has created some short cuts for you. If we would like to push our container to the IBM
container registry that we have created it's just one command.
Lets quickly talk about what's happening before running this. This will build the container, tag it
with the name of `vim` and the version of `1` and then push it to the registry. You'll need to change
your name from the `jjasghar` and maybe change the name of the container too.

**NOTE**: Don't forget the `.` at the end, or wherever you have your `Dockerfile` that's how the build command
knows where to run this.

```bash
$ ibmcloud cr build --tag registry.ng.bluemix.net/jjasghar/vim:1 .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM centos:latest
latest: Pulling from library/centos
a02a4930cb5d: Pull complete
Digest: sha256:184e5f35598e333bfa7de10d8fb1cebb5ee4df5bc0f970bf2b1e7c7345136426
Status: Downloaded newer image for centos:latest
 ---> 1e1148e4cc2c
Step 2/4 : RUN yum install vim -y && mkdir /vim

[-- snip --]

Successfully tagged registry.ng.bluemix.net/jjasghar/vim:1
The push refers to repository [registry.ng.bluemix.net/jjasghar/vim]
36ce508f1fe2: Pushed
071d8bd76517: Pushed
1: digest: sha256:831fcbac319dda1aab3d022c408ecc5cc1c1b825bcd90fc7694c3d4f0ef4eb9a size: 741

OK
```

Huzzah! We now have a container pushed to the IBM container registry that we have created!

Lets verify that it's pushed run the following command to list out your containers:

```bash
$ ibmcloud cr image-list
Listing images...

REPOSITORY               TAG   DIGEST         NAMESPACE   CREATED         SIZE     SECURITY STATUS
us.icr.io/jjasghar/vim   1     831fcbac319d   jjasghar    4 minutes ago   120 MB   No Issues

OK
```

Now lets pull your container down run the following command:

```bash
$ docker run -v ${PWD}:/vim -it registry.ng.bluemix.net/jjasghar/vim
Unable to find image 'registry.ng.bluemix.net/jjasghar/vim:latest' locally
docker: Error response from daemon: Get https://registry.ng.bluemix.net/v2/jjasghar/vim/manifests/latest: unauthorized: authentication required.
```

Oh no! So we need to log in to the registry, run the following now:


```bash
$  ibmcloud cr login
Logging in to 'registry.ng.bluemix.net'...
Logged in to 'registry.ng.bluemix.net'.

[-- snip --]

$ docker run -v ${PWD}:/vim -it registry.ng.bluemix.net/jjasghar/vim
Unable to find image 'registry.ng.bluemix.net/jjasghar/vim:1' locally
1: Pulling from jjasghar/vim
a02a4930cb5d: Already exists
209873925a88: Pull complete
Digest: sha256:831fcbac319dda1aab3d022c408ecc5cc1c1b825bcd90fc7694c3d4f0ef4eb9a
Status: Downloaded newer image for registry.ng.bluemix.net/jjasghar/vim:1
# and you should see vim now
```

Success! You now know how to create a container and publish it to the IBM Container Registry!

Last thing is to remove our container. Run the following commands to clean up your local cache,
and remove from your name space on the cloud registry.

```bash
$ docker rmi registry.ng.bluemix.net/jjasghar/vim:1
```

Where your name space, image name, and version are above. Then finally remove it from the cloud registry:

```bash
$ ibmcloud cr image-rm us.icr.io/jjasghar/vim:1
Deleting image 'us.icr.io/jjasghar/vim:1'...

Successfully deleted image 'sha256:831fcbac319dda1aab3d022c408ecc5cc1c1b825bcd90fc7694c3d4f0ef4eb9a'

OK
```

Thanks for walking through this with me, hopefully you feel more comfortable with some generic `docker`
commands, how `Dockerfiles` work, and finally how to use the IBM cloud container registry. If you have
any questions or thoughts never hesitate to reach out to me via twitter at `@jjasghar`.

[dockerinstall]: https://docs.docker.com/
[dockerfile]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
[dockervolumes]: https://docs.docker.com/storage/volumes/
[dockerweb]: https://docker.com
[ibmclinstall]: https://console.bluemix.net/docs/cli/reference/ibmcloud/download_cli.html#install_use
