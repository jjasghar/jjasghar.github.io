---
layout: post
title: "Using bhyve on FreeBSD"
date: 2019-06-03 17:02:57
categories: sysadmin freebsd
---

# How to get `bhyve` to work on FreeBSD

I've been trying to figure out how to get this done for years. I finally got it
working today. Here are my notes and steps taken from multiple resources on the
internet and making this better.

I'll admit the [this video](https://www.youtube.com/watch?v=P_XtAdP0lvo) finally
got me over the edge.

**NOTE:** These commands all have to be run as `root`.

1) First thing first, install grub2-bhyve. This allows for the Linux kernel to
boot and is _required_.

```bash
pkg install grub2-bhyve
```

2) Next, verify your kernel settings:

```bash
kldstat
```

Look for: `vmm.ko` if not:

```bash
kldload vmm
```

Look for: `nmdm.ko` if not:

```bash
kldload nmdm
```

Look for: `if_bridge.ko` if not:

```bash
kldload if_bridge
```

Look for: `if_tap.ko` if not:

```bash
kldload if_tap
```

3) Create some sysctl values to allow for networking to work:

```bash
sysctl net.link.tap.up_on_open=1
sysctl net.inet.ip.forwarding=1
```

If you want to make it permanent (you probably do), which you should `echo` these out:

```bash
echo 'net.link.tap.up_on_open=1' >> /etc/sysctl.conf
echo 'net.inet.ip.forwarding=1' >> /etc/sysctl.conf
```

4) Edit your `boot/loader.conf` file by adding:

```bash
vmm_load="YES"
nmdm_load="YES"
if_tap_load="YES"
if_bridge_load="YES"
```

5) Create some remote terminals by adding to `/etc/remote`:

```bash
echo 'vm1:dv=/dev/nmdb0B:br#9600:pa=none:' >> /etc/remote
```

6) Create the `tap`, `bridge` file:

```bash
ifconfig tap1 create
ifconfig bridge0 create
```

7) Add the `tap` to the `bridge`:

```bash
ifconfig bridge0 addm tap1
```

8) Add your network device to your bridge, (whatever has your ip address with `ifconfig` is what you add here. Mine was `re0`.

```bash
ifconfig bridge0 addm tap1 addm re0 up
```

9) Create the place for your VM to live.

```bash
mkdir -p /vms/vm1
mkdir -p /vms/iso # put your ISOs here :)
cd /vms/vm1
```

10) Create a fake hard drive `.img` for your vm, this example is a 10g hard drive.

```bash
truncate -s 10g hdd1.img
```

11) Create a `device.map` file, this tells where the hard drive and CDROM is for booting.

```bash
vi device.map
```

```bash
(hd0) /vms/vm1/hdd1.img
(cd0) /vms/iso/debian-9.9.0-amd64-netinst.iso
```

12) Lets boot your new machine!

This sets the `grub` and memory, hit `enter` again with the installer it will drop you to another terminal.
```bash
grub-bhyve -r cd0 -m /vms/vm1/device.map -M 4096 vm1
```

In that terminal run something like the following.

```bash
bhyve -c 1 -m 4096 -H -P -A \
-l com1,/dev/nmdm0A \
-s 0:0,hostbridge \
-s 1:0,lpc \
-s 2:0,virtio-net,tap1 \
-s 3,ahci-cd,/vms/iso/debian-9.9.0-amd64-netinst.iso \
-s 4,virtio-blk,/vms/vm1/hdd1.img vm1 &
```

Open up another terminal to this machine and run:

```bash
cu -l /dev/nmdm0B
```

Press `enter` again and you should see the installer screen. Take a breath, you successfully have booted your vm with your installer iso, go ahead and install like normally would, when you reboot the console will die, that's ok.

13) Kill the `bhyve` vm, to make sure everything is in a clean state.

```bash
bhyvectl --destroy --vm=vm1
```

14) Create a script (like `vm1_boot.sh`) to boot your machine from now on, take step 12 and edit it:

```bash
#!/bin/sh

grub-bhyve -r hd0,msdos1 -m /vms/vm1/device.map -M 4096 vm1

bhyve -c 1 -m 4096 -H -P -A \
-l com1,/dev/nmdm0A \
-s 0:0,hostbridge \
-s 1:0,lpc \
-s 2:0,virtio-net,tap1 \
-s 4,virtio-blk,/vms/vm1/hdd1.img vm1 &
```

Create a script to stop also (like `vm1_stop.sh`):

```bash
#!/bin/sh

bhyvectl --destroy --vm=vm1
```

15) Go ahead and `chmod +x` the previous scripts and run your startup script. If you closed your
serial port you can open it up again too:

```bash
./vm1_boot.sh
cu -l /dev/nmdm0B # optional if you closed it
```

Check for your IP, and you can ssh into your machine now! (if you need a GUI install `xrdp` seems to work)

16) How to set network devices to auto boot. Open up your `rc.conf` file on your host machine and add:

```bash
##### byhve settings
cloned_interfaces="bridge0 tap1"
ifconfig_bridge0="addm re0 addm tap1 up"
```

If you would like an auto for the vms, you will need to add startup script for the `vm1_boot.sh`.
