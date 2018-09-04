---
layout: post
title: "vmware-tools chef cookbook resurrected"
date: 2018-02-06 10:56:56
categories: sysadmin chef vmware
---
**tl;dr**: I'd like to announce that I have released an updated and modernized version of the
[vmware-tools cookbook][supermarket]. This cookbook installs [open-vm-tools][opengithub], or the
public version of [vmware-tools posted here][publicvmwaretools] which has become the [defacto standard
for vmware-tools][vmwareblog] since around the release of ESXi 6.0.

As a user/consumer of the VMware stack, you are pretty much required to use [vmware-tools][toolssite] for
your guest VMs. Most bake vmware-tools into their golden VM templates to make sure that
the ESXi hypervisor or vCenter can get VM information; or even have better performance, such as in the
case of Windows and the Web console. This is a time-honored way of making sure when your clones are
created you have at least a baseline of VMware drivers and integrations.

You're probably asking yourself, why create a Chef Cookbook to do something that should in most cases
already be there? A couple reasons, but the main one is this. vmware-tools moves extremely fast,
and with the releasing of the Linux variant of open-vm-tools, moves even quicker then
the past. This cookbook does it's best to make sure you are always running the most up-to-date version
of this technology, with the least amount of effort on your side. *Note:* As of writing this, the Windows
portion is pinned to a specific version, I'm [still working on a way to pragmatically][windows]
have it always the newest release; and if you want to help, I'd love to take a [PR][pr] to do this.

With this cookbook, you get some other surprise benefits too. If you add this cookbook to your
base cookbook, no matter what you do, every machine you bootstrap will always guarantee
to have vmware-tools updated and installed. This helps take some mental burden away because
you can trust this cookbook will at least get your VMware infrastructure baseline integration
done. In turn, you can retire some of the "golden image" steps away, allowing for an easier
pipeline and more dynamic versioning of the code.

Take this quick win example, you are just implementing Chef in your VMware infrastructure. You need
to show value quickly and get the [chef-client][chefclient] on every one of your VMs. This cookbook
is something that can be used to show continual value and only make changes to the VM if the machine
doesn't already have vmware-tools installed, which is most likely required anyway.

I too the belief of the Unix philosophy here, I wanted to create something that when you add
this cookbook to your Chef Server it does one thing and one thing well. It's one less thing to worry
about and as long as you use Chef to drive your infrastructure as code, you'll have vmware-tools
updated and installed.


[chefclient]: https://docs.chef.io/chef_client_overview.html
[github]: https://github.com/jjasghar/chef-vmware-tools
[opengithub]: https://github.com/vmware/open-vm-tools
[pr]: https://github.com/jjasghar/chef-vmware-tools/pulls
[publicvmwaretools]: https://packages.vmware.com/tools/esx/latest/windows/x64
[supermarket]: https://supermarket.chef.io/cookbooks/vmware-tools
[toolssite]: https://docs.vmware.com/en/VMware-Tools/index.html?src=af_5b804d3334401&cid=70134000001YXKx
[vmwareblog]: https://blogs.vmware.com/vsphere/2015/09/open-vm-tools-ovt-the-future-of-vmware-tools-for-linux.html?src=af_5b804d3334401&cid=70134000001YXKx
[windows]: https://github.com/jjasghar/chef-vmware-tools/blob/master/recipes/_windows.rb
