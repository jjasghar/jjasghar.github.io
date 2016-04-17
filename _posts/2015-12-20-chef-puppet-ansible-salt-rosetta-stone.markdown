---
layout: post
title: "Chef Puppet Ansible Salt Rosetta Stone"
date: 2015-12-20 12:17:31 -0600
comments: true
categories: chef puppet salt ansible sysadmin
---

_NOTE:_ I wrote the following in my [chef-book][chef-tri], but I'm copypasting
it here for more eyes. This really was the basis for my understanding of coding
with Chef; hopefully it'll help some one else.

In training with Puppet, you learn the [trifecta][trifecta], and there is a phrase:
"Package/file/service: Learn it, live it, love it.
If you can only do this, you can still do a lot." Which is very true.

## Chef trifecta

Let's create control over openssh-server in Chef.

```ruby
package 'openssh-server' do
  action :install
end

template '/etc/ssh/sshd_config' do
  source 'sshd_config.erb'
  owner 'root'
  group 'root'
  mode '0640'
  notifies :reload, 'service[ssh]'
end

service 'ssh' do
  action [:enable, :start]
  supports :status => true, :restart => true
end
```

## Puppet trifecta

The same above in Puppet less then version 4.

```ruby
package { 'openssh-server':
 ensure => installed,
}

file { '/etc/ssh/sshd_config':
 source  => 'puppet:///modules/sshd/sshd_config',
 owner   => 'root',
 group   => 'root',
 mode    => '0640',
 notify  => Service['sshd'], # sshd will restart whenever you edit this file.
 require => Package['openssh-server'],
 }

service { 'sshd':
 ensure     => running,
 enable     => true,
 hasstatus  => true,
 hasrestart => true,
}
```

## Ansible trifecta

Same with ansible.

```yaml
- name: install the latest version of openssh-server
  package: name=openssh-server state=present

- template: src=/mytemplates/sshd_confige.j2 dest=/etc/ssh/sshd_config owner=root group=root mode=0644

- service: name=ssh state=started
```

## Salt trifecta

Same with salt. _Credit_ to [deadbunny][deadbunny] for updating my example.

```yaml
openssh-server:

  pkg.installed:
    - name: openssh-server

  service.running:
    - name: sshd
    - enable: True
    - require:
      - pkg: openssh-server

  file.managed:
    - name: /etc/ssh/sshd_config
    - source: salt://ssh/sshd_config
    - user: root
    - group: root
    - mode: 640
    - watch_in:
      - service: openssh-server
```

In essence all of these are the exact same, all require a package, all require a template,
and all require a service.

As you can see it really depends on taste.


[chef-tri]: https://github.com/jjasghar/chef-book/blob/master/part2/06-write-simple-base-cookbook.md#chef-trifecta
[trifecta]: http://docs.puppetlabs.com/puppet_core_types_cheatsheet.pdf
[deadbunny]: https://www.reddit.com/user/deadbunny
