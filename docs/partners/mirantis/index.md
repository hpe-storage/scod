# Introduction

Mirantis Kubernetes Engine (MKE) is the successor of the Universal Control Plane part of Docker Enterprise Edition (Docker EE). The HPE CSI Driver for Kubernetes allows users to provision persistent storage for Kubernetes workloads running on MKE. See the note below on [Docker Swarm](#docker_swarm) for workloads deployed outside of Kubernetes.

[TOC]

## Compatability Chart

Mirantis and HPE perform testing and qualification as needed for either release of MKE or the HPE CSI Driver. If there are any deviations in the installation procedures, those will be documented here.

| MKE Version | HPE CSI Driver | Status | Installation Notes | 
| ------------| -------------- | ------ | ------------------ |
| 3.6         | [2.2.0](../../csi_driver/index.md#hpe_csi_driver_for_kubernetes_220) | Supported | Helm chart [notes](#helm_chart_install) |
| 3.4, 3.5    | -              | Untested | - |
| 3.3         | [2.0.0](../../csi_driver/index.md#hpe_csi_driver_for_kubernetes_200) | Deprecated | Advanced Install [notes for MKE 3.3 ](#mirantis_kubernetes_engine_33) | 

!!! seealso
    Ensure to be understood with the [limitations](#limitations) and the lack of [Docker Swarm](#docker_swarm) support.

### Helm Chart Install

With MKE 3.6 and onwards, it's recommend to use the HPE CSI Driver for Kubernetes Helm chart. There are no known caveats or workarounds at this time.

- [HPE CSI Driver for Kubernetes Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) on ArtifactHub.

!!! important
    Always ensure the MKE version of the underlying Kubernetes version and worker node host OS conforms to the latest [compatability and support](../../csi_driver/index.md#compatibility_and_support) table.

### Mirantis Kubernetes Engine 3.3

At the time of release of MKE 3.3, neither of the HPE CSI Driver Helm chart or operator will install correctly.

#### Prerequisites

The MKE managers and workers needs to run a supported host OS as outlined in the particular version of the HPE CSI Driver found in the [release tables](../../csi_driver/index.md#compatibility_and_support). Also verify that the HPE CSI Driver support the version Kubernetes used by MKE (see below).

#### Steps to install

MKE admins needs to familiarize themselves with the [advanced install](../../csi_driver/deployment.md#advanced_install) method of the CSI driver. Before the installation begins, make sure an account with administrative privileges is being used to the deploy the driver. Also determine the actual Kubernetes version MKE is using. 

```text
kubectl version --short true
Client Version: v1.19.4
Server Version: v1.18.10-mirantis-1
```

In this particular example, Kubernetes 1.18 is being used. Follow the steps for 1.18 highlighted within the [advanced install](../../csi_driver/deployment.md#common) section of the deployment documentation.

- **Step 1** → Install the Linux node IO settings `ConfigMap`.
- **Step 2** → Determine which backend being used (Nimble or Primera/3PAR) and deploy the corresponding CSP manifest.
- **Step 3** → Deploy the HPE CSI Driver manifests for the Kubernetes version being used.

Next, add a [supported HPE backend](../../csi_driver/deployment.md#add_an_hpe_storage_backend) and [create a `StorageClass`](../../csi_driver/using.md#base_storageclass_parameters).

Learn more about using the CSI objects in [the comprehensive overview](../../csi_driver/using.md). Also make sure to familiarize yourself with the particular features and capabilities of the backend being used.

- [Container Storage Providers](../../container_storage_provider/index.md)

## Docker Swarm

Provisioning Docker Volumes for Docker Swarm workloads from a HPE primary storage backend is deprecated.

## Limitations

- HPE CSI Driver does not support Windows workers.
