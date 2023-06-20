# Overview

Synthesized SDK generates high quality, privacy-preserving datasets for machine learning and data science use cases.

[//]: # (Available on the GCP Cloud Marketplace: TODO)

## Architecture

[//]: # (TODO)

# Installation

## Quick install with Google Cloud Marketplace

To install Synthesized SDK to a Google

[//]: # (Kubernetes Engine cluster via Google Cloud Marketplace, follow these instructions: TODO)

## Command-line instructions

### Prerequisites

#### Setting up command-line tools

You need the following tools in your development environment:

- [gcloud](https://cloud.google.com/sdk/gcloud/)
- [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
- [Docker](https://docs.docker.com/install/)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Helm](https://helm.sh/)

Configure `gcloud` as a Docker credential helper:

```shell
gcloud auth configure-docker
```

#### Creating a Google Kubernetes Engine (GKE) cluster

Create a new cluster from the command line:

```shell
export CLUSTER=sdk-cluster
export ZONE=us-west1-a

gcloud container clusters create "${CLUSTER}" --zone "${ZONE}"
```

Configure `kubectl` to connect to the new cluster:

```shell
gcloud container clusters get-credentials "${CLUSTER}" --zone "${ZONE}"
```

#### Cloning this repo

Clone this repo, as well as its associated tools repo:

```shell
git clone --recursive https://github.com/synthesized-io/sdk-on-gcp.git
```

#### Installing the Application resource definition

An Application resource is a collection of individual Kubernetes
components, such as Services, Deployments, and so on, that you can
manage as a group.

To set up your cluster to understand Application resources, run the
following command:

```shell
kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
```

You need to run this command once.

The Application resource is defined by the
[Kubernetes SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps)
community. You can find the source code at
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

### Installing the app

#### Configuring the app with environment variables

Choose an instance name and
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
for the app. In most cases, you can use the `default` namespace.

```shell
export APP_INSTANCE_NAME=synthesized-sdk
export NAMESPACE=default
```

Set up the image tag.
Example:

```shell
export TAG="2.3"
```

Configure the container images:

```shell
#TODO
#export IMAGE_REGISTRY="marketplace.gcr.io/google"
```

#### Creating namespace in your Kubernetes cluster

If you use a different namespace than the `default`, create a new namespace by
running the following command:

```shell
kubectl create namespace "${NAMESPACE}"
```

#### Creating the Service Account

To create the Service Account and ClusterRoleBinding:

```shell
export SDK_SERVICE_ACCOUNT="${APP_INSTANCE_NAME}-serviceaccount"
kubectl create serviceaccount "${SDK_SERVICE_ACCOUNT}" --namespace "${NAMESPACE}"
kubectl create clusterrole "${SDK_SERVICE_ACCOUNT}-role" --verb=get,list,watch --resource=services,nodes,pods,namespaces
kubectl create clusterrolebinding "${SDK_SERVICE_ACCOUNT}-rule" --clusterrole="${SDK_SERVICE_ACCOUNT}-role" --serviceaccount="${NAMESPACE}:${SDK_SERVICE_ACCOUNT}"
```

#### Expanding the manifest template

Use `helm template` to expand the template. We recommend that you save the
expanded manifest file for future updates to your app.

```shell
helm template chart/synthesized-sdk \
  --name "${APP_INSTANCE_NAME}" \
  --namespace "${NAMESPACE}" \
  --set envRenderSecret.SYNTHESIZED_KEY "${SYNTHESIZED_KEY}" \
  --set image.repository="${SYNTHESIZED_IMAGE}" \
  --set image.tag="${SYNTHESIZED_TRACK}" \
  > "${APP_INSTANCE_NAME}_manifest.yaml"
```

#### Applying the manifest to your Kubernetes cluster

To apply the manifest to your Kubernetes cluster, use `kubectl`:

```shell
kubectl apply -f "${APP_INSTANCE_NAME}_manifest.yaml" --namespace "${NAMESPACE}"
```

#### Viewing your app in the Google Cloud Console

To get the Cloud Console URL for your app, run the following command:

```shell
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

To view the app, open the URL in your browser.

### Accessing the User Interface

To get the external IP address of SDK, use the following
command:

[//]: # (TODO)

[//]: # (```shell)

[//]: # (```)

# Scaling

This is a single-instance version of SDK. It is not intended to be scaled
up with its current configuration.

# App metrics

At the moment, the application does not support exporting Prometheus metrics and does not have any exporter.

# Uninstalling the app

## Using the Google Cloud Console

1.  In the Cloud Console, open
    [Kubernetes Applications](https://console.cloud.google.com/kubernetes/application).

2.  From the list of apps, choose your app installation.

3.  On the **Application Details** page, click **Delete**.

## Using the command-line

### Preparing your environment

Set your installation name and Kubernetes namespace:

```shell
export APP_INSTANCE_NAME=synthesized-sdk
export NAMESPACE=default
```

### Deleting your resources

> **NOTE:** We recommend using a `kubectl` version that is the same as the
> version of your cluster. Using the same version for `kubectl` and the cluster
> helps to avoid unforeseen issues.

#### Deleting the deployment with the generated manifest file

Run `kubectl` on the expanded manifest file:

```shell
kubectl delete -f ${APP_INSTANCE_NAME}_manifest.yaml --namespace ${NAMESPACE}
```

#### Deleting the deployment by deleting the Application resource

If you don't have the expanded manifest file, delete the resources by using
types and a label:

```shell
kubectl delete application,deployment,secret,service,ingress \
  --namespace ${NAMESPACE} \
  --selector name=${APP_INSTANCE_NAME}
```

# Upgrading the app

## Preparing your environment

The steps below describe the upgrade procedure with the new version of the
Synthesized SDK Docker image.

Set your environment variables to match the installation properties:

```shell
export APP_INSTANCE_NAME=synthesized-sdk
export NAMESPACE=default
```

## Upgrading your app

[//]: # (TOOD)
