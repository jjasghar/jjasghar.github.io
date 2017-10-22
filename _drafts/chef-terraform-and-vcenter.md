---
layout: post
title: "Using chef, terraform, and vCenter"
date: 2017-10-22 15:06:47
categories: chef sysadmin vmware terraform
---
If you would like to use Terraform as your provisioning mechanism, you'll quickly discover that it's a challenge to start working with from nothing. The official terraforming documentation, as of 2017-10-22 is hit and miss, which can cause extreme frustration for many starting leveraging this workflow.

As a thought experiment, I wanted to figure out how to get terraform working with vCenter and Chef, and these are my notes and my how to. Hopefully, this will help someone in the future and give some context around the `.tf` files. The following link are two examples of working `.tf` files that work in my lab, this took a bit of time to glue together, but are commented to help explain what’s going on. I’ll be walking deeper through them in this post, so I suggest either having it open with this post beside it or pulling it down to your favorite text editor.

[https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35)

Assuming you clicked on the link you'll notice that it's called [`one_machine_chef_vcenter.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-one_machine_chef_vcenter-tf) and [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf). I did the best I could to comment what is going on, but let me explain what is happening directly here. First, scroll all the way to the bottom of the gist, and find the [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf).

## variables.tf

I realize this may seem backwards, but the best practice convention, as of 2017-10-22, is to have everything that can be overwritten to put in a [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf) file. Terraform pulls everything in a directory when you run `terraform apply`, so as a human it's nice to have good labeling for a file that you know are variables.

If you git pulled, saved, or copypasta'd these files down, the first thing you have to do is put them in _their_ own directory. This was gotcha that I missed going through the tutorial and it's good to highlight this here. Terraform creates a few other files when you run the commands, and if you don't have it in it's own directory you can clobber things when terraform processes, and that's never good.

OK, so we have the files down let's open the [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf) or look at it at the link and let's walk through it.

Lines 1 through 14 seem pretty obvious, but to make it as clear as possible, it's the username, password, and server you want to drive. Putting the password in the file is a pretty dumb idea, but this was an example, so let's say we want to inject it at `terraform plan` instead. Remove lines 6 through 9 and add something like this: `terraform plan -var 'vsphere_password=Good4bye!'` This will inject the password in the plan phase, which is a more secure. There are other options so, if you want to dig farther into the docs, this [this link](https://www.terraform.io/intro/getting-started/variables.html) is the base usage while [this link](https://www.terraform.io/docs/configuration/variables.html) is a much deeper dive of the variables. You can see there is a ton of things you can do here, but let's iterate instead of bite it all at once.

Lines 16 through 33 are my connection to the Chef server I have and some default settings so I don't have to worry about it. Having a sane defaults are always a good idea, and these are mine. You can override them just like we did with removing password from the file, as an experiment I suggest you giving that a shot. Probably the most interesting line here is line 29, `ssl_verify_mode`. As a practitioner of Chef you've probably set this mode with your self-signed Chef server multiple times. This option is deep in the docs, and this is the line that set it to your setting you know you use. As you can see it's in `" "` as a string that is passed up to the client. I can't stress how hard it was to find this setting, and I hope I helped you here. I’ve added a [PR](https://github.com/hashicorp/terraform/pull/16412) to hopefully make it more clear, and streamlined the usage.

Because I'm horrible at naming things, lines 35 through 47 are my `"connection_thingys"`. In order to get into the machine terraform needs a way to either SSH (in this case) or WinRM into the machine. Setting my default username and password for my templates and allow them to be overwritten for my future self. Just a tidbit of fun here, all my administer accounts are called `admini` because Ubuntu won't let you have `admin`, and I hate passwords so I use `admini` as the password. For CentOS, I use `root/admini` to be as consistent as I can. So if I wanted to bootstrap a CentOS machine I would do something like: `terraform plan -var “connection_thingys.connection_user=”root”`

## one_machine_chef_vcenter.tf

Scroll back up or open up [`one_machine_chef_vcenter.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-one_machine_chef_vcenter-tf) now, this is the meat of the integration. It's pretty verbose, which is a dealers choice of good or bad. For showing this off to someone, I think it's really nice but alas, figuring it out by hand is a test of patience.

Because we are going to be connecting to vCenter, we need to talk to the vSphere provider. We can talk about why they are named differently, but that's a whole other blog post. (Marketing changing names of projects always has downstream issues eh?) Lines 1 through 8 are the variablized connection options. Because I didn't namespace them, and they are at the "root" of the [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf) file they are just called with `${var.blah}` this is helpful to know as you start making your own. Because I don't have anything other than self-signed certs, line 7 allows you to connect to that vCenter you know you don't have a valid cert for.

The rest of the file is the declaration of the _one_ machine. Yes, it's only one, which seems a tad bit overkill, but as I say above it's verbose and does exactly what you'd like to do. There are three main stanzas is this section, a resource stanza which you can probably guess is the actual machine, and two the provisioner stanzas. We'll start with the resource then talk about the two provisioners.

