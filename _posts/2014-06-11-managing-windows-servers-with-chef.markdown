---
layout: post
title: "Managing Windows Servers with Chef"
date: 2014-06-11 16:42:36 -0500
comments: true
categories: book chef windows
---

I was asked to do a review of [Managing Windows Servers with Chef](http://www.packtpub.com/managing-windows-servers-with-chef/book). I've read
through the complete book, and these are my thoughts. I'll be copying the following out to a couple places, but these are my thoughts on this
book from [packtpub](http://www.packtpub.com).

## Overview
First off this book is an amazing "whistle stop tour" of the chef ecosystem. Yes it focuses mainly on the windows world, but it hits some great
overarching chef terminology to a unseasoned reader. It does assume you might have a small amount chef knowledge, in certain spots, but if you have none it's ok. It
drops you into the chef world, walks you through some basic patterns and then starts the specific windows focus. It has a great hands on example
leveraging both via knife bootstrap and cloud providers for windows, and even shows how to leverage winrm if you have no exposure to it. This is a great book, I strongly suggest
getting your hands on it.

## View as a *nix admin with chef knowlege

As a *nix admin by profession, this was a great overview, allowed me to leverage my chef knowledge already and started thinking of what I can do for my
company to leverage the repeatable windows chef infurstruture. It opened my eyes to some very basic but lacking windows support.  If you look at the following snippet:

```ruby
if platform_family? 'debian'
  package 'apache2'
elsif platform_family? 'windows'
  windows_package node['apache']['windows']['service_name'] do
    source node['apache']['windows']['msi_url']
    installer_type :msi
    # The last fourz options keep the service from failing
    # before the httpd.conf file is created
    options %W[
      /quiet
      INSTALLDIR="#{node['apache']['install_dir']}"
      ALLUSERS=1
      SERVERADMIN=#{node['apache']['serveradmin']}
      SERVERDOMAIN=#{node['fqdn']}
      SERVERNAME=#{node['fqdn']}
    ].join(' ')
  end
end
```

As you can see chef has a way to pull down `apache2` and extract, install, and run it. Before reading this book, I knew of some of the resources that chef provided, but
having legitimate examples in front of me made the difference. _I can't think of any more praise other than I really do think this has some of the best overarching view of
chef for windows, walks you through a great example and then makes you want to dig into the LWRPs that it references._


[John Ewart](https://twitter.com/soysamurai) without realizing has created a 100 page book that you can give to a Director or above and they'll get why you want to use chef and
in this case chef for windows.
