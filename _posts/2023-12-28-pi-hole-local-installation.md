---
layout: post
title: "pi-hole local installation"
date: 2023-12-28 14:40:38
categories: sysadmin docker
---

Every once in a while I realize I’ve been seeing way to many ads online. I sometimes run a pi-hole server on a raspberry pi in my house, but alas I’m not always there. So here is my howto run pi-hole on your local machine, so you can have a “disposable” pi-hole instance locally.

You’ll need:
* docker
* docker-compose
* a little bit of patience

First thing is you need a place to put the following `docker-compose.yml` file. Also a sanity check on `docker` too.

```bash
docker run hello-world
```

Assuming that works, put the following into a directory probably called `pi-hole` or something
```bash
cd ~
mkdir pi-hole
cd pi-hole
vim docker-compose.yml
```

```yaml
version: "3"

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "80:80/tcp"
    environment:
      TZ: 'America/Chicago'
      # WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    restart: unless-stopped
```

Run the following to bring up the container:
```bash
docker compose up
```

Verify that all the ports and you can reach the http://localhost/web/admin page. If you can, then you’re almost done.

Fix your DNS settings to `127.0.0.1` and then you should be able to start seeing traffic go through pi-hole. Edit how you want, and you’re done!
