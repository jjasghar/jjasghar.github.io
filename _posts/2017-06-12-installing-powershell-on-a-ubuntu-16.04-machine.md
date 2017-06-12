---
layout: post
title: "Installing powershell on a Ubuntu 16.04 machine"
date: 2017-06-12 18:54:19
categories: linux powershell ubuntu sysadmin
---

I google'd for this, and it seems that it's pretty straight forward.

```shell
$ curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
$ curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | sudo tee /etc/apt/sources.list.d/microsoft.list
$ sudo apt-get update
$ sudo apt-get install -y powershell
powershell
$ powershell
PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS /home/jj>
```
