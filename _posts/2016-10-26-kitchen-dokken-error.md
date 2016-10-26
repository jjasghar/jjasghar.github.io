---
layout: post
title: "rpc error: code = 13 desc = invalid header field"
date: 2016-10-26 14:31:52
categories: chef linux docker
---

I was attempting to set up [kitchen-dokken][dokken] and ran into the following
error:

```bash
rpc error: code = 13 desc = invalid header field value "oci runtime error: exec
failed: container_linux.go:247: starting container process caused \"exec:
\\\"/opt/chef/embedded/bin/chef-client\\\": stat /opt/chef/embedded/bin/chef-client:
no such file or directory\"\n"
```

Talking to my cohort [Kevin Reedy][kevin] answered this question for me. "Hey,
do a `docker ps --all | grep chef`."

```bash
14:28:41 JJs-MacBook-Pro ~ > docker ps --all | grep chef
65915b812974        chef/chef:latest                                   "true"                   47 hours ago        Created                                              chef-latest
f98d35ffa390        jjasghar/chef-partner-cookbook-guide               "/usr/bin/supervisord"   2 weeks ago         Exited (0) 11 days ago                               adoring_agnesi
5667af98f6a0        jjasghar/chef-partner-cookbook-guide               "/usr/bin/supervisord"   2 weeks ago         Exited (0) 2 weeks ago                               jovial_hamilton
733d63f956af        jjasghar/chef-partner-cookbook-guide               "-d"                     2 weeks ago         Created                     0.0.0.0:1948->1948/tcp   gigantic_roentgen
72d88591d5ca        jjasghar/chef-partner-cookbook-guide               "/usr/bin/supervisord"   2 weeks ago         Exited (0) 2 weeks ago                               determined_colden
b62986c336ef        38cbf3c1d9e8                                       "/usr/bin/supervisord"   9 weeks ago         Exited (0) 8 weeks ago                               chef-partner-cookbook-guide
```

Kevin took a look and said: "Ah ok, go ahead and: `docker rm --force 65915b812974`, and you should re-pull
down that container."

```bash
14:29:18 JJs-MacBook-Pro rabbitmq > (master) chef exec kitchen verify default-centos-67
-----> Starting Kitchen (v1.11.1)
-----> Creating <default-centos-67>...
       Finished creating <default-centos-67> (1m11.63s).
-----> Converging <default-centos-67>...
       Preparing files for transfer
       Preparing dna.json
       Resolving cookbook dependencies with Berkshelf 4.3.5...
       Removing non-cookbook files before transfer
       Preparing validation.pem
       Preparing client.rb
       Transferring files to <default-centos-67>
Starting Chef Client, version 12.15.19
Creating a new client identity for default-centos-67 using the validator key.
resolving cookbooks for run list: ["rabbitmq::default"]
Synchronizing Cookbooks:
  - rabbitmq (4.10.0)
  - dpkg_autostart (0.2.0)

[-- snip --]

```

It seems that the my `chef/chef:latest` was in a broken state, and removing and
re-pulling the container was the fix.


[dokken]: https://github.com/someara/kitchen-dokken
[kevin]: https://twitter.com/kevinreedy
