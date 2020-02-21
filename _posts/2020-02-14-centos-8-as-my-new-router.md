---
layout: post
title: "CentOS 8 as my new router"
date: 2020-02-14 15:55:17
categories: linux centos sysadmin
---

I had to rebuild my router, and these are my notes. Hopefully I
won't have to look all this up _again_, in the future. I really
feel like I do this more then I should.

## IPv4 Forwarding

With two NICs, you're gonna need to forward some traffic. First
thing first, forward those packets:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo vi /etc/sysctl.d/99-sysctl.conf # put the 'net' in this file
```

## Static IP

Something I always seem to have to figure/google this.

Here is a template to edit: `/etc/sysconfig/network-scripts/ifcfg-<interface>`

```ini
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="<interface>"
DEVICE="<interface>"
ONBOOT="yes"
IPADDR=123.456.789.100
PREFIX=24
GATEWAY=123.456.789.1
DNS1=8.8.8.8
```

## Fail2Ban

Being this is going to be in the internet, you should install [fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page).

I have take these notes from [here](https://tecadmin.net/install-fail2ban-centos8/).

```bash
sudo dnf install -y epel-release fail2ban
```

Configure the local jail:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

> Now we need to make necessary changes in jail.local file to create ban rules. Editthis file in your favorite editor and make changes in [DEFAULT] section.

```ini
[DEFAULT]

# "ignoreip" can be a list of IP addresses, CIDR masks or DNS hosts. Fail2ban
# will not ban a host which matches an address in this list. Several addresses
# can be defined using space (and/or comma) separator.
ignoreip = 127.0.0.1/8 192.168.1.0/24

# "bantime" is the number of seconds that a host is banned.
bantime = 60m

# A host is banned if it has generated "maxretry" during the last "findtime" seconds. as per below
# settings, 2 minutes
findtime = 5m

# "maxretry" is the number of failures before a host get banned.
maxretry = 5

[ssh-iptables]

enabled  = true
filter   = sshd
action   = iptables[name=SSH, port=22, protocol=tcp]
           sendmail-whois[name=SSH, dest=root, sender=fail2ban@example.com, sendername="Fail2Ban"]
logpath  = /var/log/secure
maxretry = 3
```

Then enable and start the service:

```bash
sudo systemctl start fail2ban.service
sudo systemctl enable fail2ban.service
```

## DNSMasq

A lot of these steps were taken from [here](https://www.tecmint.com/setup-a-dns-dhcp-server-using-dnsmasq-on-centos-rhel/). Thank you for writing it.

My router is going to be my local DNS server and my DHCP server,
there are a ton of options out there, `dnsmasq` is the easiest
to combine the two.

Install `dnsmasq`, enable and start it:

```bash
sudo dnf -y install dnsmasq
sudo systemctl start dnsmasq
sudo systemctl enable dnsmasq
```

### DNS

Edit the configuration file:

```bash
sudo vi /etc/dnsmasq.conf
```

First thing you want to do is edit the listen address for
`dnsmasq`. My network is `172.16.10.0` so my `.1` is my
machine.

```bash
listen-address=127.0.0.1,172.16.10.1
```

Next, you want to edit the interface.

```bash
interface=ens224
```

Uncomment `expand-hosts` to help with the machines that
come and go. Also set your domain to your domain. :)

```bash
expand-hosts
domain=asgharlabs.io
```

Define the upstream DNS servers:

```bash
server=8.8.8.8
server=8.8.4.4
```

This is how to get the DNS portion up, go ahead and get out
of the file and run a sanity check:

```bash
sudo dnsmasq --test
```

`dnsmasq` uses your `resolv.conf` as your upstream DNS and your
local `hosts` file as your local DNS entry. Confirm they are set
up correctly now.

If you need to make changes, `NetworkManager` will override your
changes, so you need to make the file immutable:

```bash
sudo chattr +i /etc/resolv.conf
sudo chattr -i /etc/resolv.conf
sudo vi /etc/resolv.conf
sudo chattr +i /etc/resolv.conf
sudo lsattr /etc/resolv.conf
```

Now that everything is set up, we should restart `dnsmasq` and add
the firewall changes in:

```bash
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --add-service=dhcp --permanent
sudo firewall-cmd --list-all
```

### DHCP

Now that we have a working `dnsmasq` instance, lets set up the DHCP part.

Edit the `dhcp-range` in the `/etc/dnsmasq.conf`

```bash
dhcp-range=172.16.10.100,172.16.10.250,12h
```

Next, edit the `dhcp-leasefile` and make it authoritive by uncommenting:

```bash
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
dhcp-authoritative
```

Restart `dnsmasq` and you should be good!

```bash
sudo systemctl restart dnsmasq
```

## `firewalld` configuration

Now that you have DNS and DHCP running, you need to make sure
your router actually routes things.

You need to add masquerade to your `firewalld` chain.

```bash
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --reload
```

## OpenVPN configuration

Now that you have a working router, you probably want to VPN
into your network. Lets get [OpenVPN](https://www.openvpn.org) up and running.

First thing you need to do is install `git` and pull down `Nyr`'s repo
for automaticly configuring `openvpn`.

```bash
cd ~
sudo dnf -y install git
git clone https://github.com/Nyr/openvpn-install.git
```

Run the installer in the repository:

```bash
cd openvpn-install
sudo chmod +x openvpn-install.sh
./openvpn-install.sh
```

Follow the prompts...

EDIT: It seems I couldn't get "across" my network, so I had to edit the `/etc/openvpn/server/server.conf`
with the following:

```ini
push "route 172.16.10.0 255.255.255.0"
```

Now I can get to my *internal* network, which is what I was hoping for.

Congrats! You now have a working router/vpn machine!
