# Overview

HPE and Broadcom collaborate to allow customers to run Kubernetes on VMware vSphere using HPE storage platforms. There are currently two ways of providing persistent storage to workloads running on virtualized Kubernetes clusters. Either using the vSphere CSI Driver which consumes storage services provided by vSphere or using the HPE CSI Driver for Kubernetes which consumes storage directly from the HPE storage array.

[TOC]

## Important Limitations and Comparisons

It's important to understand the inherited limitations and benefits of one CSI driver over the other before making a decision on which paradigm to choose.

| Feature | vSphere CSI Driver | HPE CSI Driver for Kubernetes | 
| ------- | ------------------ | ----------------------------- |
| Data protocols. | Dictated by the vSphere datastore. | TCP/IP based protocols only. NPIV (FC) is **not** supported. |
| Scaling and limits. | ESXi system limits with Kubernetes and vSphere CSI Driver limits layered on top. | Storage platform limits and HPE CSI Driver limits (dictated by platform CSP). |
| Security isolation. | Full isolation between the Kubernetes cluster and the storage array. | Fully exposed storage array, control plane and data paths. | 
| Data management. | Inherited by vSphere CSI Driver features and datastore capabilities. | Full suite of native data management through CSI and CSP specific CRDs. | 
| Kubernetes version and worker node OS. | Dictated by the vSphere CSI Driver. | Dictated by the Compatibility & Support table on SCOD. |

!!! danger "Important"
    The most common objection stems from running Kubernetes clusters on vSphere using an FC storage array. The HPE CSI Driver for Kubernetes will not able to attach the devices to the worker virtual machines as NPIV is not supported by the CSI node driver. The vSphere CSI Driver is the only option in these cases.

## Resources

Further reading for the vSphere CSI Driver is hosted externally. The HPE CSI Driver resources can be found on SCOD.

### vSphere CSI Driver

The vSphere CSI Driver is a replacement of the in-tree vSphere plugin previously part of the Kubernetes project. It's governed by SIG storage within the CNCF.

- vSphere CSI Driver on [GitHub](https://github.com/kubernetes-sigs/vsphere-csi-driver).
- Getting Started with VMware vSphere Container Storage Plug-in on [broadcom.com](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/container-storage-plugin/3-0/getting-started-with-vmware-vsphere-container-storage-plug-in-3-0/vsphere-container-storage-plug-in-concepts.html).

### HPE CSI Driver for Kubernetes

The installation and configuration of the HPE CSI Driver on Kubernetes clusters and vSphere is no different from any other paradigm. 

- Make sure to verify that the node OS and Kubernetes version is in the [Compatibility & Support](../../index.md#latest_release) table.
- Choose your [deployment method](../../deployment.md).
- Learn how to use [storage resources and data management](../../using.md).

Are you looking for the previous VMware page hosted on SCOD? See [legacy](legacy.md).
