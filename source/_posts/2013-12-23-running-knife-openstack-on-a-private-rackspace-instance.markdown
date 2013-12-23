---
layout: post
title: "running knife-openstack on a private rackspace instance"
date: 2013-12-23 10:36
comments: true
categories: linux ruby sysadmin chef
---

So congrats you got your new "[Private Cloud](http://www.rackspace.com/cloud/private/)" from Rackspace. You are probably ecstatic to start building your new machines.
I know I was; but alas with all new toys sometimes you hit a couple stags. Here are a couple things I ran into on my first few days.

### First issue
So I'm a chef shop, as you might now by now (assuming you've read any of my other posts). I've used [knife-rackspace](https://github.com/opscode/knife-rackspace) tons of times; and hell I even have a [commit bit](https://github.com/opscode/knife-rackspace/commits?author=jjasghar). So logically I thought I could leverage this same gem with different backend api points. Nope, I was hard core wrong.
You end up having to install [knife-openstack](https://github.com/opscode/knife-openstack). That in itself isn't bad at all...
```bash
[~] % gem install knife-openstack
```
Now you need to update your `knife.rb`
```bash
[~] % vim ~/.chef/knife.rb
```
In your handoff ticket, you probably got something that looks like this:
```bash
export OS_USERNAME=Im_awesome_admin
export OS_PASSWORD=$omeCr@zyA$$passwD
export OS_TENANT_NAME=MyCompanyName
export OS_AUTH_URL=http://10.219.0.254:5000/v2.0/
export OS_AUTH_STRATEGY=keystone:
```
Go ahead and copy them out to what they need to be, something like...
```ruby
knife[:openstack_username] = "Your OpenStack Dashboard username"
knife[:openstack_password] = "Your OpenStack Dashboard password"
### Note: If you are not proxying HTTPS to the OpenStack auth port, the scheme should be HTTP
knife[:openstack_auth_url] = "http://cloud.mycompany.com:5000/v2.0/tokens"
knife[:openstack_tenant] = "Your OpenStack tenant name"
knife[:openstack_ssh_key_id] = "my sshkey id"
```
Great! So run that great command `knife openstack flavor list` to see if everything works....
```bash
[~] % knife openstack server list
ERROR: knife encountered an unexpected error
This may be a bug in the 'openstack server list' knife command or plugin
Please collect the output of this command with the `-VV` option before filing a bug report.
Exception: NoMethodError: undefined method `[]' for nil:NilClass
```
Crap..

Ok, lets try out with `-VV`

```ruby
DEBUG: openstack_username Im_awsome_admin
DEBUG: openstack_auth_url http://10.219.0.254:5000/v2.0/
DEBUG: openstack_tenant MyCompanyName
DEBUG: openstack_insecure 
/Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/knife-openstack-0.8.1/lib/chef/knife/openstack_flavor_list.rb:51:in `rescue in run': undefined method `[]' for nil:NilClass (NoMethodError)
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/knife-openstack-0.8.1/lib/chef/knife/openstack_flavor_list.rb:41:in `run'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/chef-11.8.0/lib/chef/knife.rb:485:in `run_with_pretty_exceptions'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/chef-11.8.0/lib/chef/knife.rb:174:in `run'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/chef-11.8.0/lib/chef/application/knife.rb:133:in `run'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/chef-11.8.0/bin/knife:25:in `<top (required)>'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/bin/knife:23:in `load'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/bin/knife:23:in `<main>'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/bin/ruby_executable_hooks:15:in `eval'
	from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/bin/ruby_executable_hooks:15:in `<main>'
```

Well that's not a lot of help eh? Turns out, if you look at the ticket that Rackspace gives you and what the `[:openstack_auth_url]` requires you'll see that there's a `/tokens` at the end. Do'h!

### Second issue

Ok, so you got the ability to talk to your backend? Yay! But alas, you run your create...
```bash
[~] % knife openstack server create -S jj-mba-key -I 349168d3-5381-4324-8636-398d012f852b -f 1 -N testbox
Instance Name: testbox
Instance ID: 5e0ec79c-e06a-4fdb-9887-2b30ae1e5f80

Waiting for server.........
Flavor: 1
Image: 349168d3-5381-4324-8636-398d012f852b
SSH Keypair: jj-mba-key
ERROR: No IP address available for bootstrapping.
```
What the hell does that mean? Well I'm not going to explain it all but it seems that by default Rackspace names the "public" and "private" networks as "Fixed" and "Floating."
This is triggered a fog issue, where it's looking at the label for a network either "public" or "private" and blows up. There is a ticket in for this [here](https://tickets.opscode.com/browse/KNIFE-231) but it looks like it's stalled from late summer, early fall. Lammmeeee.

So you are probably saying "Why don't you just rename them?" Good for you, great idea...but no, Openstack doesn't support that. So at this time, it looks like you'll have to delete them and rebuild them with the "public" and "private" names. Hopefully you've noticed this at just the begining of building out your machines, otherwise you'll have to nuke and pave everything you've done to get the new networks in.


Ah!, almost forgot. Before you go I should mention a quick note, notice the lowercase p in both public and private. Yes, it's THAT picky...

