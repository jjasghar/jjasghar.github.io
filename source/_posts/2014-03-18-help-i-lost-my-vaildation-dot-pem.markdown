---
layout: post
title: "Help! I lost my vaildation.pem"
date: 2014-03-18 16:39:04 -0500
comments: true
categories: chef sysadmin 
---

So I recently moved chef servers. I had a handful of hiccups on the way. The main one was my `validation.pem` and my `chef-webui.pem`...went missing. I had to regenerate my vaildation.pem without my chef-webui. These commands are on open source
chef 11 server.

(OK, fine I broke them, but this post is how to recreate either with `knife` on your workstation incase **cough**like me**cough** you have lost your chef-webui.)

It turns out there is a great simple command to re-create your validation.pem. 

Note: This can also work as `validator.pem` also, but my code checks for `validation.pem`

```bash
[~] % knife client reregister chef-validator
# or if you want to name it and save it....
[~] % knife client reregister chef-validator -f validaton.pem
```

The first one will spit out the new `.pem` you'll need to copy it to a file otherwise you'll just have to do run the command again. This/these commands are the equivalent of the `https://<chefserver>/clients/chef-validator/edit` and clicking that
"Regenerate Private Key (Existing one will no longer work!)."

Pretty straight forward eh?

On the other hand, if you've broken you chef-webui, and you see something like....

```
2014-03-18_21:31:54.98838 Chef::Exceptions::PrivateKeyMissing: I cannot read /etc/chef-server/chef-webui.pem, which you told me to use to sign requests!
2014-03-18_21:31:54.98840 {:request_params=>
2014-03-18_21:31:54.98840   {"utf8"=>"✓",
2014-03-18_21:31:54.98841    "authenticity_token"=>"uSheCVhYuGJPBAyDBHb4AIyEfkB2EqXwLD6Uolk//ig=",
2014-03-18_21:31:54.98841    "name"=>"admin",
2014-03-18_21:31:54.98841    "commit"=>"login",
2014-03-18_21:31:54.98842    "password"=>"p@ssw0rd1",
2014-03-18_21:31:54.98842    "action"=>"login_exec",
2014-03-18_21:31:54.98842    "controller"=>"users"}}
```

In your `/var/log/chef-server/chef-server-webui/current` then the fix is pretty straight forward:

```bash
[~] % knife client reregister chef-webui
[~] % #scp it up to your chef box
chef# chown root.root /etc/chef-server/chef-webui.pem
```


