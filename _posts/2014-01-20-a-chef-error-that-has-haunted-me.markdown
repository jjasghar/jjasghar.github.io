---
layout: post
title: "A chef error that has haunted me"
date: 2014-01-20 15:42:33 -0600
comments: true
categories: chef linux sysadmin
---

Ok, you may have seen this before:

```ruby
23.987.33.854   * cookbook_file[/etc/init.d/apache2] action create
23.987.33.854 ================================================================================
23.987.33.854 Error executing action `create` on resource 'cookbook_file[/etc/init.d/apache2]'
23.987.33.854 ================================================================================
23.987.33.854
23.987.33.854
23.987.33.854 Net::HTTPServerException
23.987.33.854 ------------------------
23.987.33.854 403 "Forbidden"
23.987.33.854
23.987.33.854
```

NOTE: Yes, that is a fake ip, and yes that server Exception isn't tied just to apache2.

If you have, you know my pain.  Turns out there is a [ticket](https://tickets.opscode.com/browse/CHEF-4253) on this and also a couple [blog posts](http://sanketdangi.com/post/50897416859/chef-errors-and-their-solutions) also.

The gist of this:
> This error is encountered when we have large chef recipes whose deployment time on clients is large than 15 minutes. In order to avoid this error, please increase “s3_url_ttl" value from 900 seconds to required time interval

So the fix is:

```bash
[~] % sudo vim /etc/chef-server/chef-server.rb
# add this line: erchef[‘s3_url_ttl’] = 900 where 900 is something larger...maybe 1800?
[~] % sudo chef-server-ctl reconfigure
```

Boom, you should be good now. No more damn 403s.

# Update!

So it seems that with release of `11.10` the 403 error has...morphed. Now it looks something like:

```ruby
Chef::Exceptions::ContentLengthMismatch
---------------------------------------
Response body length 164 does not match HTTP Content-Length header 206.
```

I have a post about it from 2014-03-19
