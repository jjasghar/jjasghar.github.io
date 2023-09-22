---
layout: post
title: "Cheatsheet to install Docker or Podman"
date: 2023-09-22 09:52:56
categories: sysadmin
---

If you are looking for the commands to install Docker on an `apt` based system, or Podman on
a `rpm` based system look no farther. (I've googled this so many times, it's about time I have
it somewhere I can just _look_.)

## Podman

```bash
dnf -y install podman
podman run hello-world
dnf -y install podman-compose
```

I have also posted this on a [gist](https://gist.github.com/jjasghar/5d20a223ce8382d864554cbf6bec2d2e) where you
can `curl` against it and just run it.

```bash
curl -sSL https://gist.githubusercontent.com/jjasghar/5d20a223ce8382d864554cbf6bec2d2e/raw/0ad4a2e3206560344272638496b713c1b3f1e85f/run.sh | bash
```

## Docker

```bash
apt-get remove docker docker-engine docker.io containerd runc -y
apt-get update -y
apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
docker run hello-world
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.11.0/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

I have also posted this on a [gist](https://gist.github.com/jjasghar/fb554022aaa82daed160d61f34ecd746) where you
can `curl` against it and just run it.

```bash
curl -sSL https://gist.githubusercontent.com/jjasghar/fb554022aaa82daed160d61f34ecd746/raw/3570fa9fe76ef334f08f649f3cca25c872072c83/run.sh | bash
```
