---
layout: post
title: "chef-client in VMware templates"
date: 2017-07-18 10:01:10
categories: chef sysadmin vmware
---

As we have grown in our VMware and Chef integrations people are starting to ask for
some best practices. As of right now, (July 18, 2017), I'm going to put down my foot
and declare my first "Best Practice with Chef and VMware."

No, _don't_ put the `chef-client` in your template.

I repeat, no, **don't** put your `chef-client` in your template.

Why? A couple reasons, but it boils down to this. As you start building more and more
complex pipelines for your infrastructure, you may find you'll need humans to validate
your templates if you add the `chef-client` to your VMware template(s).

By default, when you run [test-kitchen][kitchen] it installs the latest
version of `chef-client`. If you don't pin your version in your `.kitchen.yml` to the
version that you run in your template you will find you'll be testing different versions
of the `chef-client` when you do cookbook development. Take it one step farther, if you
have the `chef-client` already installed, you'll be testing not the version you deploy
but the newest on top of the pre-baked. Decoupling the version you develop and have in
production. If you allow the version to be controlled and enforced externally via
the ways Chef gives you, consistency can be achieved.

In order to gain the consistency, leverage your bootstrap process to change the version of
your `chef-client` as a variable in either your `kitchen.yml`, your [knife][knife] bootstrap,
or `curl || bash`. This will be significantly easier to support as you're pipelines get more and
more advanced. Leveraging your code to control this and enforce the version of the `chef-client`
will save you a good amount of headache down the line.

To see how easy it is, here's some examples of pining the version of the `chef-client` in the different
tools:

## kitchen.yml

```yaml
driver:
  name: dokken
  privileged: true
  chef_version: <%= ENV['CHEF_VERSION'] || "current" %>
```

## knife

```shell
$ chef exec knife bootstrap ubuntu@web01.tirefi.re --bootstrap-version "13.2.20"
```

## curl || bash

```shell
$ curl -L https://chef.io/chef/install.sh | sudo bash -s -- -P chef -v "13.2.20"
```

[kitchen]: http://kitchen.ci/
[knife]: https://docs.chef.io/knife.html
