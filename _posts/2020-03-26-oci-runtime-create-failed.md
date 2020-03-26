---
layout: post
title: "OCI runtime create failed"
date: 2020-03-26 15:16:10
categories: docker linux fedora ubuntu
---

On a twitch stream on March 26, hell, I'm actually writing this
blog post _on_ the steam. We ran into this error:

```bash
sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:f9dfddf63636d84ef479d645ab5885156ae030f611a56f3a7ac7f2fdd86d7e4e
Status: Downloaded newer image for hello-world:latest
docker: Error response from daemon: OCI runtime create failed: container_linux.go:346: starting container process caused "process_linux.go:297: applying cgroup configuration for process caused \"open /sys/fs/cgroup/docker/cpuset.cpus.effective: no such file or directory\"": unknown.
ERRO[0003] error waiting for container: context canceled
```

We did some googling, and discovered it was a difference between, [`containerd`][containerd] and the original [`docker`][docker] implementation. We found the [github issue][github] but it was buried in the comments. I've put this post together hopefully to short circut someones searching.

If you have Fedora "31" which I'm assuming is any "new" Fedora, do the following:

```bash
cat /etc/redhat-release
Fedora release 31 (Thirty One)
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
sudo reboot
```

And if you have a Ubuntu "19.10" which I'm assuming is also any "new" Ubuntu/Debian based installation, do the following:

```bash
cat /etc/debian_version
buster/sid
sudo update-grub "systemd.unified_cgroup_hierarchy=0"
```


[containerd]: https://containerd.io/
[docker]: https://docker.com
[github]: https://github.com/docker/cli/issues/297#issuecomment-547022631
