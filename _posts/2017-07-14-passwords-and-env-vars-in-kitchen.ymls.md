---
layout: post
title: "Passwords and ENV vars in kitchen.ymls"
date: 2017-07-14 16:26:34
categories: sysadmin chef kitchen ruby
---

As you are starting to standardize on using [test-kitchen][kitchen] the problem
of passwords and security start to appear. Yeah, using kitchen locally with vagrant
is great, but as you start using remote instances you need a way to pass passwords
to track things. Here's a quick example and use case.

There are a couple ways to skin this cat, but I'll take the ones that I like to
suggest.

First set up your `kitchen.yml` so it can take variables, for instance:

```yaml
driver:
  name: vsphere
  driver_options:
    host: vcenter.tirefi.re
    user: <%= ENV['VCENTER_USER'] || "administrator@vsphere.local" %>
    password: <%= ENV['VCENTER_PASSWORD'] || "P@ssw0rd!" %>
```
Awesome, this sets the default username to `administrator@vsphere.local` and
the default password to `P@ssw0rd!`. Obviously, you shouldn't use `administrator` but
hopefully you get the point. Now if you need to over ride it with your user name and
password `ENV` variables will do the trick! Here's an example:

```shell
~$ VCENTER_USER="jj@chef.io" VCENTER_PASSWORD="N0tmyP@ssw0rd" kitchen verify
```

Pretty neat eh? But man, that's gonna be pretty annoying to type out each time, so how I get around
it is setting up file to source each time I open a new shell and add this line to my `.bashrc` ->
`source '/Users/jjasghar/keys'`.

```text
export VCENTER_USER="jj@chef.io"
export VCENTER_PASSWORD="N0tmyP@ssw0rd"
```

When it comes to windows, setting up `ENV` variables is a tad bit tricker, but luckily [Xah Lee][xah], has set
up a great tutorial. [Windows Environment Variables Tutorial][envwin].

[envwin]: http://xahlee.info/mswin/env_var.html
[kitchen]: http://kitchen.ci/
[xah]: https://twitter.com/xah_lee
