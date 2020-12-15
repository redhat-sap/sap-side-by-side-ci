# GitOps and CI/CD implementation for side-by-side demo

This repository contains the source to implement GitOps and CI/CD for the side-by-side [backend](https://github.com/redhat-sap/sap-side-by-side-be) and [frontend](https://github.com/redhat-sap/sap-side-by-side-fe) demo applications using [Argo CD](https://argoproj.github.io/argo-cd/) and [Tekton Pipelines](https://tekton.dev/)

## Instructions to deploy and test

All the required files to deploy and configure Argo CD and OpenShift Pipelines are included on this repo. The very first thing you must do is to fork and clone this repo as you will need to commit some changes into your own fork to check all the features in action.

You must be logged into your OpenShift Cluster using `oc` or `kubectl` client in order to follow the instructions bellow.


### Deploy OpenShift Pipelines Operator (Tekton)

This can be done using the OpenShift Console as well, installing `Red Hat OpenShift Pipelines Operator` from OperatorHub. If you want to use the terminal instead, just execute the following instructions.

```bash
oc create -f tekton/subscription.yml
```

### Deploy Argo CD Operator

This can be done using the OpenShift Console as well, installing `Argo CD` from OperatorHub. If you want to use the terminal instead, just execute the following instructions.

```bash
oc create -f argocd/namespace.yml
oc create -f argocd/operatorgroup.yml
oc create -f argocd/subscription.yml
```

### Create an Argo CD Deployment

```bash
oc create -f argocd/argocd.yml
oc create -f argocd/cluster-role-bind.yml
```

### Create new Applications in Argo CD

```bash
oc create -f argocd/argo-app-be-dev.yml
oc create -f argocd/argo-app-be-qa.yml
```

### Get Argo CD credentials for `admin` user

```
oc get secret sap-argocd-cluster -o jsonpath='{.data.admin\.password}' | base64 -d
```

### Configure Pipelines

```bash
oc create -f tekton/be-dev-pipeline.yml
oc create -f tekton/be-dev-pipeline-run.yml
oc create -f tekton/be-qa-pipeline.yml
oc create -f tekton/be-qa-pipeline-run.yml
```