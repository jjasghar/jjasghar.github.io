---
layout: post
title: "Options to run a local kubernetes cluster"
date: 2019-03-20 09:29:07
categories: linux sysadmin kubernetes
---

**Note:** This is as accurate as 2019-03-20 this stuff moves fast so there might be other options now.

# Running kubernetes locally

This post is a general post about the different ways to deploy kubernetes locally, and emulate what you
would do in a real cloud on your local machine. This can be helpful not only with cost savings, but the ability
to leverage portions for CI/CD on your cloud native applications, or the situation where you don't have
reliable internet access and want to do "real" work.

The different technologies we'll be walking through are:

- [minikube](#minikube)
- [docker in docker](#dockerindocker)
- [microk8s](#microk8s)
- [k3s](#k3s)
- [building your own from scratch](#scratch)

## Pre-Requirements

Before we go any farther I strongly suggest a machine with at least 8 Gigs of RAM. Some of
the following options are VM based and modern operating systems require at least 4 Gigs of
RAM before issues can arise. Having 8 Gigs of RAM allows for [VirtualBox][virtualbox] or
the like the like to get enough resources to be able to do more then the bare minimum.

Next you might as well `kubectl` installed on your local machine, it's the canonical way to
interface with any kubernetes cluster, and your local instance is no different. I will say
that you could call directly to the API, but I'd only suggest doing this if you had a real
reason to.

<a name="minikube"></a>
## minikube
<a name="minikube"></a>

`minikube` is the defacto entry point for most people learning or figuring out how to run
kubernetes on a local machine. I've even heard of some people using `minikube` to run Dev or
QA environments on shared VMs on clouds. This feels a lot like the old [devstack][devstack]
options back in the OpenStack days. Honestly, it has echos of it, and I feel like it's probably
the best analogy.

Just like any seasoned veteran of the OpenStack ecosystem would say, devstack is a designed for
a specific use case, and that's how `minikube` is targeted too. `minikube` was/is the answer
to the question "What is the fastest way I can get kubernetes running on my laptop?" and it
succeeds at this.

You should note though, it's only one worker node, one machine, relays on Virtualbox or the like
for virtualization and has only **2** Gigs of RAM by default to use. This is all but a toy, you can
run your typical commands against it, but you can't run anything that isn't stock kubernetes.

So how do you fix this, you can run `minikube` with a couple other options to give it more resources
on start time.

So how do you fix this? Luckily you have some commands you can run!
```bash
minikube --cpus 2 --memory 4024 start
```
The above, will give your `minikube` instance 2 virtual cpus, and 4 Gigs of RAM. This should allow
you to do more with it, like install [istio][istio] but not much more. If you want to create/test
something closer to your production instance, you should give it more resources. That's up to your
restrictions and specific situation.

After you're done with the instance, I'd suggest either shutting it down, or deleting it completely:
```bash
minikube stop
```
If you plan on walking away from your local computer or not needing the pods your are running for long
term, this is a good opportunity to do the following to the VM that is running your kubernetes cluster
isn't taking up unneeded resources.
```bash
minikube delete
```

I should say, if you had installed `kubectl` we didn't have `export` anything or change any configuation
around, from what I know that's by design. `kubectl` defaults to the localhost that `minikube` runs on,
which allows for a small, but valuble, positive UX situation.

<a name="dockerindocker"></a>
## docker in docker
<a name="dockerindocker"></a>

Personally this in the choice I make. It's from the kubernetes-sigs project called [kubeadm-dind-cluster][dind].
It creates 3 docker containers on your local machine, one master and 2 worker nodes. It's as
close as you can get to what you'd really run in the real world. And because they are
just containers they spin up extremely quickly.

All in all, it's just a handful of commands, for instance:

```bash
wget https://github.com/kubernetes-sigs/kubeadm-dind-cluster/releases/download/v0.1.0/dind-cluster-v1.13.sh
chmod +x dind-cluster-v1.13.sh
```

As you can see at my writing of this it's version `1.13`, but it should be the same with whatever
version is released now.

```bash
# start the cluster
./dind-cluster-v1.13.sh up

# add kubectl directory to PATH
export PATH="$HOME/.kubeadm-dind-cluster:$PATH"

kubectl get nodes
NAME          STATUS    ROLES     AGE       VERSION
kube-master   Ready     master    4m        v1.13.0
kube-node-1   Ready     <none>    2m        v1.13.0
kube-node-2   Ready     <none>    2m        v1.13.0
```

And that's it! You now have a fully working kubernetes cluster running inside your docker
instance.

```
# k8s dashboard available at http://localhost:8080/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy

# restart the cluster, this should happen much quicker than initial startup
./dind-cluster-v1.13.sh up

# stop the cluster
./dind-cluster-v1.13.sh down

# remove DIND containers and volumes
./dind-cluster-v1.13.sh clean
```

<a name="microk8s"></a>
## microk8s
<a name="microk8s"></a>

At the time of this writing [microk8s][microk8s] was a relatively new comer to the space. It's supported by
Canonical, the company behind Ubuntu. It's a simple `snap` install like the following:

```bash
snap install microk8s --classic
```

Obiviously if you don't have `snap` you're out of luck, but if you can swing it, this is a way
to get a kuberenetes cluster up and running extremely quickly. One oddity with the system though
is every command is prefixed. Take a look at this example:

```bash
microk8s.kubectl get nodes
```

It has a huge advantage, you don't have to mess with your `KUBECONFIG` to talk to your local
cluster, but if you have been using kubernetes a long time you have to retrain your muscle
memory. Take a look at the [microk8s docs][microdocs] and play around with it.

<a name="k3s"></a>
## k3s
<a name="k3s"></a>

[k3s][k3s] is the newest way to get kubernetes cluster on a local machine. Supported
by rancher, it has huge promise. To quote the Github page:
> Lightweight Kubernetes. Easy to install, half the memory, all in a binary less than 40mb.

That's extremely impressive. But it's designed for a specific use case not for the original
statement of running a kubernetes cluster on your laptop. It's designed for small environments
like IoT or Edge computing. If you have smaller environment it's worth checking out.


<a name="scratch"></a>
## from scratch on a local machine
<a name="scratch"></a>

Ok, so you've gotten to this point, maybe nothing has fit your use case, or hell you
like my writing style. The following is a very specific use case.
You've probably come here to spin up kubernetes from scratch on your
local machine, first of all, I salute you, second, wow. There are many packaged
ways to do what you're trying to do, and this will only cause heart ache.

I'm assuming you've gone through, and edited [kubernetes the hardway][k8shardway]
to work on your cloud of choice. Now you've taken it and ran it locally on a VM,
and now want bring it to your local dev environment. You might want to run
the most recent releases of kubernetes, and have no trouble figuring out what's
going wrong when your application has trouble. You, yes you, are unique. This whole
post was not aimed for you, and maybe it opened your eyes to something you haven't
thought of, but in general you, yes _you_ are a pioneer on your own boat. May
the wind will always be at your back, and have fun.

## Final Thoughts

Hopefully you can see there are a ton of options here. The crazy part is that
this is only as accurate as the **Note** at the top of the document. Kubernetes
is moving fast and becoming the way to run Cloud Native Applications, but
running it _without_ a cloud can be tricky. Most of the options that were
mentioned here hit most use cases and you'll need to do your homework
to see what fits you. Standardize on something, which will help your teams
become more successful when they all use the same base wrapper around this
amazing technology.

[devstack]: https://devstack.org
[dind]: https://github.com/kubernetes-sigs/kubeadm-dind-cluster
[k3s]: https://github.com/rancher/k3s
[k8shardway]: https://github.com/kelsyhighower/kubernetes-the-hardway
[istio]: https://istio.io
[microk8s]: https://microk8s.io/
[microk8sdocs]: https://microk8s.io/docs/
[virtualbox]: https://www.virtualbox.org
