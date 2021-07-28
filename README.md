# OpenShift CICD Demo

Christian Hernandez, Mon 3:05 PM, Edited
helm uninstall the helm install blah blah


> :warning: Although in working order, this should be
> considered more of a "POC" for the time being. You can [watch the video](http://www.youtube.com/watch?v=LLYNn0kieOg)
> if you're just looking for a quickstart.

This is a "self contained" and "self bootstrapping" OpenShift CI/CD Demo,
that uses OpenShift Pipelines (Tekton) in tandem with OpenShift GitOps
(Argo CD) to deliver a cloud native CI/CD solution.

Once deployed, you will have the following:

* OpenShift GitOps cluster scoped deployment installed
* Tekton installed
* Custom Cluster configurations
  * This includes creating a `developer` user.

The OpenShift GitOps system will then install the following:

* "Tenant" Argo CD instance under the `welcome-gitops` namespace
* A GitTea instances to house all the manifests
  * It also imports all the needed repos
* Sample `welcome-app` application in the following namespaces
  * `welcome-dev`
  * `welcome-prod`
* Tekton Pipeline under `welcome-pipeline` set up with the following
  * Builds application from source (hosted on the GitTea service)
  * Image gets stored in the internal registry
  * Tekton will "write back" to the GitTea instance for ArgoCD to act on

# Installation

> :warning: I've tested this on OCP 4.7

To deploy this demo, you need to have the following

1. An "empty" OpenShift 4.7 cluster installed
2. Cluster Admin access to the cluster
3. `oc`, `kubectl`, and `helm` CLI installed.
4. Not required; but `kustomize` is useful (for debugging)

Once you have a cluster ready, login as `cluster-admin` and add the
following helm repo (and update the defs):

```shell
helm repo add redhat-demos https://redhat-developer-demos.github.io/helm-repo
helm repo update
```

Install deploy this repo with the following command

```shell
helm install ocpcicd redhat-demos/openshift-cicd-demo
```

If you have issues, please see the [troubleshooting section](#troubleshooting)

# Demo Installation

Once deployed, you will need the following information.

## Console

Console route can be found using:

```shell
oc get route console -n openshift-console -o jsonpath='{.spec.host}{"\n"}'
```

Use the following credentials

* username: `developer`
* password: `openshift`

## Tenant Argo CD

The tenant Argo CD instance is running in the `welcome-gitops`
namespace. To reach the Web UI:

```shell
oc get route welcome-argocd-server -n welcome-gitops -o jsonpath='{.spec.host}{"\n"}'
```

The password for the tenant Argo CD instance can be extracted by:

```shell
oc extract secret/welcome-argocd-cluster -n welcome-gitops --to=-
```

## GitTea

The Git service route can be found by:

```shell
oc get route gitea -n scm -o jsonpath='{.spec.host}{"\n"}'
```

Use the following credentials for the demo:

* username: `developer`
* password: `openshift`

You will see two repos; `welcome-app` and `welcome-deploy`. 

* `welcome-app` repo is the code repo that builds the application
* `welcome-deploy` repo is the repo with the deployment manifest.


## Welcome App

The application is deployed to two namespaces: `welcome-dev` and `welcome-prod`

You can find the routes using the following:

Dev

```shell
oc get route welcome-app -n welcome-dev -o jsonpath='{.spec.host}{"\n"}'
```

Prod

```shell
oc get route welcome-app -n welcome-prod -o jsonpath='{.spec.host}{"\n"}'
```

# Running the Demo

Make a commit to the `index.php` file at the following URL:

```shell
echo $(oc get route gitea -n scm -o jsonpath='{.spec.host}')/developer/welcome-app
```

This should fire off a build that you can see progress in the
`welcome-pipeline` namespace. It's probably easier to just show you, so
watch this video:

[![Running the Demo Video](http://img.youtube.com/vi/LLYNn0kieOg/0.jpg)](http://www.youtube.com/watch?v=LLYNn0kieOg)


# Troubleshooting

Common issues that may arise are in this section in no particular order.

## Installation

If you don't want to use the helm chart, you can deploy this repo directly
by running the following:

> **NOTE** If using OCP 4.7, you __HAVE__ to use `kubectl`

```shell
until kubectl apply -k https://github.com/RedHatWorkshops/openshift-cicd-demo/bootstrap/overlays/base.cluster/
do
  sleep 5
done
```

For more information about the helm repo, visit the [chart repo](https://github.com/redhat-developer-demos/helm-repo/tree/main/stable/openshift-cicd-demo)

## OpenShift GitOps

The OpenShift GitOps installation (aka the "Cluster Argo CD) can be
reached by the following route:

```shell
oc get route openshift-gitops-server -n openshift-gitops -o jsonpath='{.spec.host}{"\n"}'
```

The Admin password can be found by:

```shell
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

## GitTea Admin

In case you need it, the GitTea Admin credentials:

* giteaAdminUser: `gitea-admin`
* giteaAdminPassword: `openshift`
