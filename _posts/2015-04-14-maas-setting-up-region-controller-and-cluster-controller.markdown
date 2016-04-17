---
layout: post
title: "maas setting up region-controller and cluster-controller"
date: 2015-04-14 11:30:25 -0500
comments: true
categories: maas sysadmin
---

I attempted to set up a `region-controller` inside of MAAS recently. It was a disaster;
I ended up breaking my whole environment. Luckly it was a demo environment, so I scratched
it and started from square one again. This post is how I got it working, most of this is
out of the internet, but this is a distilled version so you can just see what you need to
do.

First thing first, on what you want your `region-controller` to be:

```bash
$ sudo cat /var/lib/maas/secret
```

This'll give you a 16 digit hex number that you need to copy for any `cluster-controller` that you
want to connect to the `region-controller`.

After this, go to your `cluster-controller` and do the following:

```bash
$ sudo dpkg-reconfigure maas-cluster-controller # point it to the MAAS instance that is your future region-controller
$ sudo maas-provision install-shared-secret # it'll ask for the above secret you copy'd
```

If you've done everything correct you should see something like the following in the logs on the `cluster-controller`,
then check your `region-controller` and it should be under "Clusters."

```
$ sudo cat /var/log/maas/pserv.log
2015-04-14 11:27:30-0500 [ClusterClient,client] Event-loop 'vm-controller-maas:pid=1318' authenticated.
2015-04-14 11:27:30-0500 [ClusterClient,client] Event-loop 'vm-controller-maas:pid=1317' authenticated.
```

I rebooted my `cluster-controller` to make sure everything came up as expected, and saw this error is `/var/log/apache2/error.log`.

```
$ sudo cat /var/log/apache2/error.log
[Tue Apr 14 13:29:36.548078 2015] [:error] [pid 2766:tid 140219708958464] [remote 127.0.0.1:7825] AssertionError: The secret stored in the database does not match the secret stored on the filesystem at /var/lib/maas/secret. Please investigate.
```

I resolved it by:

```bash
$ sudo su - maas
$ psql maasdb
maasdb => select * from maasserver_config;
 id |       name        |               value
----+-------------------+------------------------------------
  1 | rpc_shared_secret | "05e2NOT_REALLY_MY_SECRET03d412547"
(1 row)
maasdb=> begin transaction ;
BEGIN
maasdb=> update maasserver_config set value = '83f247fMY_NEW_VALUEcf00b2d1120aa' where id = 1;
UPDATE 1
maasdb=> end
maasdb-> ;
COMMIT
maasdb=> select * from maasserver_config ;
 id |       name        |              value
----+-------------------+----------------------------------
  1 | rpc_shared_secret | 83f247fMY_NEW_VALUEcf00b2d11280aa
(1 row)

maasdb=>
```

Hopefully the `installed-shared-secret` will update the database in the future too, so you don't have to hack it like this.
