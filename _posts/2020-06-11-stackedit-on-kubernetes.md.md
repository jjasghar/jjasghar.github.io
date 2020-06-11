

---
layout: post
title: "Managing Windows Servers with Chef"
date: 2014-06-11 16:42:36 -0500
comments: true
categories: chef windows
---
1. Login the kubernetes cluster
1. create a new namespace
```bash
kubectl create namespace stackedit
```
1. add the helm repo
```bash
helm repo add stackedit https://benweet.github.io/stackedit-charts/
```
1. update the repo
```bash
helm repo update
```
1. Create a github oauth app
1. Create a values.yaml with the secrets
1. Deploy the app via `helm`
```bash
helm install stackedit stackedit/stackedit --values=stackedit_values.yaml -n stackedit
```


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkxMjg0MDI0NF19
-->