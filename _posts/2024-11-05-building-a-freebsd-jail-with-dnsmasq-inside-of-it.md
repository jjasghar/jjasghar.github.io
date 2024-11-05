---
layout: post
title: "Building a freebsd jail with dnsmasq inside of it"
date: 2024-11-05 09:59:49
categories: freebsd sysadmin adblock
---

Ever since FreeBSD 5?6? I've always wanted to get FreeBSD jails working. Only recently I've gotten it working,
and I couldn't be prouder. I built a "personal pi-hole" with DNSMasq with an inspiration of a few other blogposts.

- <https://vlads.me/post/setting-up-dns-adblocker-freebsd-jail/>
- <https://kbeezie.com/freebsd-jail-single-ip/>

Let's start by setting up the _actual_ jail.

## FreeBSD Jail

First thing first, you "install" the Jail source code with `bsdinstall` like the following:
```bash
bsdinstall jail /usr/home/jails/dnshole
```

Next you need to create a `/etc/jail.conf.d/<jailname>.conf` where `jailname` is the name of jail like `dnshole`.
```
dnshole {
        host.hostname = dnshole.tardis; # hostname
        ip4.addr = "192.168.0.3/24";
        interface = lo1;
        path = "/usr/home/jails/dnshole";       # path to jail
        devfs_ruleset = 2; # devfs ruleset
        mount.devfs;                    # mount devfs inside
        #allow.raw_sockets=1;
        exec.start = "/bin/sh /etc/rc"; # start command
        exec.stop = "/bin/sh /etc/rc.shutdown"; # stop command
}
```

After that you need to confirm your _host_ machine's `rc.conf` you will probably need to add something like:
```
gateway_enable="yes"
ipv6_gateway_enable="yes"
jail_enable="YES"
jail_enable="YES"
pf_enable="YES"
cloned_interfaces="lo1"
ipv4_addrs_lo1="192.168.0.2-20/24"
```

And finally you'll need to add routing rules for `pf.conf`, this is just a start, but hopefully it'll make some sense:
```
IP_PUB="192.168.86.116"
IP_JAIL="192.168.0.2"
DNS_JAIL="192.168.0.3"
NET_JAIL="192.168.0.0/24"
PORT_WWW="{80,443,2020}"
scrub in all
nat pass on re0 from $NET_JAIL to any -> $IP_PUB
rdr pass on re0 proto tcp from any to $IP_PUB port $PORT_WWW -> $IP_JAIL
rdr pass on re0 proto { udp, tcp } from any to $IP_PUB port 53 -> $DNS_JAIL
```

**NOTE**: The DNS Jail is `0.3` but the `0.2` was a test one, but you can see you can forward ports like 80, 443, 2020 if you want.

Now you can spin up your jail with:
```
service jail start dnshole
```

And check the status with `jls`, and get "into" the jail with `jexec 1 sh`

## DNSMasq

Now that you've gotten a jail running, you need to install some software. This is pretty straight foward:

```
pkg install dnsmasq
```

Next edit the jail's `rc.conf`
```
dnsmasq_enable="YES"
```

If you want to debug the dns lookups, edit `/usr/local/etc/rc.d/dnsmasq` to add logging.
```
#command_args="-x $pidfile -C $dnsmasq_conf"
command_args="-x $pidfile -C $dnsmasq_conf --log-facility=/var/log/dnsmasq.log"
```

Next back up the `dnsconf.conf` and create a `dnsmasq.conf.d` directory:
```
cp /usr/local/etc/dnsmasq.conf /usr/local/etc/dnsmasq.conf.org
mkdir /usr/local/etc/dnsmasq.conf.d/
```

Copy this "sane" default in of all the setting (at least I want) into `/usr/local/etc/dnsmasq.conf`
```
domain-needed
bogus-priv
no-resolv
listen-address=0.0.0.0
bind-interfaces
no-hosts
cache-size=1000
log-queries
conf-dir=/usr/local/etc/dnsmasq.conf.d/,*.conf
server=8.8.4.4
server=2001:4860:4860::8844
```

Finall pull down some adblocking:
```
cd /usr/local/etc/dnsmasq.conf.d
fetch https://raw.githubusercontent.com/acidwars/AdBlock-Lists/master/adblock.conf
fetch https://raw.githubusercontent.com/acidwars/AdBlock-Lists/master/ads01.conf
dnsmasq --test
service dnsmasq restart
```

Then test your dnshole from another machine!
```
dig @192.168.86.116 freebsd.org
```
