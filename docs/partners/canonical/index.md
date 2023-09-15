## Overview

*"Canonical Kubernetes is pure upstream and works on any cloud, from bare metal to public and edge. Deploy single node and multi-node clusters with Charmed Kubernetes and MicroK8s to support container orchestration, from testing to production. Both distributions bring the latest innovations from the Kubernetes community within a week of upstream release, allowing for time to learn, experiment and upskill."*<sup>1</sup>

<div align="right"><small><sup>1</sup> = quote from <a href="https://ubuntu.com/kubernetes/">Canonical Kubernetes</a>.</small></div>
<br/>
HPE supports Ubuntu LTS releases along with recent upstream versions of Kubernetes for the HPE CSI Driver. As long as the CSI driver is installed on a [supported host OS](../../csi_driver/index.md#compatibility_and_support) with a CNCF certified Kubernetes distribution, the solution is supported.

Both Charmed Kubernetes on private cloud and MicroK8s for edge has been field tested with the HPE CSI Driver for Kubernetes by HPE.

[TOC]

### Charmed Kubernetes

Charmed Kubernetes is deployed with the Juju orchestration engine. Juju is capable of deploying and managing the full life-cycle of CNCF certified Kubernetes on various infrastructure providers, both private and public. Charmed utilize Ubuntu LTS for the node OS.

It's most relevant for HPE CSI Driver users when deployed on Canonical MAAS and VMware vSphere.

#### Notes on VMware vSphere

- It's important to keep in mind that only iSCSI is supported with the HPE CSI Driver on vSphere. If Fibre Channel is being used, consider deploying the [vSphere CSI Driver](../vmware/index.md) instead.
- When deploying Charmed Kubernetes with Juju, machines may only be deployed with a "primary-network" and an "external-network" option. The "primary-network" is used primarily for the Juju controller but may be dual purposed. In this situation, the machines will end up sharing the subnet with iSCSI data traffic (using a single network path) and application or control-plane traffic, this is sub-optimal from a performance and data availability (only one network path for iSCSI) perspective.

!!! note
    Canonical MAAS has not been formally tested at this time to provide guidance but the solution is supported by HPE.

#### Installing the HPE CSI Driver on Charmed Kubernetes

No special considerations needs to be taken when installing the HPE CSI Driver on Charmed Kubernetes. It's recommended to use the Helm chart.

- [HPE CSI Driver for Kubernetes Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver) on ArtifactHub.

When the chart is installed, [Add an HPE Storage Backend](../../csi_driver/deployment.md#add_an_hpe_storage_backend).

### MicroK8s

MicroK8s is an opinionated lightweight fully certified CNCF Kubernetes distribution. It's easy to install and manage.

#### Notes on using MicroK8s with the HPE CSI Driver

- MicroK8s is only supported by the HPE CSI Driver on Ubuntu LTS releases at this time. It will most likely work on other Linux distributions.

!!! important
    Older versions of MicroK8s did not allow the CSI driver privileged `Pods` and some tweaking may be needed in the controller-manager of MicroK8s. Please use a recent version of MicroK8s and Ubuntu LTS to avoid problems.

#### Installing the HPE CSI Driver on MicroK8s

As MicroK8s is installed with confinement using `snap`, the "kubeletRootDir" needs to be configured when installing the Helm chart or Operator. Advanced install with YAML is strongly discouraged.

Install the Helm chart:
```text
microk8s helm install --create-namespace \
  --set kubeletRootDir=/var/snap/microk8s/common/var/lib/kubelet \
  -n hpe-storage my-hpe-csi-driver hpe-storage/hpe-csi-driver
```

Go ahead and [Add an HPE Storage Backend](../../csi_driver/deployment.md#add_an_hpe_storage_backend).

!!! hint
    When installing the chart on other Linux distributions than Ubuntu LTS, the "kubeletRootDir" will most likely differ.
