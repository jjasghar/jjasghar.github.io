---
layout: post
title: "Using ENV variables to change DCs"
date: 2017-05-30 13:35:32
categories: sysadmin chef vmware bash
---

Recently there has been a question about dealing with multiple VMware vCenters
and our Chef plugins. Unlike AWS or AZURE, vCenter is (normally) deployed per
Datacenter to allow for local control over your vSphere infrastructure.

(Yes, yes, I know you can make a super set vCenter instance, but more people I talk to,
it's just easier to connect to the DC you want to administer.)

Unfortunately, as you start using `knife-vsphere` or `knife-vrealize` you may need
to flip back and forth between DCs. There are plugins like [knife-block][block], which
I have even used in the past, but some places don't allow for extra plugins. This
is a suggested way to do it purely through Ruby, Chef and your Shell.

First thing first, you need to edit your `knife.rb` to allow for the ability to switch,
take this fake example:

```ruby
knife[:ssh_user] = "root"
knife[:vsphere_insecure] = true
knife[:vsphere_host] = "central.tirefi.re"
knife[:vsphere_dc] = "Datacenter"
knife[:vsphere_user] = "administrator@vsphere.local"

if ENV['DC'] == 'EAST'
  knife[:vsphere_host] = "east.tirefi.re"
  knife[:vsphere_dc] = "Datacenter"
end

if ENV['DC'] == 'WEST'
  knife[:vsphere_host] = "west.tirefi.re"
  knife[:vsphere_dc] = "Datacenter"
end
```

Now most of this should make some sense. You can pretty much assume I have 3 Datacenters, `central`
`east` and `west`. My default datacenter, the one I connect to most of the time is `central`.

Now, lets say I did a `knife vsphere vm list`:

```shell
14:16:20 JJs-MacBook-Pro ~ > knife vsphere vm list
Folder:
	VM Name: jenkins
		IP:
		RAM: 4096
		State: off

[-- snip --]

	VM Name: pfsense
		IP:
		RAM: 4024
		State: on
```

So I connect out to `centeral`s vCenter instance, and get back by Default information.

Ok, but my boss wants to see what's in my `east` datacenter. This is where [Environment variables][wiki]
come into play. With the changes to the `knife.rb` above, you can "flip" with just giving the Shell
command an override.

Take for instance for `east`:

```shell
14:20:20 JJs-MacBook-Pro ~ > DC=EAST knife vsphere vm list
Folder:
	VM Name: WIN-AD-EAST
		IP:
		RAM: 8096
		State: on

[-- snip --]

	VM Name: QuakeIIServer
		IP:
		RAM: 4024
		State: on
```

or in the case of `west`:

```shell
14:23:20 JJs-MacBook-Pro ~ > DC=WEST knife vsphere vm list
Folder:
	VM Name: WIN-AD-WEST
		IP:
		RAM: 8096
		State: on

[-- snip --]

	VM Name: TF2
		IP:
		RAM: 4024
		State: on
```

Notice the `DC=EAST` and `DC=WEST`, which "flips" the location of the command endpoint. Just to be clear,
this works with any of the `knife` commands, `vm clone` etc, so you can change it via an option instead
of editing/copying `knife.rb`s!

[block]: https://github.com/knife-block/knife-block
[wiki]: https://en.wikipedia.org/wiki/Environment_variable