The resource declaration is line 11 through 39. We tell the resource that we want a `vsphere_virtual_machine`, which tells it to create a VM object in vCenter. If you click on [this link](https://www.terraform.io/docs/providers/vsphere/r/virtual_machine.html) you can see the official documentation, and more interestingly, take a look at the left-hand sidebar. There are all of these different resources that you can declare, so _in theory_ you could spin up a whole infrastructure in your VMware environment, and tear it down with two commands. I challenge you to attempt this after you get the basics of just one machine down.

Line 13 is the name of the machine you want _inside_ vCenter. If you have some standardization of naming conventions this is where you'd add this though as you get more comfortable with this maybe create this as a variable? You could extend this and run it on the command line if you had to do multiple of these.

Lines 14 through 17 are the DNS entries. If you want the DNS _inside_ the VM to be set, this is a way for the drive to do it, line 15 is _has_ to be a list. This is something like `["tirefi.re","example.com"]` but as you can see, I only use `tirefi.re`. By default the driver adds `vsphere.local` as the default external DNS, if you want to force a different domain, line 17 is how to do it.

Lines 18 through 27 are where you'd like to put it inside of vCenter. My datacenter's name is "Datacenter", if you had multiple DC objects, like "West" and "East", in vCenter you could force it here by changing it to "East" or "West." Maybe this is another place for a variable?
If you need to bump the size of your cloned template lines 20 through 23 is a way to do it. I think these are my default settings from the template, but this at least shows this off. Line 24 and 25 is the resource pool that you want to put it in. I believe that this is required, but I need to confirm this, if it is, I'll add the PR to make it required in the docs. Lines 26 and 27 aren't required, but you should be using linked clones if you aren't.

Assuming your template has at least one snapshot, this will clone the template via [linked clone](https://www.vmware.com/support/ws5/doc/ws_clone_overview.html) and the _newest_ snapshot. It's significantly faster, per my test I show here:

```
vsphere_virtual_machine.terraform_01: Creation complete after 3m24s (ID: terraform-1) # as a normal clone
vsphere_virtual_machine.terraform_01: Creation complete after 1m27s (ID: terraform-1) # as a linked clone
```

Now you have the machine declared, you probably need a place to put on the network, lines 29 through 32 is an example of how to do it. My main network from my VMs is called `"Internal Network 3208"` so that's how it creates and connections the vNIC to the network.

Lines 34 to 39 is the disk setup. This is the _bare_ minimum you need to create what is needed for your VM to work, which are the template and the datastore you want to inject it into. As you can see the template line is self-explanatory, but line 38 is more interesting. As you can see my name of my data store is `"vsanDatastore"` which yes is a vSAN datastore. So the driver allows you to leverage vSAN if you use it, which was a pleasant surprise, to say the least.

Now that we have fully declared the machine including networking and disk, you could remove the rest of the file and run this. That's pretty neat, a repeatedly variablized plan to always request vCenter for exactly what you want. But I like the next stanzas more, this is where the post provisioning requests start to shine.

Let's continue by talking about the first section lines 42 through 56. This provisioner called `remote-exec` as you can assume runs arbitrary commands on the remote machine. Being this demo is a Ubuntu box and has bash, line 43 to 50 are just bash commands. You can also use powershell, if you have powershell access or on a Windows template, but I’m on Ubuntu here so bash is all I have. I've had trouble with locked `dpkg`s in the past, so this is me forcing unlocking it, and installing any broken dependency packages including auto removing any hanging packages. Line 49 has probably caught your eye. This is the _only_ way I could figure out how to inject something to the `/etc/hosts` on initial post provisioning. Like all DNS administrators know,  DNS is always more challenging to work with then you'd like, so I added this line to force where my Chef server ip is located.

Let's take a detour and a quick explanation on this specific command:

```shell
echo admini | sudo -S echo '10.0.0.15 chef chef.tirefi.re' | sudo tee -a /etc/hosts
```

First the `echo` is the `sudo` password for my user `admini`. The middle pipe is runs `sudo` with the `read password from standard input` flag `-S` on it, and `echo`s the ip short name and fully qualified domain name. The third pipe runs `sudo` again with the cached password and appends the `/etc/hosts` file using `tee -a`. Yes, this seems like a lot, but the beauty of this example it now shows you how to inject arbitrary lines on post provisioning with `sudo`. It’ll be worth playing with this chain of commands to wrap your head around this. It’ll help you a lot as you get more advanced.

Line 51 to 55, is the way to SSH into the machine. It pulls in from the [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf) and injects them in there.

The next provisioner stanza is from line 58 to 75 andis how to bootstrap chef into the machine. If you use chef all of these settings should seem extremely familiar. I won't go through them all, but I'll highlight the possible gotchas. Line 62 is how you can get the `.pem` in as your `username` from line 60. It seems that terraform doesn't like this to be interpolated so this was the only way I could figure out how to get it to work. If you have an initial base cookbook, line 65 is where you can declare the run list for it. The connection section uses the same settings as the `remote-exec` which needless to say is pretty useful because we can just declare it once via the [`variables.tf`](https://gist.github.com/jjasghar/f061f493ad8f631a6d4b5b5085c7cb35#file-variables-tf).

With all of these settings, if you wanted to bootstrap a VM, with some chef injected and ran, in vCenter you can, with a few simple commands like the following:

```shell
~/terraform/testing $ terraform plan # makes sure that the plan works and creates the initial plan.
~/terraform/testing $ terraform init # if the previous fails due to missing plugins or settings, this command pulls down vsphere, for instance
~/terraform/testing $ terraform apply # run the terraform plans after you've set everything up this should be the only command you ever need to run to build this.
~/terraform/testing $ terraform destroy # will remove the running VM, but _not_ remove it from the Chef server. Take note of this, the recreate client option will recreate it
```

Hopefully, you've learned something here or at least seen the power of leveraging terraform for  something like this. I realize you can do this with just a simple `knife` command, but if you are building up _multiple_ nodes repeatedly in your infrastructure terraform is just a better user experience. I’m working on Windows and multiple node examples now too, so stay tuned for these examples. I’ll build off what I have here and comment any major changes to get those more advanced examples to work.
