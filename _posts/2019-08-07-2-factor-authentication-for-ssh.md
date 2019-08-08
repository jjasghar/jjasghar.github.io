---
layout: post
title: "2 factor authentication for SSH"
date: 2019-08-07 19:47:44
categories: linux sysadmin
---

I recently got some access to some impressive hardware. I put in on the internet
and no joke less then 24 hours someone had already gotten in. Honestly, I did
set the `ssh` password to `P@ssw0rd!` but still, less then 24 hours? Come on
script kiddies, give me at least 24 hours to set up. pppff whatever.

So I decided that I need to lock down my internet facing machines. First, I had
to make some changes to the main `/etc/ssh/sshd_config` file to lock down the machine.

I changed the file to have:

```
PermitRootLogin no
PasswordAuthentication no
```

And copied my main `id_rsa.pub` to the `authorized_keys` of my user on the box. I enabled
`sudo` for it and now I feel hella safer.

But that isn't what this post is about, it's about those times I need to knock on the front
door and get access to this machine. So, lets get 2 factor authentication working.

There's a handful of options and Open Source projects to do 2fa, but I've decided to try
using the `libpam-google-authenticator` which you can find the code [here][googlecode].

> Before I go any farther, this post is a voltron like walk through from [this post][first],
> [this post][second], and [this post][third]. With out their work I would have never gotten
> this to work. Thank you.

It seems that every user has to run the authentication software in order to get the keys created,
there is a way to use something like Chef or Ansible for automation, but that'll be at the
end of this tutorial. For now lets just get it working with one user (admini).

I have a brand new machine up and running:

```bash
admini@debian-template:~$
```

First thing first, gotta install the software. I'm on Debian, but you however you install your
packages do something like:

```bash
admini@debian-template:~$ sudo apt-get update \
    && sudo apt-get install libpam-google-authenticator
```

Hopefully no errors, so we now have the application installed. Next lets run the application:

```bash
admini@debian-template:~$ google-authenticator

Do you want authentication tokens to be time-based (y/n) y
Warning: pasting the following URL into your browser exposes the OTP secret to Google:
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/admini@debian-template%3Fsecret%3DKQ35JFEFTLDB3TA7QZILK4N7UE%26issuer%3Ddebian-template

[-- A QR CODE --]

Your new secret key is: KQ35JFEFTLDB3TA7QZILK4N7UE
Your verification code is 994945
Your emergency scratch codes are:
  98406167
  49922485
  77014685
  43482243
  47556507

Do you want me to update your "/home/admini/.google_authenticator" file? (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) y

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```

Now you want to say `y` to the first one, otherwise the application won't work.
Second question is just good practice _you_want_ the passwords to expire
every 30 seconds. Save the emergency scratch codes in a trusted location.
If you can't take a picture of the qr code, then use the `secret key` and `verification code`
by hand. You should be able to take a picture I'd imagine though.
Then the extra questions: you shouldn't share your tokens so `y` is a good option.
Same with the third and forth questions are just good practice, so answer `y` on each
too.

Congrats! You've successfully configured the application. Now lets link it
up to `ssh`.

Now we need to edit the `/etc/pam.d/sshd` file, you'll need to be `root` so
`sudo su -` up to it, or however you get there.

```bash
admini@debian-template:~$ sudo su -
root@debian-template:~# vi /etc/pam.d/sshd
```

Go ahead and add this to the top of the file:

```
auth required pam_google_authenticator.so
```

Also comment out, if you don't want a password check:

```
#@include common-auth
```

Write and quit the file and open up:

```bash
root@debian-template:~# vi /etc/ssh/sshd_config
```

Change the following line to:

```
ChallengeResponseAuthentication yes
```

And add this to the file:

```
AuthenticationMethods publickey keyboard-interactive
```

If you want to be really secure you can add a `,` between it to make sure that you have
BOTH your `id_rsa.pub` on the account and your 2fa enabled. I'd only do this after you've
got your `publickey` on the box otherwise you'll get errors attempting to reconnect.

Example:

```
AuthenticationMethods publickey,keyboard-interactive
```

Validate your `/etc/ssh/sshd_config` via:

```bash
root@debian-template:~# sshd -t
```

If any errors appeared go ahead and fix them NOW. If not, restart `sshd`!

```bash
root@debian-template:~# systemctl reload sshd.service
```

Go ahead and logout and log in, and you should see something like:

```bash
~ > ssh admini@172.16.0.87
Verification code:
Linux debian-template 4.19.0-5-amd64 #1 SMP Debian 4.19.37-5 (2019-06-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Aug  7 19:39:27 2019 from 169.44.150.34
admini@debian-template:~$
```

Congratulations! You now have 2fa working on your machine.

## Bonus!

If you would like to automate this with a Chef Cookbook or Ansible playbook, you can!

You can run all the questions via command line, if you run `google-authenticator --help`
you can see all your options:

```bash
admini@debian-template:~$ google-authenticator --help
google-authenticator [<options>]
 -h, --help                     Print this message
 -c, --counter-based            Set up counter-based (HOTP) verification
 -t, --time-based               Set up time-based (TOTP) verification
 -d, --disallow-reuse           Disallow reuse of previously used TOTP tokens
 -D, --allow-reuse              Allow reuse of previously used TOTP tokens
 -f, --force                    Write file without first confirming with user
 -l, --label=<label>            Override the default label in "otpauth://" URL
 -i, --issuer=<issuer>          Override the default issuer in "otpauth://" URL
 -q, --quiet                    Quiet mode
 -Q, --qr-mode={NONE,ANSI,UTF8} QRCode output mode
 -r, --rate-limit=N             Limit logins to N per every M seconds
 -R, --rate-time=M              Limit logins to N per every M seconds
 -u, --no-rate-limit            Disable rate-limiting
 -s, --secret=<file>            Specify a non-standard file location
 -S, --step-size=S              Set interval between token refreshes
 -w, --window-size=W            Set window of concurrently valid codes
 -W, --minimal-window           Disable window of concurrently valid codes
 -e, --emergency-codes=N        Number of emergency codes to generate
```

You'd have to add this for each of your users on the machine, it is possible to
automate the setup :) .

[googlecode]: https://github.com/google/google-authenticator-libpam
[first]: https://bash-prompt.net/guides/ssh-2fa/
[second]: https://hackertarget.com/ssh-two-factor-google-authenticator/
[third]: https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04
