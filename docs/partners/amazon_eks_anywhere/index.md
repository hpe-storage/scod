
# Overview

Amazon Elastic Kubernetes Service (EKS) Anywhere allows customers to deploy Amazon EKS-D (Amazon Elastic Kubernetes Service Distro) on their private or non-AWS clouds. AWS users familiar with the ecosystem gain the ability to cross clouds and manage their Kubernetes estate in a single pane of glass. 

This documentation outlines the limitations and considerations when using the HPE CSI Driver for Kubernetes when deployed on EKS-D.

[TOC]

## Limitations

These limitations may be expanded or detracted in future releases of either Amazon EKS Anywhere or HPE CSI Driver.

### Bottlerocket OS

The default Linux distribution AWS favors is Bottlerocket OS which is a container-optimized distribution. Due to the slim host library and binary surface, Bottlerocket OS does not include the necessary utilities to support SAN storage. This limitation can be tracked in [this GitHub issue](https://github.com/bottlerocket-os/bottlerocket/issues/2570).

!!! note
    Any other OS supported by EKS-A and is listed in the [Compatibility and Support table](../../csi_driver/index.md#compatibility_and_support) is supported by the HPE CSI Driver.

### EKS Anywhere on vSphere

Only iSCSI is supported as the HPE CSI Driver does not support NPIV which is required for virtual Fibre Channel host bus adapters (HBA). More information on this limitation is elaborated on in the [VMware section](../vmware/index.md#deployment) on SCOD.

Due to `VSphereMachineConfig` VM templates only allow a single vNIC, no multipath redundancy is available to the host. Ensure network fault tolerance according to VMware best practices is available to the VM. Also keep in mind that the backend storage system needs to have a data interface in the same subnet as the HPE CSI Driver will not try to discover targets over routed networks.

!!! tip
    The vSphere CSI Driver and HPE CSI Driver may co-exist in the same cluster but make sure there's only one default `StorageClass` configured before creating `PersistentVolumeClaims`. Please see the [official Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/) on how to change the default `StorageClass`.

## Installation Considerations

EKS-D is a CNCF compliant Kubernetes distribution and no special steps are required to deploy the HPE CSI Driver for Kubernetes. It's crucial to ensure that the compute nodes run a supported OS and the version of Kubernetes is supported by the HPE CSI Driver. Check the [Compatibility and Support table](../../csi_driver/index.md#compatibility_and_support) for more information.

Proceed to installation documentation:

- HPE CSI Driver for Kubernetes Helm chart on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) (recommended)
- HPE CSI Operator for Kubernetes on [OperatorHub.io](https://operatorhub.io/operator/hpe-csi-operator)
- [Advanced Install](../../csi_driver/deployment.md#advanced_install) using YAML manifests
