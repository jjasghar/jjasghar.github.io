---
layout: post
title: "Getting Fedora 30 setup to do LFS"
date: 2019-08-14 19:17:15
categories: linux sysadmin
---

I've decided to give LFS another shot. I booted into the Fedora 30 live cd as my working
OS, and mounted a drive. Here are some notes of things I had to do along the way to
get it to work.

First I had to install a few packages with `dnf`:

```bash
$ sudo dnf groupinstall "Development Tools"
$ sudo dnf install gcc-c++ texinfo m4 bison flex-devel bison-devel byacc
```

Then the `version-check.sh` script passed.

```
$ bash version-check.sh
bash, version 5.0.2(1)-release
/bin/sh -> /usr/bin/bash
Binutils: version 2.31.1-29.fc30
bison (GNU Bison) 3.0.5
Usage: /usr/bin/yacc [options] filename

Options:
  -b file_prefix        set filename prefix (default "y.")
  -B                    create a backtracking parser
  -d                    write definitions (.tab.h)
  -i                    write interface (y.tab.i)
  -g                    write a graphical description
  -l                    suppress #line directives
  -L                    enable position processing, e.g., "%locations"
  -o output_file        (default ".tab.c")
  -p symbol_prefix      set symbol prefix (default "yy")
  -P                    create a reentrant parser, e.g., "%pure-parser"
  -r                    produce separate code and table files (y.code.c)
  -s                    suppress #define's for quoted names in %token lines
  -t                    add debugging support
  -v                    write description (y.output)
  -V                    show version information and exit
yacc is
bzip2,  Version 1.0.6, 6-Sept-2010.
Coreutils:  8.31
diff (GNU diffutils) 3.7
find (GNU findutils) 4.6.0
GNU Awk 4.2.1, API: 2.0 (GNU MPFR 3.1.6-p2, GNU MP 6.1.2)
/usr/bin/awk -> /usr/bin/gawk
gcc (GCC) 9.1.1 20190503 (Red Hat 9.1.1-1)
g++ (GCC) 9.1.1 20190503 (Red Hat 9.1.1-1)
(GNU libc) 2.29
grep (GNU grep) 3.1
gzip 1.9
Linux version 5.0.9-301.fc30.x86_64 (mockbuild@bkernel04.phx2.fedoraproject.org) (gcc version 9.0.1 20190312 (Red Hat 9.0.1-0.10) (GCC)) #1 SMP Tue Apr 23 23:57:35 UTC 2019
m4 (GNU M4) 1.4.18
GNU Make 4.2.1
GNU patch 2.7.6
Perl version='5.28.1';
Python 3.7.3
sed (GNU sed) 4.5
tar (GNU tar) 1.32
texi2any (GNU texinfo) 6.6
xz (XZ Utils) 5.2.4
g++ compilation OK
```

I should mention that the `yacc` application doesn't support the `--version` flag, and uses
the `-V` now instead. So but if you fix the `if then` it comes out like:

```bash
$ echo yacc is `/usr/bin/yacc -V | head -n1`
yacc is /usr/bin/yacc - 1.9 20170709
```
