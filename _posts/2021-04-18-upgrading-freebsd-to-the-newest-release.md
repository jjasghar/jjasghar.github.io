---
layout: post
title: "Upgrading FreeBSD to the newest release"
date: 2021-04-18 11:44:18
categories: freebsd sysdamin
---

I swear I've written this before, but I couldn't find it. So here we go, "another"
post on how to update your major RELEASE version of FreeBSD.

Here are my commands to upgrade from FreeBSD 12 to 13, this should also work for other major versions:

## First confirm your FreeBSD instance is up to date

```
freebsd-update fetch
freebsd-update install
pkg upgrade
```

This will verify and install any outstanding patches.

## Preparation for the newest release
At this writing it was `13.0-RELEASE`.

```
freebsd-update -r 13.0-RELEASE upgrade
```
You will see a bunch of patches and other things being updated. _Most_ likely you’ll
say `Y` to everything.

After everything is patched, run the following commands to “commit” the changes:

```
freebsd-update install
```

After this is done, run the `reboot` command, and you’ll be in the newest release.
## Cleanup and confirmation
Now that you are running the newest release, you need to pull/update your packages and clean up your system.

```
freebsd-update install # yes again
```

This will clean up any non-needed files from your previous update.

Now we need to upgrade your packages:

```
pkg-static install -f pkg
pkg bootstrap -f
pkg update
pkg upgrade
```

This will force an update of your `pkg` package and get everything into a state ready to be upgraded.

Run the following command one last time, and then after it’s complete you’re done!

```
freebsd-update install
```

#blogpost
