---
layout: post
title: "Building a stackedit instance on Kubernetes"
date: 2020-06-11 16:52:36 -0500
comments: true
categories: kubernetes linux sysadmin
---
1) Login to a kubernetes cluster

2) create a new namespace

```bash
kubectl create namespace stackedit
```
3) add the helm repo

```bash
helm repo add stackedit https://benweet.github.io/stackedit-charts/
```
4) update the repo

```bash
helm repo update
```
5) Create a Github OAuth App

6) Create a `stackedit-secrets-values.yaml` with the secrets

```yaml
githubClientId: ""
githubClientSecret: ""
```

7) Create a `stackedit-values.yaml` with ingress for ibmcloud

```yaml
ingress:
  enabled: true
  hosts:
    - host: stackedit.k8sasgharlabsio-706821-0e3e0ef4c9c6d831e8aa6fe01f33bfc4-0000.us-south.containers.appdomain.cloud
      paths: ["/"]
  tls:
    - hosts: ["stackedit.k8sasgharlabsio-706821-0e3e0ef4c9c6d831e8aa6fe01f33bfc4-0000.us-south.containers.appdomain.cloud"]
      secretName: k8sasgharlabsio-706821-0e3e0ef4c9c6d831e8aa6fe01f33bfc4-0000
```

8) Deploy the app via `helm`

```bash
helm install stackedit stackedit/stackedit --values=stackedit-secrets-values.yaml --values=stackedit-values.yaml -n stackedit
```


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM1MDU5MzYxNiwzOTIyNTgxNDZdfQ==
-->
