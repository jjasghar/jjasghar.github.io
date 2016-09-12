---
layout: post
title: "How do I create a Mac USB boot disk"
date: 2015-01-07 08:23:55 -0800
comments: true
categories: sysadmin mac
---

I've been installing Linux from my laptop on hardware for a while now. I can
never remember how to do it, so I keep googling for [this](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx).
Being I like distilling information down to the more important parts, here is my
"generic" way of doing what is described in that link:

```bash
~$ wget http://some_iso_on_some_site/image.iso
~$ hdiutil convert -format UDRW -o ~/path/to/target.img ~/path/to/image.iso
~$ mv ~/path/to/target.img.dmg  ~/path/to/target.img
~$ # ideally you already have your flash drive inserted
~$ diskutil list
~$ diskutil unmountDisk /dev/disk<the_number_of_the_disk>
~$ sudo dd if=/path/to/target.img of=/dev/rdisk<the_number_of_the_disk> bs=1m
~$ diskutil eject /dev/disk<the_number_of_the_disk>
```

Maybe I should script this...
