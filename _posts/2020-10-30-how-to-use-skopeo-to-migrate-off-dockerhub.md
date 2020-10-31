---
layout: post
title: "How to use skopeo to migrate off Docker Hub"
date: 2020-10-30 15:15:15
categories: linux sysadmin docker
---

# Scope

If you don't know, it seems [Docker][docker] is starting to limit their free/anonymous
tier at their container registry.

> On Monday, November 2, 2020 at 9am Pacific Standard Time, Docker will begin enforcing rate limits on container pulls for Anonymous and Free users. Anonymous (unauthenticated) users will be limited to 100 container image pulls every six hours, and Free (authenticated) users will be limited to 200 container image pulls every six hours, when enforcement is fully implemented.

The above was taken from an email I received, needless to say I decided to migrate
away from Docker Hub.

These are the steps I took to get to both [quay.io][quay] and [GitHub Container Registry][ghcr],
hopefully it'll help someone in the future.

**Note**: If you want to see me figuring out this read time, this is my [youtube video][youtube]/stream. 

# Setup

First thing first, you need to install the app that'll do the heavy lifting for you.
This is from Red Hat and called [skopeo][skopeo]. There's a couple ways to do install
it, check out [the install.md][installmd] for your operating system.

## Login to Docker Registry

The first thing you do after getting `skopeo` installed, you need log into the Docker Registry,
you do it by the following command:

```bash
skopeo login docker.io
```

It'll ask for you `username` and `password`, and then you should be good.

# Migrate to `quay.io`

The first example I'm going to show is how to migrate to Quay.io. Quay is the Red Hat
container registry, and has some nice to haves built in. Lets go!

Now you need to log into Quay.io here, run the following command:

```bash
skopeo login quay.io
```

Now that you're logged into both Docker and Quay, you can copy from one to another, I'm not 100% sure
on this next statement...but I'm pretty sure you _don't_ proxy on your machine, you copy from one to another.
I have this one container from a stream called the "catapp", in my following examples that's what we'll migrate.

For a sanity check, I'll verify that `skopeo` can see the Docker Hub container, I'll use the `inspect` command
here:

```bash
skopeo inspect docker://jjasghar/catapp:latest
```

Notice the username of `jjasghar`, like most Cloud Native apps now-a-days it assumes you're talking to the Docker Hub.

Now that I can see that the container is there, I'll use the following to copy from Docker Hub to Quay.io:

*Note*: make sure you have the tag!

```bash
skopeo copy docker://jjasghar/catapp:latest docker://quay.io/jjasghar/catapp:latest
```

Notice how for quay, you need to add the `quay.io` but for docker you don't. I realized I've already said
this but it caught me a couple times.

With that you should see the copy starting to happen, when it's complete, open up the new registry repository
on <https://quay.io> and you'll notice it's set to private by default. You'll need to go into the settings
(I haven't figured out how to do it programmatic-ly) and flip it to "public." You should be good now!

# Migrate to GitHub Container Registry

Copying over to GitHub's container registry is a little more involved, but not to bad. You'll have to create
a [Personal Access Token aka PAT][PAT] to start. This will be your "password." Go ahead and log in via the following
command:

```bash
skopeo login ghrc.io
username: <github-username>
password: <github-PAT>
```

After that, it's just like copying over to quay, just pointing to a different URL, to complete 
our example:

```bash
skopeo copy docker://jjasghar/catapp:latest docker://ghcr.io/jjasghar/catapp/catapp:latest
```

And with that you should be able to go [here][ghcrpkg] (where `jjasgahr` is your username) 
and you should see your container in just a couple seconds.

[docker]: https://docker.com
[quay]: https://quay.io
[ghcr]: https://github.com/features/packages
[youtube]: https://www.youtube.com/watch?v=yrFjdV08f5w
[skopeo]: https://github.com/containers/skopeo
[installmd]: https://github.com/containers/skopeo/blob/master/install.md
[PAT]: https://github.com/settings/tokens/new
[ghcrpkg]: https://github.com/jjasghar?tab=packages
