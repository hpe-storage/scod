# Overview

The Commvault intelligent data management platform provides Kubernetes-native protection, application mobility, and disaster recovery for containerized applications. Combined with Commvault Command Center™, Commvault provides enterprise IT operations and DevOps teams an easy-to-use, self-service dashboard for managing the protection of Kubernetes.

HPE and Commvault have been delivering end-to-end solutions for many years. Learn more about HPE and Commvault's partnership here: [https://www.commvault.com/supported-technologies/hpe](https://www.commvault.com/supported-technologies/hpe).

!!! note
    HPE and Commvault collaborate continuously to deliver assets relevant to our joint customers.

    - Data protection for Kubernetes using Commvault Backup & Recovery, HPE Apollo Servers, and HPE CSI Driver for Kubernetes ([PDF](https://www.hpe.com/psnow/doc/a00105578enw))
    - Data Protection for Kubernetes using Commvault Backup & Recovery with HPE Alletra ([YouTube](https://www.youtube.com/HPMobileEnterprise))

[TOC]

## Pre-requisites

The HPE CSI Driver has been validated on Commvault Complete Backup and Recovery 2022E. 
Check that the [HPE CSI Driver](https://scod.hpedev.io/csi_driver/index.html#compatibility_and_support) and [Commvault software](https://documentation.commvault.com/2022e/essential/124720_system_requirements_for_kubernetes.html#kubernetes-release-supportability) versions are compatible with the Kubernetes version being used.

##### Permissions

This guide assumes you have administrative access to Commvault Command Center and administrator access to a Kubernetes cluster with `kubectl`. Refer to the [Creating a Service Account for Kubernetes Authentication](https://documentation.commvault.com/2022e/essential/129223_creating_kubernetes_cluster_admin_service_account_for_commvault.html) documentation to define a `serviceaccount` and `clusterrolebinding` with `cluster-admin` permissions.

##### Cluster requirements

The cluster needs to be running Kubernetes 1.22 or later and have the CSI snapshot `CustomResourceDefinitions` (CRDs) and the CSI external snapshotter deployed. Follow the guides available on SCOD to:

- [Enable CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots)
- [Using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots)

!!! note
    The rest of this guide assumes the default `VolumeSnapshotClass` and `VolumeSnapshots` are functional within the cluster with a compatible Kubernetes snapshot API level between the CSI driver and Commvault.

## Configure Kubernetes protection

To configure data protection for Kubernetes, follow the [official Commvault documentation](https://documentation.commvault.com/2022e/essential/123634_protecting_kubernetes_with_commvault.html) and ensure the version matches the software version in your environment.
As a summary, complete the following:

- [Core Setup Wizard](https://documentation.commvault.com/2022e/essential/86638_step_3_complete_core_setup_wizard.html) to complete Commvault deployment
- Review [System Requirements for Kubernetes](https://documentation.commvault.com/2022e/essential/124720_system_requirements_for_kubernetes.html)
- Complete the [Kubernetes Guided Setup](https://documentation.commvault.com/2022e/essential/131390_completing_guided_setup_for_kubernetes.html)

#### Backup and Restores

To perform snapshot and restore operations through Commvault using the HPE CSI Driver for Kubernetes, please refer to the Commvault documentation.

- [Backup](https://documentation.commvault.com/2022e/essential/123639_backups_for_kubernetes.html)
- [Restores](https://documentation.commvault.com/2022e/essential/123640_restores_for_kubernetes.html)

!!! note
    Above links are external to [documentation.commvault.com](https://documentation.commvault.com/).
