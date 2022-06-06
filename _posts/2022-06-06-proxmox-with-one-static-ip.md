---
layout: post
title: "Proxmox with one static IP"
date: 2022-06-06 08:34:01
categories: sysadmin linux
---

Lets say you're building a [Proxmox][proxmox] machine, and you've only been given
one IP. Could be a static IP on a hosted bare metal machine, could be your home
lab, it doesn't really matter. If you wanted to spin up machine _inside_ of the
Proxmox instance these notes are for you.

First thing first, you need to create _two_ Linux Bridges. Lets say you have 
`eth0` as your physical NIC, with the IP of `192.168.1.100/24`. You'd want to
create `vmbr0` giving it that `.100` address and `slaving` it to `eth0`.

Now with that you can create `vmbr1` which'll be the actual NIC that you're
internal network will live on. You can give it any IP range, but strongly suggest
an internal network space like `10.`, `172.`, or `192.`.

Now when you spin up machines "inside" your network add the `vmbr1` NIC to it, 
wiether it be a LXC container or VM, and give it an IP in that space and you should
be able to `ping` around.

If you want to get out to the internet you need to some `iptables` built.

`ssh` into your host machine and run this:
```
iptables -A POSTROUTING -t nat -s 172.16.1.0/24 -j MASQUERADE
```

Where the `172` is the network that you gave `vmbr1` and you'll be able to
`ping`/`curl` the internet.

If you'd like to set up some NATing from that static IP, here are some useful
`iptables` for `80`,`8080`,`9000`. **Note**: the internal IP and `public-ip` 
will be different for you.

```
iptables -t nat -A PREROUTING -p tcp -d <public-ip> --dport 80 -i vmbr0 -j DNAT --to-destination 172.16.1.100:80
iptables -t nat -A PREROUTING -p tcp -d <public-ip> --dport 8080 -i vmbr0 -j DNAT --to-destination 172.16.1.101:8080
iptables -t nat -A PREROUTING -p tcp -d <public-ip> --dport 9000 -i vmbr0 -j DNAT --to-destination 172.16.1.200:9000
```

Finally, adding static IPs isn't fun, so it's useful to have a DHCP server on that network.

Here's the _minimal_ `dnsmasq` you can use to get it to give out IPs in `172.16.1.0/24`
network. **Note**: the `172.168.1.1` is the IP for `vmbr1` which is the gateway, and
I'm only giving out IPs from `.50` to `.240`.

```ini
server=8.8.8.8
server=8.8.4.4
interface=ens18
listen-address=172.16.1.2
expand-hosts
domain=asgharlabs.io
dhcp-range=172.16.1.50,172.16.1.240,12h
dhcp-option=3,172.16.1.1
dhcp-leasefile=/var/lib/misc/dnsmasq.leases
dhcp-authoritative
```

[proxmox]: https://www.proxmox.com
