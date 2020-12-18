# GitOps and CI/CD implementation for side-by-side demo

This repository contains the source to implement GitOps and CI/CD for the side-by-side [backend](https://github.com/redhat-sap/sap-side-by-side-be) and [frontend](https://github.com/redhat-sap/sap-side-by-side-fe) demo applications using [Argo CD](https://argoproj.github.io/argo-cd/) and [Tekton Pipelines](https://tekton.dev/)

## Instructions to deploy and test

All the required files to deploy and configure Argo CD and OpenShift Pipelines are included on this repo. The very first thing you must do is to **fork and clone this repo** as you will need to commit some changes into your own fork to check all the features in action.

You must be logged into your OpenShift Cluster using `oc` or `kubectl` client in order to follow the instructions bellow.

All the Tekton pipelines and required objects will be managed by Argo CD and these are part of the Argo CD Applications deployed by the instructions bellow. Different Tekton Pipelines will be automatically configured for `dev` and `qa` branches of the following repositories which contain the application code:

- [Backend Microservice](https://github.com/redhat-sap/sap-side-by-side-be)
- [Frontend Microservice](https://github.com/redhat-sap/sap-side-by-side-fe)

To test all the capabilities included on this demo you will need to fork and clone both repositories as you will need to configure different Hooks in your forked repositories so you can test how changes on the `dev` or `qa` branches of those repositories go from there to a running microservice in OpenShift with your code.

The following files must be changed to point to your own repositories:

- app-fe-cicd/dev-webhook.yml
- app-fe-cicd/pipeline-resources.yml
- app-fe-cicd/qa-webhook.yml
- app-be-cicd/dev-webhook.yml
- app-be-cicd/pipeline-resources.yml
- app-be-cicd/qa-webhook.yml

Also you will need to change the following files to match your own repo for Argo CD so you can test changes in your configuration and see how Argo CD automatically syncs the information from GitHub into OpenShift:

- argocd/argo-app-be-qa.yml
- argocd/argo-app-be-cicd.yml
- argocd/argo-app-be-dev.yml
- argocd/argo-app-fe-dev.yml
- argocd/argo-app-fe-qa.yml
- argocd/argo-app-fe-cicd.yml


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
oc create -f argocd/argo-app-fe-dev.yml
oc create -f argocd/argo-app-fe-qa.yml
oc create -f argocd/argo-app-be-cicd.yml
oc create -f argocd/argo-app-fe-cicd.yml
```

### Get Argo CD credentials for `admin` user

```
oc get secret sap-argocd-cluster -o jsonpath='{.data.admin\.password}' | base64 -d
```

### Check results

Use the credential for Argo CD `admin` user to login into Argo webconsole and you should see something similar to this, whith all the applications managed by Argo CD started to sync their objects into OpenShift.

![Argo Applications](img/argo01.png)

If you click on any of the Applications you can see more details on the objects that have been synced already and the actual status.

![Argo Application details](img/argo02.png)

If you check the existing Pipelines on any of the `cicd` Projects that have been created in OpenShift you will see the Pipeline status and the results from the first PipelineRun once you have committed any code to your GitHub `dev` or `qa` branches. This will not happen until you follow the last step explained bellow configuring your GitHub hooks to notify the EventListeners configured for your Pipelines.

![Pipeline](img/tekton01.png)

### Configure your GitHub hook

On each GitHub repository you need to configure 2 different hooks. Use the following command to get the endpoint information for each one:

- Frontend microservices hook endpoitns

    ```bash
     oc get routes -n app-fe-cicd fe-pipeline-webhook-dev -o jsonpath='{.spec.host}'
     oc get routes -n app-fe-cicd fe-pipeline-webhook-qa -o jsonpath='{.spec.host}'
    ```

- Backend microservices hook endpoitns

    ```bash
     oc get routes -n app-be-cicd be-pipeline-webhook-dev -o jsonpath='{.spec.host}'
     oc get routes -n app-be-cicd be-pipeline-webhook-qa -o jsonpath='{.spec.host}'
     ```