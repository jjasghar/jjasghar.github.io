---
layout: post
title: "Update to the chef error that has haunted me"
date: 2014-03-19 16:45:03 -0500
comments: true
categories: chef sysadmin
---

So it seems that with release of chef-client `11.10` the 403 error have...morphed. Now they look  something like:

```
Chef::Exceptions::ContentLengthMismatch
---------------------------------------
Response body length 164 does not match HTTP Content-Length header 206.
```

I moved chef servers and didn't change my `s3_url_ttl` and ran into this issue. So the annoying 403's are now something about `HTTP Content-Length`.

If you've forgotten the fix is:
```bash
[~] % sudo vim /etc/chef-server/chef-server.rb
# add this line: erchef[‘s3_url_ttl’] = 900 where 900 is something larger...maybe 1800?
[~] % sudo chef-server-ctl reconfigure
```
