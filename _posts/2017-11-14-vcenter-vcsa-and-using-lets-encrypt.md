---
layout: post
title: "vCenter (VCSA) and using Let's Encrypt for SSL Certificates"
date: 2017-11-14 15:32:03
categories: sysadmin vmware
---

If you are using the [VCSA][vcsa] for your vCenter you might have searched
around to figure out how to update the certificate from Let's Encrypt. It seems that
throughout my Googling I personally wasn't able to find a tutorial so this is
mine. If you have suggestions or ideas I'd love to hear them, reach out via
twitter: [@jjasghar][twitter].

# Prerequisites

You need to set up [certbot][certbot] on your local machine. There are a few
ways to do that if you click that link please figure it out. Second, you
need the `root` login to your VCSA, with `ssh` turned on. You'll be running
some commands at the shell of the VCSA and if you can't get there you won't
be able to update your certificate. And finally you'll need some `administrator` privileges to your vCenter, defaulting to `administrator@vsphere.local`.

# Requesting the cert from Let's Encrypt

Whatever your domain name is, in order for Let's Encrypt to say that you
own the domain you'll need to add a `TXT` entry for the vCenter you are
getting the certificate for. For instance here is mine:

![](../../../../../pics/acme-challenge-vcenter.png)

Ok, assuming you have your DNS provider up, let's send the commands to Let's
Encrypt:

```shell
~ > sudo certbot certonly --manual --preferred-challenges=dns -d vcenter.tirefi.re
```

Notice the `sudo` you have to run this command with `root` privileges. This sends
the request and gives you a couple prompts, the most important being:

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.vcenter.tirefi.re with the following value:

tuS3NO-WAY-IMPUTTING34p2MY-ACTUAL32341KEY-HERE

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
```

Press the Enter key when you are sure the `TXT` DNS entry has propagated and you should see something like:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/vcenter.tirefi.re/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/vcenter.tirefi.re/privkey.pem
   Your cert will expire on 2018-02-12. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
```

Wonderful, now keep this terminal/window open you'll need it in a bit.

# Updating the SSL Certificate on your VCSA

Now that you have your files on your local machine, you'll need to get them
on your VCSA. There are a couple ways to do this, the easiest way I found was
to `cat` out the certificates and open up `vim` on the VCSA paste them in and save the files. You
can get `scp` or others working, but I didn't want to go through all that.

So here are my steps, first I `ssh` into my VCSA:

```
~ > ssh vcenter.tirefi.re -l root

VMware vCenter Server Appliance 6.5.0.10100

Type: vCenter Server with an embedded Platform Services Controller

root@vcenter.tirefi.re's password:
Last login: Tue Nov 14 20:55:38 2017 from 172.16.20.10
Connected to service

    * List APIs: "help api list"
    * List Plugins: "help pi list"
    * Launch BASH: "shell"

Command> shell
Shell access is granted to root
root@vcenter [ ~ ]#
```

I create the three files I'll need to update on the VCSA.

```
root@vcenter [ ~ ]# touch cert.pem
root@vcenter [ ~ ]# touch privkey.pem
root@vcenter [ ~ ]# touch fullchain.pem
```

Now I go to my other window and type:

```
~ > sudo ls -l /etc/letsencrypt/live/vcenter.tirefi.re/
total 40
-rw-r--r--  1 root  wheel  543 Nov 14 15:01 README
lrwxr-xr-x  1 root  wheel   41 Nov 14 15:01 cert.pem -> ../../archive/vcenter.tirefi.re/cert1.pem
lrwxr-xr-x  1 root  wheel   42 Nov 14 15:01 chain.pem -> ../../archive/vcenter.tirefi.re/chain1.pem
lrwxr-xr-x  1 root  wheel   46 Nov 14 15:01 fullchain.pem -> ../../archive/vcenter.tirefi.re/fullchain1.pem
lrwxr-xr-x  1 root  wheel   44 Nov 14 15:01 privkey.pem -> ../../archive/vcenter.tirefi.re/privkey1.pem
~ >
```

Notice the consistent naming convention here.

Now cat out each, like this:

