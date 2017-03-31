---
layout: post
title: "PhotonOS as your backend for kitchen-docker"
date: 2017-03-29 15:09:11
categories:
---

![](https://camo.githubusercontent.com/4d4cc8eedee941e882dc521eb37dd354c6beca20/687474703a2f2f73746f726167652e676f6f676c65617069732e636f6d2f70726f6a6563742d70686f746f6e2f766d772d6c6f676f2d70686f746f6e2e737667)

I have been interested in using [PhotonOS][photon] as my backend [docker][docker] OS for a while now.
If you don't know, Photon is VMware's next generation Cloud Native OS that is native for their vSphere,
ecosystem.

So imagine this: your company has a huge VMware based data center, you're starting to leverage the
concept of "infrastructure as code," and you're using Chef or another plugin to [test-kitchen][testkitchen].
You can't run Docker or Virtualbox on your laptop due to security reasoning, but you have a lab
environment testing out PhotonOS.

*Naturally it's easier to get another VMware product, especially because it's FREE, in your data
centers instead of approval for Virtualbox on your local laptop right?*

This tutorial is a way to get [kitchen-docker][kitchendocker] to talk to PhotonOS and any settings
or gotcha's you might find.

## PhotonOS setup

So, the first thing you have to do is make sure that PhotonOS is installed in your vCenter. If your
team hasn't done this yet, you can go to the [PhotonOS Wiki][photonwiki] and walk through the process.
It's just importing a `.ova` with the typical workflow that any VMware engineer can understand.

After the power on and boot up, you'll need to login to the machine with the password of `root/changeme`
and change the password to something secure.

When you get to the command prompt: `root@photon-machine [ ~ ]#` you are ready for the next steps.

I could repeat everything here, but it's probably just better read it directly from [Ryan Kelly][ryan]'s mouth.
So, click this following [link to allow PhotonOS to allow remote connections][remote].

To test this, bring up your command line and type

```shell
$ export DOCKER_HOST=tcp://DOCKERHOST:2375
$ docker info`
```

You should see one of the lines come back say: `Operating System: VMware Photon/Linux`.

Congrats! You now have a remote docker host running PhotonOS.

## test-kitchen setup

Ok, now here comes the specific `kitchen` steps.

First, open up your `.kitchen.yml` or `.kitchen.local.yml` or `.kitchen.docker.yml` which ever you want to drive
your kitchen settings.

Here's a quick snippet to set it up for the changes you need to make at the top:

```yaml
---
driver:
  name: docker
  socket: tcp://DOCKERHOST:2375
```

With this, you'll also need to either add `kitchen-docker` to your Gemfile if you use one, or `chef gem install kitchen-docker`
to put it inside the chefdk.

With these settings and the gem installed in the correct place, you can now run:

```shell
$ kitchen list
```

You should see all of the base `.kitchen.yml` settings such as:

```shell
Instance             Driver  Provisioner  Verifier  Transport  Last Action    Last Error
default-ubuntu-1604  Docker  ChefZero     Inspec    Ssh        <Not Created>  <None>
default-centos-72    Docker  ChefZero     Inspec    Ssh        <Not Created>  <None>
```

Go ahead and run `kitchen test -c 2`, and you'll see how fast it is!

**NOTE**: The first initial run will take a bit due to the caching of images on the PhotonOS host, but after that, it'll be significantly faster.

**Double NOTE**: As of the writing of this blog post, 2017-03-30 it seems that there is a problem with the `overlay` and `inode` usage. You'll need to keep an eye on it, if you run out of `no space left on device` you need to look at `df -i`. A way to clean up the overlay directory "safely" is via:

```shell
$ docker rm $(docker ps --all | cut -f 1 -d\  )
$ docker rmi $(docker images | cut -d\  -f 1)
$ docker rmi $(docker images -q)
```

**Triple NOTE:** I have put in issue [#619][619] to attempt to track this `inode` issue. I did discover that adding another harddrive to the `OVA` and mounting it at `/var` seems to be a workaround for this issue.

[619]: https://github.com/vmware/photon/issues/619
[photon]: https://vmware.github.io/photon/
[docker]: https://www.docker.com/
[testkitchen]: http://kitchen.ci/
[kitchendocker]: https://github.com/test-kitchen/kitchen-docker
[photonwiki]: https://github.com/vmware/photon/wiki
[ryan]: https://twitter.com/vmtocloud
[remote]: https://blogs.vmware.com/cloudnative/enable-docker-remote-api-photon-os/
