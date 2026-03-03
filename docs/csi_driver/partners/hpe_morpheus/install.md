# Introduction

HPE Kubernetes Service (HKS) part of HPE Morpheus Enterprise allows customers to deploy and manage Kubernetes clusters through the Morpheus hybrid cloud management platform. Since Morpheus uses a standard Linux distribution and upstream Kubernetes, the solution is fully supported by HPE CSI Driver for Kubernetes.

Familiarize yourself on how to install a [HPE Kubernetes Service](https://www.hpe.com/support/morpheus-enterprise-documentation-latest) cluster on your infrastructure

[TOC]

!!! tip "Brownfield Managed Clusters"
    Clusters that have been deployed prior to being managed by Morpheus are subject to qualification using the [Compatibility and Support](../../index.md#latest_release) matrix. Both the host OS and Kubernetes distribution needs to be supported.

## Installation

Users may deploy the HPE CSI Driver for Kubernetes on the managed cluster with their preferred method. HPE strongly recommend using the Helm chart.

- Visit [ArtifactHub.io](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) for installation instructions for the Helm chart.
- Instructions how to [install via the Operator-managed](../../deployment.md#operator) Helm chart.

### Next Steps

Once the CSI driver is installed, a `Secret` and a `StorageClass` is needed to provision `PersistentVolumes`.

- [Add an HPE storage backend](../../deployment.md#add_an_hpe_storage_backend).
- [Create a base `StorageClass`](../../using.md#base_storageclass_parameters).

Optionally, since HKS installs CEPH, the "rook-ceph-block" `StorageClass` needs to be disabled as the default `StorageClass`.

```text
kubectl annotate --overwrite storageclass/rook-ceph-block storageclass.kubernetes.io/is-default-class=false
```

## Known Issues and Limitations

All most recent configurations will most likely work and be supported by HPE. Here are some of the current limitations and issues.

- Morpheus allows users to deploy and manage Kubernetes on AWS. The logical choice for storage would be [HPE GreenLake Block Storage for AWS](https://aws.amazon.com/marketplace/pp/prodview-rvhlswizjagfs) but the HPE CSI Driver for Kubernetes is not yet supported with the storage platform.