```
~ > sudo cat /etc/letsencrypt/live/vcenter.tirefi.re/cert.pem
~ > sudo cat /etc/letsencrypt/live/vcenter.tirefi.re/privkey.pem
~ > sudo cat /etc/letsencrypt/live/vcenter.tirefi.re/fullchain.pem
```

From your local machine, and copy everything in each file to the window that
is your VCSA.

Now that have your three files on your VCSA lets get them inside your machine.

## certificate-manager

Go ahead and run this next command:

```
root@vcenter [ ~ ]# /usr/lib/vmware-vmca/bin/certificate-manager
```

You should see something like the following:

```
		 _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
		|                                                                     |
		|      *** Welcome to the vSphere 6.0 Certificate Manager  ***        |
		|                                                                     |
		|                   -- Select Operation --                            |
		|                                                                     |
		|      1. Replace Machine SSL certificate with Custom Certificate     |
		|                                                                     |
		|      2. Replace VMCA Root certificate with Custom Signing           |
		|         Certificate and replace all Certificates                    |
		|                                                                     |
		|      3. Replace Machine SSL certificate with VMCA Certificate       |
		|                                                                     |
		|      4. Regenerate a new VMCA Root Certificate and                  |
		|         replace all certificates                                    |
		|                                                                     |
		|      5. Replace Solution user certificates with                     |
		|         Custom Certificate                                          |
		|                                                                     |
		|      6. Replace Solution user certificates with VMCA certificates   |
		|                                                                     |
		|      7. Revert last performed operation by re-publishing old        |
		|         certificates                                                |
		|                                                                     |
		|      8. Reset all Certificates                                      |
		|_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _|
Note : Use Ctrl-D to exit.
Option[1 to 8]:
```

If you have an error or something doesn't show up, you aren't running
6.5 vCenter and you'll need to debug what's going on.

Luckily the rest of the commands to get the certificates updated isn't too
complicated:

First, select `1. Replace Machine SSL certificate with Custom Certificate`
to update the certificate:

```
Option[1 to 8]: 1
```

It will prompt you for your `administrator` level privilege to update the
certificate, and the next option:

```
Please provide valid SSO and VC privileged user credential to perform certificate operations.
Enter username [Administrator@vsphere.local]:
Enter password:
	 1. Generate Certificate Signing Request(s) and Key(s) for Machine SSL certificate

	 2. Import custom certificate(s) and key(s) to replace existing Machine SSL certificate

Option [1 or 2]: 2
```

We want to import the custom certificate so select `2` as I did above.

Fill out the next with the suggestions we walked through at the beginning of
this post:

```shell
Please provide valid custom certificate for Machine SSL.
File : /root/cert.pem

Please provide valid custom key for Machine SSL.
File : /root/privkey.pem
```

The next option is the one that was where the trick of this whole thing is,
vCenter asks for the `signing certificate of the Machine SSL certificate`
where if you [google around][googling] you'll only ever see references to vCenter and
not what it actually means. Luckily, Let's Encrypt puts this in the
`fullchain.pem` so that's all you have to add:

```shell
Please provide the signing certificate of the Machine SSL certificate
File : /root/fullchain.pem

You are going to replace Machine SSL cert using custom cert
Continue operation : Option[Y/N] ? : Y
Command Output: /root/cert.pem: OK

Get site nameCompleted [Replacing Machine SSL Cert...]
default-site
Lookup all services
```

The final option is the confirmation you'll like to replace the Machine
SSL cert, and select `Y`.

A ton of UUIDs and data will stream by and it may take up to 10-15 minutes, but when you see this:

```
Updated 29 service(s)
Status : 70% Completed [stopping services...]
Status : 100% Completed [All tasks completed successfully]
```

You have successfully updated your Certificate!

![](../../../../../pics/green-vcenter.png)

[certbot]: https://certbot.eff.org/
[googling]: https://www.google.com/search?q=signing+certificate+of+the+Machine+SSL+certificate
[twitter]: https://twitter.com/jjasghar
[vcsa]: https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vcsa.doc/GUID-223C2821-BD98-4C7A-936B-7DBE96291BA4.html?src=af_5b804d3334401&cid=70134000001YXKx
