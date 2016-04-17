---
layout: post
title: "Using chef-provisioning-fog for your OpenStack cloud"
date: 2015-09-25 15:22:14 -0500
comments: true
categories: linux sysadmin chef
---

So you've decided to use [chef-provisioning](https://github.com/chef/chef-provisioning) and you want to use [chef-provisioning-fog](https://github.com/chef/chef-provisioning-fog) to talk to your [OpenStack](https://www.openstack.org/) cloud. Awesome, but there are a few things you need to do before it works.

First, you need to probably read up on [chef-provisioning](https://github.com/chef/chef-provisioning). I'll be honest, it's confusing, complex but crazy powerful if you get the foundation. My **tl;dr** of using chef-provisioning is: you want a `machine` resource to build a cluster/multiple machines to do something.  For instance building out a Dev/QA environment from scratch for an application you support. Or building out a "POD" for another instance of your application; chef-provisioning'll take care of the machines and orchestration for you.

Like all chef recipes, it starts at the top. So if you need to build out a specific machine first, then have later machines check in with it. A provisioning recipe will complete each one in order, and not continue until it's done. With this type of orchestration you can create interdependencies using your chef server with chef search to have machines "know" about each other during this provisioning cycle. Pretty slick. This example won't show this; but this'll at least get you started on that path.

Ok, so lets say you want 4 machines, 3 dev-webservers, and 1 qa-webserver, with different base images, and user accounts. For whatever reason you're dev is Ubuntu, and your QA is CentOS. You'll need to write something like the following:

```ruby
require 'chef/provisioning'

with_machine_options({
                       :bootstrap_options => {
                         :flavor_ref  => 3,
                         :image_ref => 'my-fake-ubuntu-image-0c1f2c38432b',
                         :nics => [{ :net_id => 'my-tenantnetwork-id-89afddb9db6c'}],
                         :key_name => 'mykeyname',
                         :floating_ip_pool => 'ext-net'
                         },
                       :ssh_username => 'ubuntu'
                     })

machine_batch 'dev' do
  1.upto(3) do |n|
    instance = "#{name}-webserver-#{n}"
    machine instance do
      recipe 'apache::default'
      tag "#{name}-webserver-#{n}"
      converge true
    end
  end
end

machine 'qa-webserver' do
  tag 'qabox'
  machine_options({
                    bootstrap_options: {
                      :flavor_ref  => 3,
                      :nics => [{ :net_id => 'my-tenantnetwork-id-89afddb9db6c'}],
                      :key_name => 'jdizzle',
                      :image_ref => 'my-centos-image-2b0b6bb7b0c12b0b6bb7b0c1',
                      :floating_ip_pool => 'some-other-network'
                      },
                    :ssh_username => 'centos'
                  })
  role 'webserver'
  converge true
end
```

As you can see, you want the same "default" options for Ubuntu, but you can override the `machine_options` in the `machine` resource. You can also add a specific `role` to the boxes, in this case they are both `webserver`.

Here's an opportunity to build a `destroy.rb` so you can blow away your machine(s). Something simple like this:

```ruby
require 'chef/provisioning'

machine_batch do
  machines search(:node, '*:*').map { |n| n.name }
  action :destroy
end
```

After doing this, you'll need to install the `chef-provisioning-fog` gem.

```
$ gem install chef-provisioning-fog
```

If you have a `knife.rb` already, you can break it open and this is where you can declare how to connect to your OpenStack cloud. You'll need to fill in the following with your own strings:

```ruby
driver 'fog:OpenStack'
driver_options :compute_options => { :openstack_auth_url => 'http://YOUROPENSTACK-CLOUD:5000/v2.0/tokens',
                                     :openstack_username => 'YOURUSERNAME',
                                     :openstack_api_key  => 'YOURPASSWORD',
                                     :openstack_tenant   => 'YOURTENANTIDNAME' }
```

Something that bit me, was the name of the `:key_name`. It seems that the ssh key needs to match what you call it in the `machine_options` above. So I did this:

```
cp ~/.ssh/id_rsa ~/.ssh/jdizzle
```

I know it's not great, but hell it worked and I'm not going to complain.

After this, you can go to where you wrote your provisioning script and run these commands,

```
$ export CHEF_DRIVER=fog:OpenStack # this is only needed if you haven't added the driver to your knife.rb
$ chef-client -z my_provisioning_script.rb
```

You should see [chef-zero](https://github.com/chef/chef-zero) kick off and soon you'll see your machines in your OpenStack cloud. When you want to blow everything away, all you need to do is:

```
$ chef-client -z destroy.rb
```

This should be enough to get you off the ground. Happy Provisioning!
