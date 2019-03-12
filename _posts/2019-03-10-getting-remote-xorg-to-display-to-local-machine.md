---
layout: post
title: "Getting remote xorg to display to local machine"
date: 2019-03-10 15:12:59
categories: sysadmin linux
---

I was talking to some coworkers recently. Working at IBM I have access to a cloud
that allows me to us it for remote workloads. We started talking about getting [xorg][xorg]
working on our remote machine and connect via X to my local laptop.

These are my notes to getting this working with a CentOS remote machine with a Ubuntu laptop.

1) Spin up a remote machine with CentOS installed, I have tested this with CentOS 7 server
and Ubuntu 18.04 laptop.

2) Confirm you can SSH to your remote machine, if you remote in as `root` I suggest creating
a user and giving them `sudo` access. I suggest reading [this link][sudo] if you have never
set it up.

3) `SSH` as your user with the following command:
```bash
ssh -X username@your_remote_machine
```

3.1) Now you might see this error, if so you should fix it by the following:
```bash
$ ssh -X username@server
X11 forwarding request failed on channel 0
[username@server]$
```

You need to edit your `sshd` configuration:
```bash
[username@server]$ sudo vi /etc/ssh/sshd_config
# Note if you don't have sudo as your username ssh in as someone who can, or as root if you have access
```

Uncomment the following lines:
```
X11Forwarding yes
X11UseLocalhost no
```

And restart `sshd`:
```bash
[username@server]$ sudo service sshd reload
Redirecting to /bin/systemctl reload sshd.service
```

3.2) If you still see the error, make sure you have Xorg installed, if not run this command:
```bash
sudo yum groupinstall "X Window System"
```

4) Now if you don't have `xauth` installed you'll see this error:
```bash
ssh -X username@server
Last login: Tue Mar 12 11:27:17 2019 from localmachine
/usr/bin/xauth:  file /home/username/.Xauthority does not exist
```
Finally Go ahead and install `dbus-x11`, `xauth` and `xeyes` via this:
```bash
$ sudo yum install xauth xeyes dbus-x11
```

5) Go ahead and logout/exit from the machine after your setup, and `ssh` back in with the
following command and use `xeyes` to validate your set up. Run `Control-C` to close
`xeyes` (which is running on the remote machine but displayed locally).
```bash
ssh -X username@server
Last login: Tue Mar 12 12:09:32 2019 from localmachine
[username@server]$ xeyes
```

Congratulations! You now have a way to run remote X applications and display them
on your local machine. Go ahead install something like `firefox` or `chrome`
and go to <https://whatsmyip.org> and then open it on your local machine,
it should be radically different!


[sudo]: https://linuxize.com/post/create-a-sudo-user-on-centos/
[xorg]: https://www.x.org/wiki/
