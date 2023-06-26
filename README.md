# Overview

Synthesized Scientific Data Kit (SDK) is a comprehensive framework for generative modelling for structured data (tabular, time-series and event-based data). The SDK helps you create compliant statistical-preserving data snapshots for BI/Analytics and ML/AI applications. Right-size your data with AI-supported data transformations.

This delivery contains the SDK bundled up with a Jupyter notebook. This provides an easy platform to start working with synthetic data.

Available on the GCP Cloud Marketplace: https://console.cloud.google.com/marketplace/product/synthesized-public/synthesized-sdk-notebook-byol

# Installation

## Quick install with Google Cloud Marketplace

To install Synthesized SDK Jupyter Notebook to a Google Kubernetes Engine cluster via Google Cloud Marketplace, follow the
[on-screen instructions](https://console.cloud.google.com/marketplace/product/synthesized-public/synthesized-sdk-notebook-byol).

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
export APP_INSTANCE_NAME=sdk-jupyter-server
export NAMESPACE=default
```

Set up the image tag.
Example:

```shell
export TAG="2.7.0"
```

Configure the container images:

```shell
export IMAGE_REGISTRY="gcr.io/synthesized-public/sdk-jupyter-server"
```

Configure the Synthesized licence key:

```shell
export SYNTHESIZED_KEY=[YOUR KEY]
```

(Optional) Set computation resources limit:

```shell
export RESOURCES_LIMITS_CPU=1
export RESOURCES_LIMITS_MEMORY=1Gi
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
helm template chart/sdk-jupyter-server \
  --name-template "${APP_INSTANCE_NAME}" \
  --namespace "${NAMESPACE}" \
  --set envRenderSecret.SYNTHESIZED_KEY "${SYNTHESIZED_KEY}" \
  --set image.repository="${IMAGE_REGISTRY}" \
  --set image.tag="${TAG}" \
  --set resources.limits.cpu="${RESOURCES_LIMITS_CPU}" \
  --set resources.limits.memory="${RESOURCES_LIMITS_MEMORY}" \
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

You can expose Jupyter Web Server port:

```shell
kubectl port-forward \
    --namespace "${NAMESPACE}" \
    svc/${APP_INSTANCE_NAME}-service \
    8888:8888
```

# Using the app

## How to use Synthesized Jupyter Notebook

* Open the demo notebook or create a new notebook
* Inside the notebook, if you did not set your license key earlier, ensure you set the license key by adding the following at the top: `import os; os.environ["SYNTHESIZED_KEY"] = <INSERT_KEY_HERE>`
* Now synthesized can be imported into the notebook, data can be loaded, a synthesizer trained, and synthetic data generated as explained in the quickstart guide in [Synthesized’s docs](https://docs.synthesized.io/sdk/latest/getting_started/quickstarts/tabular)
* After data has been generated, it can be saved to a permanent location using standard python libraries and functions. To save files to gcp for example follow instructions [here](https://cloud.google.com/appengine/docs/legacy/standard/python/googlecloudstorageclient/read-write-to-cloud-storage)


# Scaling

This is a single-instance version of SDK Jupyter Notebook. It is not intended to be scaled
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
export APP_INSTANCE_NAME=sdk-jupyter-server
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
kubectl delete application,deployment,secret,service,backendconfig \
  --namespace ${NAMESPACE} \
  --selector name=${APP_INSTANCE_NAME}
```
