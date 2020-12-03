# Overview

Kasten K10 by Veeam is a data management platform designed to run natively on Kubernetes to protect applications. K10 integrates seamlessly with the HPE CSI Driver for Kubernetes thanks to the native support for CSI `VolumeSnapshots` and `VolumeSnapshotClasses`.

HPE and Veeam have a long-standing alliance. Read about the extended partnership with Kasten in [this blog post](https://community.hpe.com/t5/around-the-storage-block/kubernetes-backup-and-recovery-with-hpe-and-kasten-by-veeam/ba-p/7109289).

!!! tip
    All the steps below are captured in a [tutorial available on YouTube](https://www.youtube.com/watch?v=bTHUlRBUcTM) and in the SCOD [Video Gallery](../../learn/video_gallery/index.md#get_started_with_kasten_k10_by_veeam_and_the_hpe_csi_driver).

[TOC]

## Prerequisites

The cluster needs to be running Kubernetes 1.17 or later and have the CSI snapshot beta `CustomResourceDefinitions` (CRDs) and the CSI snapshot-controller deployed. Follow the guides available on SCOD to:

- [Enable CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots)
- [Using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots)

!!! note
    The rest of this guide assumes a default `VolumeSnapshotClass` and `VolumeSnapshots` are functional on the cluster.

### Annotate the VolumeSnapshotClass

In order to allow K10 to perform snapshots and restores using the `VolumeSnapshotClass`, it needs an annotation.

Assuming we have a default `VolumeSnapshotClass` named "hpe-snapshot":

```markdown
kubectl annotate volumesnapshotclass hpe-snapshot k10.kasten.io/is-snapshot-class=true
```

## Installing Kasten K10

Kasten K10 installs in its own namespace using a Helm chart. It also assumes there's a performant default `StorageClass` on the cluster to serve the various `PersistentVolumeClaims` needed for the controllers.

- [Pre-flight checks and prerequisites](https://docs.kasten.io/latest/install/requirements.html#pre-flight-checks)
- [Install K10 on Kubernetes](https://docs.kasten.io/latest/install/other/other.html)

!!! note
    Above links are external to [docs.kasten.io](https://docs.kasten.io).

## Snapshots and restores

Kasten K10 provides the user with a graphical interface and dashboard to schedule and perform data management operations. There's also an API that can be manipulated with `kubectl` using CRDs.

To perform snapshot and restore operations through Kasten K10 using the HPE CSI Driver for Kubernetes, please refer to the Kasten K10 documentation.

- [Accessing K10](https://docs.kasten.io/latest/access/access.html)
- [Using K10](https://docs.kasten.io/latest/usage/usage.html)

!!! note
    Above links are external to [docs.kasten.io](https://docs.kasten.io).
