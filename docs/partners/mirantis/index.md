# Introduction

Mirantis Kubernetes Engine (MKE) is the successor of the Universal Control Plane part of Docker Enterprise Edition (Docker EE). The HPE CSI Driver for Kubernetes allows users to provision persistent storage for Kubernetes workloads running on MKE. See the note below on [Docker Swarm](#docker_swarm) for workloads deployed outside of Kubernetes.

[TOC]

## Installation

At this time neither of the HPE CSI Driver Helm chart or operator will install correctly on MKE. This ecosystem guide will be updated if future versions of the chart or operator will be supported.

### Prerequisites

The MKE managers and workers needs to run a supported host OS as outlined in the particular version of the HPE CSI Driver found in the [release tables](../../csi_driver/index.md#compatibility_and_support). Also verify that the HPE CSI Driver support the version Kubernetes used by MKE (see below).

### Steps to install

MKE admins needs to familiarize themselves with the [advanced install](../../csi_driver/deployment.md#advanced_install) method of the CSI driver. Before the installation begins, make sure an account with administrative privileges is being used to the deploy the driver. Also determine the actual Kubernetes version MKE is using. 

```markdown
kubectl version --short true
Client Version: v1.19.4
Server Version: v1.18.10-mirantis-1
```

In this particular example, Kubernetes 1.18 is being used. Follow the steps for 1.18 highlighted within the [advanced install](../../csi_driver/deployment.md#common) section of the deployment documentation.

- **Step 1** → Install the Linux node IO settings `ConfigMap`.
- **Step 2** → Determine which backend being used (Nimble or Primera/3PAR) and deploy the corresponding CSP manifest.
- **Step 3** → Deploy the HPE CSI Driver manifests for the Kubernetes version being used.

Next, add a [supported HPE backend](../../csi_driver/deployment.md#add_a_hpe_storage_backend) and [create a `StorageClass`](../../csi_driver/using.md#base_storageclass_parameters).

Learn more about using the CSI objects in [the comprehensive overview](../../csi_driver/using.md). Also make sure to familiarize yourself with the particular features and capabilities of the backend being used.

- [HPE Nimble Storage](../../container_storage_provider/hpe_nimble_storage/index.md)
- [HPE Primera and 3PAR](../../container_storage_provider/hpe_3par_primera/index.md)

## Docker Swarm

Some HPE storage backends have Docker Volume Plugins capable of supporting Docker Swarm workloads.

- [HPE Nimble Storage with Linux workers](../../docker_volume_plugins/hpe_nimble_storage/index.md)
- [HPE Nimble Storage with Windows workers](https://infosight.hpe.com/org/urn%3Ainfosight%3A605d9baf-c394-4882-9742-a44bd8678cad/resources/nimble/software/Integration%20Kits/HPE%20Nimble%20Storage%20Volume%20Plugin%20for%20Docker%20Windows%20Containers) (link require HPE InfoSight login)
- [HPE Cloud Volumes with Linux workers](../../docker_volume_plugins/hpe_cloud_volumes/index.md)

## Limitations

- Only MKE 3.3 and newer is supported.
- Helm chart or operator delivery vehicle of the HPE CSI Driver is not supported.
- HPE CSI Driver does not support Windows workers.
