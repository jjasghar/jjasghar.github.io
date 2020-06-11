---
layout: post
title: "Building a stackedit instance on Kubernetes"
date: 2020-06-11 16:52:36 -0500
comments: true
categories: kubernetes linux sysadmin
---
1. Login the kubernetes cluster
2. create a new namespace
```bash
kubectl create namespace stackedit
```
3. add the helm repo
```bash
helm repo add stackedit https://benweet.github.io/stackedit-charts/
```
4. update the repo
```bash
helm repo update
```
5. Create a Github OAuth App
6. Create a values.yaml with the secrets
```yaml
githubClientId: ""
githubClientSecret: ""
```
8. Deploy the app via `helm`
```bash
helm install stackedit stackedit/stackedit --values=stackedit_values.yaml -n stackedit
```


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk3MTgzNzMwN119
-->