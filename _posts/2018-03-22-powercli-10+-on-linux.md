---
layout: post
title: "PowerCLI 10+ on Ubuntu Linux"
date: 2018-03-22 15:53:07
categories: vmware sysadmin powershell
---

Here are some direct steps to getting PowerCLI 10+ working on Ubuntu Linux.

First thing first, you need to get on a Ubuntu (debian) Machine. Now you need to
get Powershell installed:

```
~$ sudo apt-get install curl
~$ curl  https://packages.microsoft.com/keys/microsoft.asc > MS.key
~$ sudo apt-key add MS.key
~$ curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | sudo tee /etc/apt/sources.list.d/microsoft.list
~$ sudo apt-get update
~$  sudo apt-get install -y powershell
```

Awesome! Ok, next you should verify that Powershell is working as expected.

```
~$ pwsh # <--- THIS IS HOW TO START POWERSHELL
PowerShell v6.0.2
Copyright (c) Microsoft Corporation. All rights reserved.

https://aka.ms/pscore6-docs
Type 'help' to get help.

PS > Get-Date –Format U
Thursday, March 22, 2018 8:57:26 PM
PS > $PSVersionTable

Name                           Value
----                           -----
PSVersion                      6.0.2
PSEdition                      Core
GitCommitId                    v6.0.2
OS                             Linux 4.10.0-28-generic #32~16.04.2-Ubuntu SMP Thu Jul 20 10:19:48 UTC 2017
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0
PS > exit
```

Notice the exit, you need to go back to your Ubuntu command prompt to to create a directory. You might
be able to do that in the PowerShell prompt, but due to the "non-production" release, it's best to
have this created outside of the shell then recreate it.

```
~$ mkdir -p ~/.local/share/powershell/Modules
```

Now go back into Powershell and run the following to install PowerCLI.

```
~$ pwsh
PowerShell v6.0.2
Copyright (c) Microsoft Corporation. All rights reserved.

https://aka.ms/pscore6-docs
Type 'help' to get help.

PS > Install-Module -Name VMware.PowerCLI  -Scope CurrentUser
Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy
value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS >
```

Note: See the `-Scope CurrentUser`? That is required to make sure that the module gets installed in
the local directory we created in the above step.

You should be able to do the following now:

```
PS > Set-PowerCLIConfiguration -InvalidCertificateAction Ignore
Perform operation?
Performing operation 'Update PowerCLI configuration.'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): A

Scope    ProxyPolicy     DefaultVIServerMode InvalidCertificateAction  DisplayDeprecationWarnings WebOperationTimeout
                                                                                                  Seconds
-----    -----------     ------------------- ------------------------  -------------------------- -------------------
Session  UseSystemProxy  Multiple            Ignore                    True                       300
User                                         Ignore
AllUsers

PS > Connect-VIServer -Server SERVERIP -User administrator@vsphere.local -Password YOURPASSWORD
PS > Get-VM
```
