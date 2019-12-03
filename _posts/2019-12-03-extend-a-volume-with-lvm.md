---
layout: post
title: "Extend a volume with LVM"
date: 2019-12-03 15:31:24
categories: sysadmin linux fedora
---


I had to look this up, so here are my notes. I'm slowly converting to Fedora as my
work OS, and coming from the Debian world this has been a challenge. The first
thing was mounting my new hard drive. I just installed a 1 TB hard drive, installed
Fedora 31 server on it via the installation software.

I did the "auto-partition" option, thought nothing of it, then rebooted the machine.

I logged in, and the first thing I did was:
```bash
[root@skippy ~]# df -h
Filesystem                               Size  Used Avail Use% Mounted on
devtmpfs                                 3.8G     0  3.8G   0% /dev
tmpfs                                    3.8G     0  3.8G   0% /dev/shm
tmpfs                                    3.8G  1.1M  3.8G   1% /run
/dev/mapper/fedora_localhost--live-root   15G  1.9G   14G  13% /
tmpfs                                    3.8G  4.0K  3.8G   1% /tmp
/dev/sda1                               1014M  182M  833M  18% /boot
tmpfs                                    771M     0  771M   0% /run/user/0

```

Hang on, my `/` partion is only 14 GBs? Next I ran:

```bash
[root@skippy ~]# fdisk -l
Disk /dev/sda: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: WDC WD10EZEX-08W
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xa91a7392

Device     Boot   Start        End    Sectors   Size Id Type
/dev/sda1  *       2048    2099199    2097152     1G 83 Linux
/dev/sda2       2099200 1953523711 1951424512 930.5G 8e Linux LVM


Disk /dev/mapper/fedora_localhost--live-root: 15 GiB, 16106127360 bytes, 31457280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

OK, this is odd, the hardrive is 930 GBs but my disk which LVM is running is only mapped to 15 GBs.

So I need to extend it? How the hell do I do that? Well after some Googling, I found [this page](https://www.rootusers.com/lvm-resize-how-to-increase-an-lvm-partition/) and this is what I did.

```bash
[root@skippy ~]# lvdisplay

[-- snip --]

  --- Logical volume ---
  LV Path                /dev/fedora_localhost-live/root
  LV Name                root
  VG Name                fedora_localhost-live
  LV UUID                qdUJ8h-6NdK-hl9K-z6Mk-Vs5S-OK9m-dDMkOY
  LV Write Access        read/write
  LV Creation host, time skippy.asgharlabs.io, 2019-12-03 14:47:41 -0500
  LV Status              available
  # open                 1
  LV Size                15.00 GiB
  Current LE             3840
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

Verified that the `LV Size` was 15.00 GiB, then I extended it, first by 500 Gigs, then 400, because
I didn't know what was going to happen.

```bash
[root@skippy ~]# lvextend -L+500G /dev/fedora_localhost-live/root
  Size of logical volume fedora_localhost-live/root changed from 15.00 GiB (3840 extents) to 515.00 GiB (131840 extents).
  Logical volume fedora_localhost-live/root successfully resized.
[root@skippy ~]#
[root@skippy ~]# lvextend -L+400G /dev/fedora_localhost-live/root
  Size of logical volume fedora_localhost-live/root changed from 515.00 GiB (131840 extents) to 915.00 GiB (234240 extents).
  Logical volume fedora_localhost-live/root successfully resized.
```

OK, that's looking better, now I need to extend the actual file system.

```bash
[root@skippy ~]# df -h
Filesystem                               Size  Used Avail Use% Mounted on
devtmpfs                                 3.8G     0  3.8G   0% /dev
tmpfs                                    3.8G     0  3.8G   0% /dev/shm
tmpfs                                    3.8G  1.1M  3.8G   1% /run
/dev/mapper/fedora_localhost--live-root   15G  1.9G   14G  13% /
tmpfs                                    3.8G  4.0K  3.8G   1% /tmp
/dev/sda1                               1014M  182M  833M  18% /boot
tmpfs                                    771M     0  771M   0% /run/user/0
[root@skippy ~]# xfs_growfs /
meta-data=/dev/mapper/fedora_localhost--live-root isize=512    agcount=4, agsize=983040 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=3932160, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 3932160 to 239861760
```

It wanted the mount point, not the device, `/`.

And one final check:

```bash
[root@skippy ~]# df -h
Filesystem                               Size  Used Avail Use% Mounted on
devtmpfs                                 3.8G     0  3.8G   0% /dev
tmpfs                                    3.8G     0  3.8G   0% /dev/shm
tmpfs                                    3.8G  1.1M  3.8G   1% /run
/dev/mapper/fedora_localhost--live-root  915G  8.2G  907G   1% /
tmpfs                                    3.8G  4.0K  3.8G   1% /tmp
/dev/sda1                               1014M  182M  833M  18% /boot
tmpfs                                    771M     0  771M   0% /run/user/0
```

Sweet! I got all my space.
