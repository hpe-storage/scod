# Introduction

Mirantis Kubernetes Engine (MKE) is the successor of the Universal Control Plane part of Docker Enterprise Edition (Docker EE). The HPE CSI Driver for Kubernetes allows users to provision persistent storage for Kubernetes workloads running on MKE. See the note below on [Docker Swarm](#docker_swarm) for workloads deployed outside of Kubernetes.

[TOC]

## Compatability Chart

Mirantis and HPE perform testing and qualification as needed for either release of MKE or the HPE CSI Driver. If there are any deviations in the installation procedures, those will be documented here.

!!! important
    Always ensure the MKE version of the underlying Kubernetes version and worker node host OS conforms to the latest [compatability and support](../../index.md#compatibility_and_support) table.

| MKE Version | HPE CSI Driver | Status | Installation Notes | 
| ------------| -------------- | ------ | ------------------ |
| 4.0         | [2.5.2](../../index.md#hpe_csi_driver_for_kubernetes_252) | Supported | k0s [notes](#k0s_and_k0rdent_considerations) |
| 3.8         | [2.5.2](../../index.md#hpe_csi_driver_for_kubernetes_252) | Supported | Helm chart [notes](#helm_chart_install) |
| 3.7         | [2.4.0](../../archive.md#hpe_csi_driver_for_kubernetes_240) | Supported | Helm chart [notes](#helm_chart_install) |
| 3.6         | [2.2.0](../../archive.md#hpe_csi_driver_for_kubernetes_220) | Deprecated | Helm chart [notes](#helm_chart_install) |
| 3.4, 3.5    | -              | Untested | - |
| 3.3         | [2.0.0](../../archive.md#hpe_csi_driver_for_kubernetes_200) | Deprecated | Advanced Install [notes for MKE 3.3 ](#mirantis_kubernetes_engine_33) | 

!!! seealso
    Ensure to be understood with the [limitations](#limitations) and the lack of [Docker Swarm](#docker_swarm) support.

### k0s and k0rdent Considerations

MKE 4.0 onwards is based on the k0s Kubernetes distribution and managed by k0rdent. Depending on how k0s was installed, the kubelet root directory may differ. The HPE CSI Driver for Kubernetes Helm chart needs the the correct path passed to the chart with the `kubeletRootDir` parameter.

- If the clusters are deployed with `mkectl`, use `--set kubeletRootDir=/var/lib/k0s/kubelet`
- If upstream k0s is deployed standalone, use `--set kubeletRootDir=/varlib/k0s/kubelet`

Unless there are any special circumstances it's recommended to deploy the CSI driver through the k0rdent catalog as a `MultiClusterService` from a `ServiceTemplate`.

- Visit the HPE CSI Driver for Kubernetes in the [k0rdent catalog](https://catalog.k0rdent.io/v0.3.0/apps/hpe-csi/).

If there are special needs, proceed to the [Helm Chart Install](#helm_chart_install).

!!! note
    Both k0rdent `ServiceTemplate` and Helm chart install are supported by HPE.

### Helm Chart Install

With MKE 3.6 and onwards, it's recommend to use the HPE CSI Driver for Kubernetes Helm chart. There are no known caveats or workarounds at this time.

- [HPE CSI Driver for Kubernetes Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) on ArtifactHub.

### Mirantis Kubernetes Engine 3.3

At the time of release of MKE 3.3, neither of the HPE CSI Driver Helm chart or operator will install correctly.

#### Prerequisites

The MKE managers and workers needs to run a supported host OS as outlined in the particular version of the HPE CSI Driver found in the [release tables](../../index.md#compatibility_and_support). Also verify that the HPE CSI Driver support the version Kubernetes used by MKE (see below).

#### Steps to install

MKE admins needs to familiarize themselves with the [advanced install](../../deployment.md#advanced_install) method of the CSI driver. Before the installation begins, make sure an account with administrative privileges is being used to the deploy the driver. Also determine the actual Kubernetes version MKE is using. 

```text
kubectl version --short true
Client Version: v1.19.4
Server Version: v1.18.10-mirantis-1
```

In this particular example, Kubernetes 1.18 is being used. Follow the steps for 1.18 highlighted within the [advanced install](../../deployment.md#common) section of the deployment documentation.

- **Step 1** → Install the Linux node IO settings `ConfigMap`.
- **Step 2** → Determine which backend being used (Nimble or Primera/3PAR) and deploy the corresponding CSP manifest.
- **Step 3** → Deploy the HPE CSI Driver manifests for the Kubernetes version being used.

Next, add a [supported HPE backend](../../deployment.md#add_an_hpe_storage_backend) and [create a `StorageClass`](../../using.md#base_storageclass_parameters).

Learn more about using the CSI objects in [the comprehensive overview](../../using.md). Also make sure to familiarize yourself with the particular features and capabilities of the backend being used.

- [Container Storage Providers](../../container_storage_provider/index.md)

## NFS Server Provisioner on MKE 3.8 and earlier

In order to allow the HPE CSI Driver to deploy privileged NFS servers in the default NFS `Namespace` of "hpe-nfs", the MKE configuration file needs to be updated with the following configuration directly inside the `[cluster_config]` stanza:

```text
[cluster_config]
  priv_attributes_allowed_for_service_accounts = ["kernelCapabilities", "privileged"]
  priv_attributes_service_accounts = ["hpe-nfs:hpe-csi-nfs-sa"]
```

Configuring a `StorageClass` with `.parameters.nfsNamespace: csi.storage.k8s.io/pvc/namespace` or any custom `Namespace` would require all `ServiceAccounts` to be enumerated in "priv_attributes_service_accounts" above.

!!! important "How do I update the MKE configuration file?"
    Updating the MKE configuration file requires administrative privileges and access to the control plane. See the Mirantis documentation for more details.

    * [MKE 3.8](https://docs.mirantis.com/mke/3.8/ops/administer-cluster/configure-an-mke-cluster/use-an-mke-configuration-file.html#modify-an-existing-mke-configuration)
    * [MKE 3.7](https://docs.mirantis.com/mke/3.7/ops/administer-cluster/configure-an-mke-cluster/use-an-mke-configuration-file.html#modify-an-existing-mke-configuration)
    * [MKE 3.6](https://docs.mirantis.com/mke/3.6/ops/administer-cluster/configure-an-mke-cluster/use-an-mke-configuration-file.html#modify-an-existing-mke-configuration)

    Versions prior to MKE 3.6 are untested but are supported if the NFS servers come up.

## Docker Swarm

Provisioning Docker Volumes for Docker Swarm workloads from a HPE primary storage backend is deprecated.

## Limitations

- HPE CSI Driver does not support Windows workers.
