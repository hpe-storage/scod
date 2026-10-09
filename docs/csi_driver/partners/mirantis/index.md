# Introduction

Mirantis Kubernetes Engine (MKE) is the successor of the Universal Control Plane part of Docker Enterprise Edition (Docker EE). The HPE CSI Driver for Kubernetes allows users to provision persistent storage for Kubernetes workloads running on MKE. See the note below on [Docker Swarm](#docker_swarm) for workloads deployed outside of Kubernetes.

[TOC]

## Compatability Chart

Mirantis and HPE perform testing and qualification as needed for either release of MKE or the HPE CSI Driver. If there are any deviations in the installation procedures, those will be documented here.

!!! important
    Always ensure the MKE version of the underlying Kubernetes version and worker node host OS conforms to the latest [compatability and support](../../index.md#compatibility_and_support) table.

| MKE Version | HPE CSI Driver | Status | Installation Notes | 
| ------------| -------------- | ------ | ------------------ |
| 4.x         | [2.5.2](../../archive.md#hpe_csi_driver_for_kubernetes_252) to [latest](../../index.md#latest_release) | Supported | k0s [notes](#k0s_and_k0rdent_considerations) |
| 3.x         | [2.5.2](../../archive.md#hpe_csi_driver_for_kubernetes_252) | Supported | Helm chart [notes](#helm_chart_install) |

!!! seealso
    Ensure to be understood with the [limitations](#limitations) and the lack of [Docker Swarm](#docker_swarm) support.

### k0s and k0rdent Considerations

MKE 4.0 onwards is based on the k0s Kubernetes distribution and managed by k0rdent. Depending on how k0s was installed, the kubelet root directory may differ. The HPE CSI Driver for Kubernetes Helm chart needs the the correct path passed to the chart with the `kubeletRootDir` parameter.

- If the clusters are deployed with `mkectl`, use `--set kubeletRootDir=/var/lib/k0s/kubelet`
- If upstream k0s is deployed standalone, use `--set kubeletRootDir=/varlib/k0s/kubelet`

Unless there are any special circumstances it's recommended to deploy the CSI driver through the k0rdent catalog as a `MultiClusterService` from a `ServiceTemplate`.

- Visit the HPE CSI Driver for Kubernetes in the [k0rdent catalog](https://catalog.k0rdent.io/latest/apps/hpe-csi/).

If there are special needs, proceed to the [Helm Chart Install](#helm_chart_install).

!!! note
    Both k0rdent `ServiceTemplate` and Helm chart install are supported by HPE.

### Helm Chart Install

With MKE 3.6 and 3.9, it's recommend to use the HPE CSI Driver for Kubernetes Helm chart. There are no known caveats or workarounds at this time.

- [HPE CSI Driver for Kubernetes Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) on ArtifactHub.

## NFS Server Provisioner on MKE 3.9 and earlier

In order to allow the HPE CSI Driver to deploy privileged NFS servers in the default NFS `Namespace` of "hpe-nfs", the MKE configuration file needs to be updated with the following configuration directly inside the `[cluster_config]` stanza:

```text
[cluster_config]
  priv_attributes_allowed_for_service_accounts = ["kernelCapabilities", "privileged"]
  priv_attributes_service_accounts = ["hpe-nfs:hpe-csi-nfs-sa"]
```

Configuring a `StorageClass` with `.parameters.nfsNamespace: csi.storage.k8s.io/pvc/namespace` or any custom `Namespace` would require all `ServiceAccounts` to be enumerated in "priv_attributes_service_accounts" above.

!!! important "How do I update the MKE configuration file?"
    Updating the MKE configuration file requires administrative privileges and access to the control plane. See the Mirantis documentation for more details.

    * [MKE 3.9](https://docs.mirantis.com/mke/3.9/ops/administer-cluster/configure-an-mke-cluster/use-an-mke-configuration-file.html#modify-an-existing-mke-configuration)
    * [MKE 3.8](https://docs.mirantis.com/mke/3.8/ops/administer-cluster/configure-an-mke-cluster/use-an-mke-configuration-file.html#modify-an-existing-mke-configuration)
    * [MKE 3.7](https://docs.mirantis.com/mke/3.7/ops/administer-cluster/configure-an-mke-cluster/use-an-mke-configuration-file.html#modify-an-existing-mke-configuration)

    Versions prior to MKE 3.6 are untested but are supported if the NFS servers come up.

## Docker Swarm

Provisioning Docker Volumes for Docker Swarm workloads from a HPE primary storage backend is deprecated.

## Limitations

- HPE CSI Driver does not support Windows workers.
