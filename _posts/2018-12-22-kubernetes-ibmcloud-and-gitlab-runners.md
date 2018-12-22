---
layout: post
title: "How to use IBM Kubernetes Service and Gitlab Runners"
date: 2018-12-22 12:26:08
categories: ibmcloud kubernetes gitlab
---

# Scope

This document helps teach you how to connect your IBM Kubernetes cluster to a Hosted Gitlab
instance so you can leverage your own infrastructure for continuous integration and continuous
deployment.

Most of this document is influenced from the [official documentation][gitlabrunnerdocs] from Gitlab.

## Prerequisites

1. A Kubernetes cluster on the IBM cloud
- An IP that the your Gitlab instance can reach
2. [ibmcloud][ibmcloudcli] CLI installed and configured

## Steps

1. Authenticate with the IBM cloud CLI so you can connect to your Kubernetes cluster.
```bash
ibmcloud login # with --sso if you need to use single sign on
ibmcloud cs region-set us-south # The region you have your cluster in
ibmcloud cs cluster-config YOURCLUSTERNAME # you will need to run the export KUBECONFIG command
kubectl get nodes # a sanity check to make sure you can authenticate with your Kubernetes cluster
```

2. Create a directory to do some work inside of it, and save some files if needed.
```bash
mkdir gitlab-runner
cd gitlab-runner
```

3. Add the `helm` repo from gitlab.
```bash
helm repo add gitlab https://charts.gitlab.io
```

4. Initialize `helm` on both your local machine and in your Kubernetes cluster.
```bash
helm init
```

5. Create a `values.yaml` for your configuration of gitlab-runner, the full [values.yml is here][values].
```bash
wget https://gitlab.com/charts/gitlab-runner/raw/master/values.yaml
vi values.yaml
```

6. The only two settings you have to edit in the `values.yaml` are the following:
 - `gitlabUrl` - the GitLab Server URL (with protocol) to register the runner against
 - `runnerRegistrationToken` - The Registration Token for adding new Runners to the GitLab Server. This must be retrieved from your GitLab Instance. See the [GitLab Runner Documentation][gitlabrunner] for more information.

7. After the configuration is complete, run the following command with a NAMESPACE that won't step on another project. Using something like `gitlab-runners` as the namespace will isolate the pods in your Kubernetes cluster.
```bash
helm install --namespace NAMESPACE --name gitlab-runner -f values.yaml gitlab/gitlab-runner
```

8. You should now be able to see your runner in the `admin/runners` URL on your Gitlab instance.

## Uninstalling

To uninstall the GitLab Runner Chart, run the following:
```bash
helm delete --namespace NAMESPACE RELEASE-NAME
```

[gitlabrunner]: https://docs.gitlab.com/ee/ci/runners/README.html#creating-and-registering-a-runner
[gitlabrunnerdocs]: https://docs.gitlab.com/ee/install/kubernetes/gitlab_runner_chart.html
[helm]: https://docs.helm.sh/using_helm/#quickstart
[ibmcloudcli]: https://console.bluemix.net/docs/cli/index.html#overview
[values]: https://gitlab.com/charts/gitlab-runner/blob/master/values.yaml
