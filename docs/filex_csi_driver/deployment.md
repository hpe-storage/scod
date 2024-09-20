# Overview

The HPE GreenLake for File Storage CSI Driver is deployed by using industry standard means, either a Helm chart or an Operator.

[TOC]

## Helm

[Helm](https://helm.sh) is the package manager for Kubernetes. Software is being delivered in a format designated as a "chart". Helm is a [standalone CLI](https://helm.sh/docs/intro/install/) that interacts with the Kubernetes API server using your `KUBECONFIG` file.

The official Helm chart for the HPE GreenLake for File Storage CSI Driver is hosted on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-greenlake-file-csi-driver). In an effort to avoid duplicate documentation, please see the chart for instructions on how to deploy the CSI driver using Helm.

- Go to the chart on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-greenlake-file-csi-driver).

!!! note
    It's possible to use the HPE CSI Driver for Kubernetes steps for v2.4.2 or later to mirror the required images to an internal registry for installing into an [air-gapped environment](../csi_driver/deployment.md#helm_for_air-gapped_environments).

## Operator

The [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) is based on the idea that software should be instantiated and run with a set of custom controllers in Kubernetes. It creates a native experience for any software running on Kubernetes.

### Red Hat OpenShift Container Platform

<!--
The HPE GreenLake for File Storage CSI Operator is a fully certified Operator for OpenShift. There are a few tweaks needed and there's a separate section for OpenShift.

- See [Red Hat OpenShift](../partners/redhat_openshift/index.md) in the partner ecosystem section
-->
During the beta, it's only possible to sideload the HPE GreenLake for File Storage CSI Operator using the Operator SDK.

The installation procedures assumes the "hpe-storage" `Namespace` exists:

```text
oc create ns hpe-storage
```

<div id="scc" />First, deploy or [download]({{ config.site_url}}partners/redhat_openshift/examples/scc/hpe-filex-csi-scc.yaml) the SCC:

```text
oc apply -f {{ config.site_url}}partners/redhat_openshift/examples/scc/hpe-filex-csi-scc.yaml
```

Install the Operator:

```text
operator-sdk run bundle --timeout 5m -n hpe-storage quay.io/hpestorage/filex-csi-driver-operator-bundle-ocp:v1.0.0-beta
```

The next step is to create a `HPEGreenLakeFileCSIDriver` resource, this can also be done in the OpenShift cluster console.

```yaml fct_label="HPE GreenLake for File Storage CSI Operator v1.0.0-beta"
# oc apply -f {{ config.site_url }}filex_csi_driver/examples/deployment/hpegreenlakefilecsidriver-v1.0.0-beta-sample.yaml
{% include "examples/deployment/hpegreenlakefilecsidriver-v1.0.0-beta-sample.yaml" %}```

For reference, this is how the Operator is uninstalled:

```text
operator-sdk cleanup hpe-filex-csi-operator -n hpe-storage
```

## Add a Storage Backend

Once the CSI driver is deployed, two additional resources need to be created to get started with dynamic provisioning of persistent storage, a `Secret` and a `StorageClass`.

!!! tip
    Naming the `Secret` and `StorageClass` is entirely up to the user, however, to keep up with the examples on SCOD, it's highly recommended to use the names illustrated here.

### Secret Parameters

All parameters are mandatory and described below.

| Parameter   | Description |
| ----------- | ----------- |
| endpoint    | This is the management hostname or IP address of the actual backend storage system. |
| username    | Backend storage system username with the correct privileges to perform storage management. |
| password    | Backend storage system password. |

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: hpe-file-backend
  namespace: hpe-storage
stringData:
  endpoint 192.168.1.1
  username: my-csi-user
  password: my-secret-password
```

Create the `Secret` using `kubectl`:

```text
kubectl create -f secret.yaml
```

!!! tip
    In a real world scenario it's more practical to name the `Secret` something that makes sense for the organization. It could be the hostname of the backend or the role it carries, i.e "hpe-greenlake-file-sanjose-prod".

Next step involves [creating a default StorageClass](using.md#base_storageclass_parameters).
