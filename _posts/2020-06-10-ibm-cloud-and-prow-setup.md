---
layout: post
title: "IBM Cloud and prow setup"
date: 2020-06-10 15:26:29
categories: ibmcloud, kubernetes, cicd, devops
---
# Setting up basic `prow` on the IBM Cloud

## Scope

This will tell walk you through specifics on getting [prow][prow] working on the
IBM Cloud. A lot of this is taken from the [getting started][guide] but there are
some specifics you need for the IBM Cloud. You can also take this as an example
for a generic Kubernetes Cluster minus the IBM Cloud commands.

### Create a Bot

The first thing you should do is create a GitHub bot. It'll be the account that
will work from your `prow` instance. If you don't know how ore where use this
[signup][signup] page.

Be sure to add the bot to the repositories you expect it to watch, it should be
an administrator/owner, to make sure it can see and do what it needs with the
repository. Next create an GitHub Personal Access Token for the bot.

For permissions add the following:
Create a personal access token for the GitHub bot account, adding the following scopes
- Must have the public_repo and repo:status scopes
- Add the repo scope if you plan on handing private repos
- Add the admin:org_hook scope if you plan on handling a github org

Place this API key in a file like `github_token` or the like.

### Check out the `prow` code

Next you need the `prow` code so you can install it on your Kubernetes Cluster.
Go ahead and checkout the code with the following command.

```bash
git clone git@github.com:kubernetes/test-infra.git
```

Change directory into the `test-infra` directory you created, and continue
to the next step.

### Create cluster role bindings

You'll need to create a `clusterrolebinding` for your user you log in as.
On the IBM Cloud, your username is `IAM@IBMid` as an example below is mine.

```bash
export USER="IAM#jja@ibm.com"
kubectl create clusterrolebinding cluster-admin-binding-${USER} --clusterrole cluster-admin --user="${USER}"
```

This will make sure that when you apply the manifests the `prow` instance can
create all the different things that is required on your Kubernetes cluster.

### Create a GitHub Secret

Next you need ot cerate your secrets for your instance of `prow`. Using the following
commands you can create your main `secret` to send the Webhook endpoint.
Use the `github_token` you created earlier for the 3rd command.

```bash
openssl rand -hex 20 > ./secret
kubectl create secret generic hmac-token --from-file=hmac=./secret
kubectl create secret generic oauth-token --from-file=oauth=../github_token
```

### Apply starter.yaml

Now that you have the majority set up, you need to deploy the actual manifests for
`prow`. The following command will push and start installing your instance. 

```bash
kubectl apply -f config/prow/cluster/starter.yaml
```

Now verify the deployments, everything should be in `READY` state and `AVAILABLE`.

```bash
kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deck               2/2     2            2           2m
hook               2/2     2            2           2m
horologium         1/1     1            1           2m
plank              1/1     1            1           2m
sinker             1/1     1            1           2m
statusreconciler   1/1     1            1           2m
tide               1/1     1            1           2m
```

### Setting up Ingress correctly

Verify the ingress is set up via this next command. The default ingress controller 
needs some editing, but to make sure that it is "up" the following command will
verify it.

```bash
kubectl get ingress ing
```

If everything looks OK, you can now move to change the ingress via the next specific 
to IBM Cloud steps.
Using the IBM cloud you get an ingress controller for free. Run the following command
to get the needed information, where `k8s.asgharlabs.io` is the name of your cluster:

```bash
$ ibmcloud ks cluster get --cluster k8s.asgharlabs.io
Retrieving cluster k8s.asgharlabs.io...
OK

Name:                           k8s.asgharlabs.io
ID:                             brfakb8d0dlm8ddhq91g
State:                          normal
Created:                        2020-06-08T21:15:21+0000
Location:                       dal13
Master URL:                     https://c108.us-south.containers.cloud.ibm.com:31230
Public Service Endpoint URL:    https://c108.us-south.containers.cloud.ibm.com:31230
Private Service Endpoint URL:   -
Master Location:                Dallas
Master Status:                  Ready (1 day ago)
Master State:                   deployed
Master Health:                  normal
Ingress Subdomain:              k8sasgharlabsio-706821-0e3eA_FAKE_HASH1e8aa6fe01f33bfc4-0000.us-south.containers.appdomain.cloud
Ingress Secret:                 k8sasgharlabsio-706821-0e3eA_REALLY_REALLY_FAKE_SECRETf33bfc4-0000
Ingress Status:                 healthy
Ingress Message:                All Ingress components are healthy
Workers:                        3
Worker Zones:                   dal13
Version:                        1.18.3_1514
Creator:                        jja@ibm.com
Monitoring Dashboard:           -
Resource Group ID:              5eb57fd577b64b51beb832c2e9d5287a
Resource Group Name:            Default
```

Take note of your `Ingress Subdomain` and Ingress Secret` for the next step.

## Updating `Ingress` to work

Go ahead and take the following `yaml` and change it for your subdomain and change
the `prow` to something else if you disire.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  namespace: default
  name: ing
spec:
  tls:
  - hosts:
    - prow.k8sasgharlabsio-706821-0e3eA_FAKE_HASH1e8aa6fe01f33bfc4-0000.us-south.containers.appdomain.cloud
    secretName: k8sasgharlabsio-706821-0e3eA_REALLY_REALLY_FAKE_SECRETf33bfc4-0000
  rules:
  - host: prow.k8sasgharlabsio-706821-0e3eA_FAKE_HASH1e8aa6fe01f33bfc4-0000.us-south.containers.appdomain.cloud
    http:
      paths:
      - path: /hook
        backend:
          serviceName: hook
          servicePort: 8888
      - path: /
        backend:
          serviceName: deck
          servicePort: 80
```

## Conclusion

Go to that address in a web browser and verify that the "echo-test" job has a green check-mark next to it. At this point you have a prow cluster that is ready to start receiving GitHub events!

[prow]: https://github.com/kubernetes/test-infra/tree/master/prow
[guide]: https://github.com/kubernetes/test-infra/blob/master/prow/getting_started_deploy.md
[signup]: https://github.com/join
