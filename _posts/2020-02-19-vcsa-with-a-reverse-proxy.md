---
layout: post
title: "VCSA with a reverse proxy"
date: 2020-02-19 14:15:36
categories: vmware sysadmin nginx
---

## Introduction

In the IBM Cloud, there is a base management network. If you don't know,
there isn't an _obvious_ way to access it from your laptop. After some headbanging
I've figured out how to bridge my management network and my workload network
with a [nginx reverse proxy][proxy] to allow me to run commands from working
network to interface with vCenter.

> tl;dr: I want to run terraform from my laptop to my vCenter, I can't VPN into my 10.x but I can VPN into my 172.x network. Using a machine that bridges with nginx I can now talk to my vCenter through my proxied machine.

## Setup

I'll go ahead and use my _real_ IPs here so we don't get lost with fake numbers,
luckily these are all behind a firewall that is nowhere near the internet, so knowing
these you shouldn't be able to use these other than an example. Ok, here we go:

First thing, I had to create a machine that had two NICs, working on the internal management
network, and my internal VPN network. My management network was named: `asgharlabs-asghar-dpg-mgmt`
with the IP range of: `10.220.145.x`. My VPN/workload network was named: `vxw-dvs-40-virtualwire-3-sid-6002-Workload`
with the IP range of: `172.16.10.x`. I added a static IP to the management network,
but let DHCP take care of my workload network. Awesome, I have a machine that can
now ping both sides and reach both locations I'm trying to get to.

Next, I do the obvious, I install [nginx][nginx], I'm using CentOS, so I did it via
the following command:

```bash
sudo yum install nginx
```

Then I enable and start the process:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

I verified that I saw the "Welcome to Nginx" page, to verify that everything was setup
as expect.

Next, I made sure my `firewalld` was set up correctly:

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

And finally, ([Major](https://twitter.com/majorhayden/) please don't yell at me) I disabled
SELinux. I converted it from `enforcing` to `disabled` and reboot my machine. Now, let’s
configure the meat of this blog post.

## Nginx reverse proxy set up

So if you've Google'd around looking for a way to do this, you probably have come
across [this repository](https://github.com/nicholasvmoore/vcsa-nginx-rp), 5 years ago
this was really the only reference for this action. I took from his work and updated
for the VCSA 6.7+ that I'm running. Luckily everything goes over `https` now and standard
ports, so it actually makes the `vcsa.conf` hella, easier.

Take the following configuration file, drop it in a logical place, such as `/etc/nginx/conf.d/vcsa.conf`
and change out the IPs, that is commented on in it.

```conf
proxy_set_header            Host            $http_host;
proxy_set_header            X-Real-IP       $remote_addr;
proxy_set_header            X-Forwared-For  $proxy_add_x_forwarded_for;

#
# The upstream VCSA hostname or IP address for port 443
#
upstream vcsa-443 {
  server 10.220.145.x:443;
}

#
# HTTP => HTTPS redirect
#
server {
  listen        80;
  server_name   172.16.10.x; # Your local machine

  location / {
    allow all;
    return 302 https://$server_name$request_uri;
  }
}

#
# Main HTTPS Reverse Proxy for the VCSA
#
server {
  listen        443 ssl;
  server_name   172.16.10.217;

  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt; # you'll need your selfsigned cert here
  ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key; # you'll need your selfsigned key here
  ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers    HIGH:!aNULL:!MD5;
  keepalive_timeout 60;

  location / {
    allow all;
    proxy_set_header Host $http_host;
    proxy_pass https://vcsa-443;
    proxy_ssl_server_name on;
  }
}
```

Lastly, you'll need to set up some self-signed certs, I found the easiest way was [here][certs], it
walks you through everything, and as you can see drops your certs in `/etc/ssl/certs` which is
coded in the above configuration file.

Finally, restart `nginx` via something like `service nginx restart` and you should be good to go.

Oh! A final gotcha, you may have to add to your DNS or (like in my case) to my `/etc/hosts` file
the full VCSA hostname to the _proxied_ VM, (`x` is the IP ;) ). For instance:

```bash
172.16.10.x asgharlabs-vc.asgharlabs.io
```

After figuring out the incantation, it seems pretty straight forward, and hope this helps someone
in the future.

[certs]: https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7
[nginx]: https://www.nginx.com/
[proxy]: https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
