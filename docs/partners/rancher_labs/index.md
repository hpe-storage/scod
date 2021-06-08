# Overview

Rancher Labs provides a platform to deploy Kubernetes-as-a-service everywhere. HPE partners with Rancher Labs to provide effortless management of the CSI and FlexVolume driver on managed Kubernetes clusters. This allows our joint customers and channel partners to enable hybrid cloud stateful workloads on Kubernetes.

[TOC]

## Deployment considerations

Rancher is capable of managing Kubernetes across a broad spectrum of managed and BYO clusters. It's important to understand that the HPE CSI Driver for Kubernetes does not support the same amount of combinations Rancher does. Consult the support matrix on [the CSI driver overview page](../../csi_driver/index.md#compatibility_and_support) for the supported combinations of the HPE CSI Driver, Kubernetes and supported node Operating Systems.

## Supported versions

Rancher uses Helm to deploy and manage partner software. The concept of a Helm repository in Rancher is organized as a "catalog". The HPE CSI Driver for Kubernetes and the HPE Volume Driver for Kubernetes FlexVolume plugin are both partner solutions present in the official Rancher Catalog.

| Rancher release | Install methods                   | Recommended CSI/FlexVolume driver |
| --------------- | --------------------------------- | --------------------------------- |
| 2.3             | Cluster Manager                   | latest                            |
| 2.4             | Cluster Manager                   | latest                            |
| 2.5             | Cluster Manager, Cluster Explorer | latest                            |

!!! tip
    Learn more about Catalogs, Helm Charts and Apps in the [Rancher documentation](https://rancher.com/docs/rancher/v2.x/en/catalog/).

### HPE CSI Driver for Kubernetes

The HPE CSI Driver is part of the official Helm v3 library in Rancher. The CSI driver is deployed on managed Kubernetes clusters like any ordinary "App" in Rancher. You may use either the [Rancher CLI](https://rancher.com/docs/rancher/v2.x/en/cli/) or the web UI to deploy the CSI driver.

!!! note
    In Rancher 2.5 an "Apps & Marketplace" component was introduced in the new "Cluster Explorer" interface. This is the new interface moving forward. Upcoming releases of the HPE CSI Driver for Kubernetes will only support installation via "Cluster Explorer".

#### Rancher CLI install

Switch to the project you want to install the CSI driver. For this example, the default project on a managed cluster is being used.

```markdown
$ rancher context current
Cluster:torta Project:Default
```

Steps to install the CSI driver.

```markdown
$ rancher app install hpe-csi-driver hpe-csi-driver --no-prompt
run "app show-notes hpe-csi-driver" to view app notes once app is ready
$ rancher app
ID                       NAME             STATE     CATALOG         TEMPLATE         VERSION
p-k28xd:hpe-csi-driver   hpe-csi-driver   active    helm3-library   hpe-csi-driver   1.3.1
```

!!! note
    This is installs the driver with the default parameters which is the most common deployment option. Please see the official Helm chart [documentation](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) for supported parameters.

#### Rancher Cluster Explorer

In Rancher 2.5 and newer, the "Apps & Marketplace" in the "Cluster Explorer" may be used to install the HPE CSI Driver. This is recommended for new installs.

![](img/cluster_explorer.png)
<small>Rancher Cluster Explorer</small>

#### Rancher Cluster Manager

In Rancher 2.5 and earlier, the "Apps" interface is the default method of installing the HPE CSI Driver.

![](img/cluster_manager.png)
<small>Rancher Cluster Manager</small>

!!! note
    Installing the CSI driver with default parameters (simply hit "Launch" in the UIs) is the most common deployment option. Please see the official Helm chart [documentation](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) for supported parameters.

#### Post install steps

For Rancher Apps to make use of persistent storage from HPE, a supported backend needs to be configured along with a `StorageClass`. These procedures are generic regardless of Kubernetes distribution being used.

- Go ahead and [add an HPE storage backend](../../csi_driver/deployment.md#add_an_hpe_storage_backend)

### HPE Volume Driver for Kubernetes FlexVolume plugin

Only use the FlexVolume driver for Kubernetes 1.12 and below. The FlexVolume driver is provided as a Helm v2 chart in the official Rancher Catalog. Parameters are very specific to the environment to where the driver is being installed to. Please follow the steps in the FlexVolume Helm chart [documentation](https://artifacthub.io/packages/helm/hpe-storage/hpe-flexvolume-driver) for further guidance. Also understand that the FlexVolume driver only supports HPE Nimble Storage and HPE Cloud Volumes.

!!! caution
    The FlexVolume driver is being deprecated. Reach out to your HPE representative if you think deploying the FlexVolume driver on your Rancher managed Kubernetes cluster is the correct course of action.
