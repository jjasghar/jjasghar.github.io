---
layout: post
title: "Using chef, terraform, vCenter and Windows"
date: 2017-10-27 14:41:23
categories: chef sysadmin vmware windows terraform
---
I posted recently about using Terraform, vCenter, and Chef, and promised a follow-up post about extending the Terraform plans to work with Windows and multiple Virtual Machines. Here is the follow-up, it will be significantly shorter than [the previous post](http://jjasghar.github.io/blog/2017/10/22/chef-terraform-and-vcenter/) but it will still build off of this plan:

[https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35)

## Windows

Go ahead and open up the following link to show off the `.diff` that has Windows working with the above Terraform plan.

[https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267](https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267)

If you don't know how to read `diff`s, the green or `+` lines are lines I've added, while the red or `-` are lines I have removed.

With a successful Linux VM build, the next natural progression is to get Windows working with the same base plan. There is some prep work that is required down this path and I chose the one that made the most sense for me. You'll need to figure out what is the correct path for you here, and it will probably require talking to your VMware Administrators and Windows Licensing people to make sure you're in compliance with your environments rules.

I don't use [Customization Specs](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-EB5F090E-723C-4470-B640-50B35D1EC016.html) very heavily so I'm going to move past it. If you do use them, you'll need to first remove [this line](https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267#file-one_machine_chef_vcenter_win-diff-L8), that skips customization specs and just request the template in its base form. If you don't have this line, and no customization spec, you'll find that Terraform [`sysprep`](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview)s the machine, and can wipe out WinRM settings. This was a gotcha that caused the longest delay in figuring out how to get Terraform to request a Windows VM for me. These settings are required for Chef to be bootstrapped into your Windows VM, you need to enable and set some WinRM settings. Taken from the [winrm-cli](https://github.com/masterzen/winrm-cli) README, you need to set the following at a Powershell prompt:

```powershell
winrm quickconfig # say y here ;)
winrm set winrm/config/service/Auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
```

Take either a snapshot of the Virtual Machine, or create a template out of it, and then change [this line](https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267#file-one_machine_chef_vcenter_win-diff-L17) to that name. The next thing to take notice of is the `diff` of the `variables.tf`. Scroll down to the next file, and look at [Lines 16 to 25](https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267#file-variables-diff-L16-L25) you'll see the changes that are required to talk to the template I created. Notice the WinRM change, for the `connection_type` the user of `Administrator`, and my super secure `Admini@` local administrator password. If you scroll back up to [Line 55](https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267#file-one_machine_chef_vcenter_win-diff-L55) in the `diff` you'll notice one more setting I set for the connection. For the settings for WinRM above, we connect over `http` and by default Terraform connects over `https`. This option forces the correct protocol.

Just to make sure this is clear, the final major change is the middle stanza of [lines 22 to 38](https://gist.github.com/jjasghar/dbd7348240f23a26a31cd7a02dcb4267#file-one_machine_chef_vcenter_win-diff-L22-L38). Not having specific Ubuntu available commands and `bash` on my Windows VM, these commands would ultimately fail on run. I am pretty confident that you can run `Powershell` commands instead of `bash` here, though to be honest, I haven't tried it. Reading the [`remote-exec`](https://www.terraform.io/docs/provisioners/remote-exec.html) docs it seems that `The remote-exec provisioner supports both ssh and winrm type connections` which at least imply it can run whatever commands you type there.

## Multiple machines

Now that we've walked through creating a virtual machine of both Linux and Windows based operating systems, we need to figure out how to make multiple machines spin up at once. This was surprisingly easy as soon as the base understanding of the `.tf` plans are put together. Let's take a look at [this diff](https://gist.github.com/jjasghar/e91c0cce6e6ba64736d04019ade67340) to show how to extend the base plan to 3 virtual machines.

Looking first at the `variables.tf` diff, you'll notice we added three lines for a default number of 3 and called it `count`. This should be pretty straightforward, and if you needed to override it you know how to from the previous post. Because we don't know how many nodes we want we had to remove [line 18 and 19](https://gist.github.com/jjasghar/e91c0cce6e6ba64736d04019ade67340#file-variables-diff-L18-L19) so we don't hardcode the default name now.

Scroll back up to the main `diff` file, you'll see that if you give the `vsphere_virtual_machine` resource a `count` it will create that many machines, as we do on line 8. There are a lot of examples on how to name the machines, I chose from:

```
variable "count" {
  default = 2
}

resource "aws_instance" "web" {
  # ...

  count = "${var.count}"

  # Tag the instance with a counter starting at 1, ie. web-001
  tags {
    Name = "${format("web-%03d", count.index + 1)}"
  }
}
```
And edited it to my liking on line 12. This way now my machines will be called `terraform-0X` where `X` is the number of the machine count we created. Finally, line 20 and 21 create the node objects on the chef server with the same name of the machine inside vCenter, which helps keep things in line.

## Clean up

So if you've played with these Terraform plans, the easiest way is just to run `terraform destroy` to nuke the machines from vCenter. It won't delete the node objects from the Chef server, that's by design, and the reason why we have the `recreate_client` line set to `true`.

I hope this helps bootstrap your use cases of Terraform, vCenter, and Chef, makes the path to your success that much easier. I know as I stated in the beginning, starting off from nothing with these three technologies is hard, but as soon as you wrap your head around some simple examples you'll find yourself programmatically requesting resources from repeatably and efficiently in no time.
