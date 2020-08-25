---
layout: post
title: "How to create a cluster-admin bearer token on Kubernetes"
date: 2020-08-25 18:02:33
categories: sysadmin, kubernetes
---

Some times you need a `cluster-admin` bearer token. Here are the commands to create
one:

**NOTE**: "clusteradmin-sa" can be any name, it's good to have something-sa so you know what it is.

```bash
kubectl create sa clusteradmin-sa
kubectl create clusterrolebinding software-sa --clusterrole=cluster-admin --serviceaccount=default:software-sa
kubectl get secrets | grep software-sa
kubectl describe secret software-sa-token-<SOME-HASH>
```

The following is a `yaml` defintion that should give you the secret that does basiclly a `cluster-admin`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: software
  namespace: default
secrets:
- name: software-secret
---
apiVersion: v1
kind: Secret
metadata:
  name: software-secret
  annotations:
    kubernetes.io/service-account.name: software
type: kubernetes.io/service-account-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: software-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: software-role-binding
roleRef:
  kind: ClusterRole
  name: software-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: software
  namespace: default
```
