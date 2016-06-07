---
layout: post
title: "Setting up a service account in Azure"
date: 2016-06-07 16:47:03
categories:
---

I have been tasked with some [Azure][azure] work for chef, including [knife-azure][knife].
In the process of setting it up, the new version of Azure is called [ARM][arm],
unfortunatly the majority of plugins play off of [ASM][asm] also known as classic.

The newest version of knife-azure 1.6.0, now supports `knife azurerm` commands
to directly talk to ARM.

Unfortunatly you need to have a [Service Account][service] for this to work. I
walked through this; and here are my notes for my future self. Including the errors: :wink:

```bash
12:22:01 JJs-MacBook-Pro ~ > azure config mode arm # always do this, you don't want ASM.
12:23:01 JJs-MacBook-Pro ~ > azure login
12:24:01 JJs-MacBook-Pro ~ > azure ad app create --name "serviceapp" --home-page "https://www.contoso.org" --identifier-uris "https://www.contoso.org/example" --password <FAKEPASSWORD>
info:    Executing command ad app create
+ Creating application serviceapp
error:   {"odata.error":{"code":"Request_BadRequest","message":{"lang":"en","value":"Another object with the same value for property identifierUris already exists."},"values":[{"item":"PropertyName","value":"identifierUris"},{"item":"PropertyErrorCode","value":"ObjectConflict"}]}}
info:    Error information has been recorded to /Users/jjasghar/.azure/azure.err
error:   ad app create command failed

12:38:07 JJs-MacBook-Pro ~ > azure ad app create --name "serviceapp" --home-page "https://www.contoso.org" --identifier-uris "https://jjasghar.github.io/azure/serviceapp" --password <FAKEPASSWORD>
info:    Executing command ad app create
+ Creating application serviceapp
data:    AppId:                   0c27a338-60cd-4b19-9521-5ecf3bd02098
data:    ObjectId:                bedf8d6a-e2f1-4504-8c15-357af9493c1b
data:    DisplayName:             serviceapp
data:    IdentifierUris:          0=https://jjasghar.github.io/azure/serviceapp
data:    ReplyUrls:
data:    AvailableToOtherTenants:  False
info:    ad app create command OK
12:38:45 JJs-MacBook-Pro ~ > azure ad sp create 0c27a338-60cd-4b19-9521-5ecf3bd02098
info:    Executing command ad sp create
+ Creating service principal for application 0c27a338-60cd-4b19-9521-5ecf3bd02098
data:    Object Id:               f7d9a1f1-2d67-4250-9ffa-5c804816b7a8
data:    Display Name:            serviceapp
data:    Service Principal Names:
data:                             0c27a338-60cd-4b19-9521-5ecf3bd02098
data:                             https://jjasghar.github.io/azure/serviceapp
info:    ad sp create command OK
13:35:02 JJs-MacBook-Pro ~ > azure role assignment create --objectId f7d9a1f1-2d67-4250-9ffa-5c804816b7a8 -o Reader -c /subscriptions/FAKESUB/
info:    Executing command role assignment create
+ Finding role with specified name
/data:    RoleAssignmentId     : /subscriptions/FAKESUB/providers/Microsoft.Authorization/roleAssignments/1178037b-16e9-4bb9-803d-b1ad3cd7383b
data:    RoleDefinitionName   : Reader
data:    RoleDefinitionId     : acdd72a7-3385-48ef-bd42-f606fba81ae7
data:    Scope                : /subscriptions/FAKESUB
data:    Display Name         : serviceapp
data:    SignInName           :
data:    ObjectId             : f7d9a1f1-2d67-4250-9ffa-5c804816b7a8
data:    ObjectType           : ServicePrincipal
data:
+
info:    role assignment create command OK
13:38:33 JJs-MacBook-Pro ~ > tenantId=$(azure account show -s FAKESUB --json | jq -r '.[0].tenantId')
13:38:36 JJs-MacBook-Pro ~ > appId=$(azure ad app show --search serviceapp --json | jq -r '.[0].appId')
13:38:41 JJs-MacBook-Pro ~ > azure login -u "$appId" --service-principal --tenant "$tenantId"
info:    Executing command login
Password: **************
-info:    Added subscription Partner Engineering
+
info:    login command OK
13:38:56 JJs-MacBook-Pro ~ >
```

Now you'll notice that i used `-o Reader` on the account. That was a screwup, it
needed to be, `-o Contributor`. I ended up having to do:

```bash
$ azure account clear
$ azure login
```

I ended up logging in as my Service Account and wasn't able to delete myself.
Doing that `clear` allowed me to log back in as my User then delete the account via:

```bash
$ azure role assignment delete --objectId f7d9a1f1-2d67-4250-9ffa-5c804816b7a8 --roleName Reader
```

And I was able to recreate with the correct `-o Contributor` rights.

I set my `.chef/knife.rb` with the following:

```ruby
knife[:azure_tenant_id] # found via: tenantId=$(azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId')
knife[:azure_subscription_id] # found via: <subscriptionId>
knife[:azure_client_id] # appId=$(azure ad app show --search <principleappcreated> --json | jq -r '.[0].appId')
knife[:azure_client_secret] # password you set at initally
```

After this I was able to login via this account by `knife-azure` and create a vm:

```bash
$ be knife azurerm server create --azure-image-os-type debian --azure-vm-name jjtesting --azure-vm-size medium --azure-service-location "eastus" --ssh-user jj --ssh-password P@ssw0rd!
```


[asm]: https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/
[arm]: https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/
[azure]: https://azure.microsoft.com/en-us/
[knife]: https://github.com/chef/knife-azure
[service]: https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal-cli/
