---
layout: post
title: "Getting Debian 10 setup to do LFS"
date: 2019-08-19 18:30:47
categories: linux sysadmin
---

So I attempted to get Fedora 30 to be my base OS for
Linux From Scratch (LFS). It seems I broke something
and the versioning of the system was different. Instead
of debugging it, I've decided to move to Debian 10 as my
base OS. Here are my notes and what I did to get the
host OS to work.

In a blank installation with the [live-cd standard](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-10.0.0-amd64-standard.iso) the `version-check.sh` really doesn't pass.

```bash
root@debian:~# bash version-check.sh
bash, version 5.0.3(1)-release
/bin/sh -> /usr/bin/dash
ERROR: /bin/sh does not point to bash
Binutils: (GNU Binutils for Debian) 2.31.1
version-check.sh: line 10: bison: command not found
yacc not found
bzip2,  Version 1.0.6, 6-Sept-2010.
Coreutils:  8.30
diff (GNU diffutils) 3.7
find (GNU findutils) 4.6.0.225-235f
version-check.sh: line 22: gawk: command not found
/usr/bin/awk -> /usr/bin/mawk
gcc (Debian 8.3.0-6) 8.3.0
g++ (Debian 8.3.0-6) 8.3.0
(Debian GLIBC 2.28-10) 2.28
grep (GNU grep) 3.3
gzip 1.9
Linux version 4.19.0-5-amd64 (debian-kernel@lists.debian.org) (gcc version 8.3.0 (Debian 8.3.0-7)) #1 SMP Debian 4.19.37-5 (2019-06-19)
version-check.sh: line 36: m4: command not found
GNU Make 4.2.1
GNU patch 2.7.6
Perl version='5.28.1';
Python 3.7.3
sed (GNU sed) 4.7
tar (GNU tar) 1.30
version-check.sh: line 43: makeinfo: command not found
xz (XZ Utils) 5.2.4
g++ compilation OK
```

- First thing to fix, is to remove and rewire up `/bin/sh` to bash:

```bash
root@debian:~# ls -l /bin/sh
lrwxrwxrwx 1 root root 4 Jul  6 10:25 /bin/sh -> dash
root@debian:~# rm /bin/sh
root@debian:~# ln -s /usr/bin/bash /bin/sh
root@debian:~# ls -l /bin/sh
lrwxrwxrwx 1 root root 13 Aug 19 23:46 /bin/sh -> /usr/bin/bash
```

- Next, install some packages:

```bash
root@debian:~# apt-get install bison flex gawk m4 texinfo -y
```

- And looks like everything in `version-check.sh` passes!

```bash
root@debian:~# bash version-check.sh
bash, version 5.0.3(1)-release
/bin/sh -> /usr/bin/bash
Binutils: (GNU Binutils for Debian) 2.31.1
bison (GNU Bison) 3.3.2
/usr/bin/yacc -> /usr/bin/bison.yacc
bzip2,  Version 1.0.6, 6-Sept-2010.
Coreutils:  8.30
diff (GNU diffutils) 3.7
find (GNU findutils) 4.6.0.225-235f
GNU Awk 4.2.1, API: 2.0 (GNU MPFR 4.0.2, GNU MP 6.1.2)
/usr/bin/awk -> /usr/bin/gawk
gcc (Debian 8.3.0-6) 8.3.0
g++ (Debian 8.3.0-6) 8.3.0
(Debian GLIBC 2.28-10) 2.28
grep (GNU grep) 3.3
gzip 1.9
Linux version 4.19.0-5-amd64 (debian-kernel@lists.debian.org) (gcc version 8.3.0 (Debian 8.3.0-7)) #1 SMP Debian 4.19.37-5 (2019-06-19)
m4 (GNU M4) 1.4.18
GNU Make 4.2.1
GNU patch 2.7.6
Perl version='5.28.1';
Python 3.7.3
sed (GNU sed) 4.7
tar (GNU tar) 1.30
texi2any (GNU texinfo) 6.5
xz (XZ Utils) 5.2.4
g++ compilation OK
```

That should be it to get you ready to start playing with
Linux From Scratch!
