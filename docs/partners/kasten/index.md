# Overview
<img src="img/veeam-ready.png" align="right" width="192" hspace="12" vspace="2" />
Veeam Kasten is a data management platform designed to run natively on Kubernetes to protect applications. Kasten integrates seamlessly with the HPE CSI Driver for Kubernetes thanks to the native support for CSI `VolumeSnapshots` and `VolumeSnapshotClasses`.

HPE CSI Driver for Kubernetes is on the Veeam Alliance Partner Technical Programs designated as Veeam Ready.

- View HPE CSI Driver for Kubernetes in the [Veeam Ready database](https://www.veeam.com/sys1041).

!!! tip
    All the steps below are captured in a [tutorial available on YouTube](https://www.youtube.com/watch?v=bTHUlRBUcTM) and in the SCOD [Video Gallery](../../learn/video_gallery/index.md#get_started_with_kasten_k10_by_veeam_and_the_hpe_csi_driver).

[TOC]

## Prerequisites

The cluster needs to be running Kubernetes 1.25 or later and have the CSI snapshot `CustomResourceDefinitions` (CRDs) and the CSI snapshot-controller deployed. Follow the guides available on SCOD to:

- [Enable CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots)
- [Using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots)

!!! note
    The rest of this guide assumes a default `VolumeSnapshotClass` and `VolumeSnapshots` are functional on the cluster.

### Annotate the VolumeSnapshotClass

In order to allow Kasten to perform snapshots and restores using the `VolumeSnapshotClass`, it needs an annotation.

Assuming we have a default `VolumeSnapshotClass` named "hpe-snapshot":

```text
kubectl annotate volumesnapshotclass hpe-snapshot k10.kasten.io/is-snapshot-class=true
```

## Installing Kasten

Kasten installs in its own namespace using a Helm chart. It also assumes there's a performant default `StorageClass` on the cluster to serve the various `PersistentVolumeClaims` needed for the controllers.

- [Pre-flight checks and prerequisites](https://docs.kasten.io/latest/install/requirements.html#pre-flight-checks)
- [Install Kasten on Kubernetes](https://docs.kasten.io/latest/install/other/other.html)

!!! note
    Above links are external to [docs.kasten.io](https://docs.kasten.io).

## Snapshots and restores

Kasten provides the user with a graphical interface and dashboard to schedule and perform data management operations. There's also an API that can be manipulated with `kubectl` using CRDs.

To perform snapshot and restore operations through Kasten using the HPE CSI Driver for Kubernetes, please refer to the Kasten documentation.

- [Accessing Kasten](https://docs.kasten.io/latest/access/access.html)
- [Using Kasten](https://docs.kasten.io/latest/usage/usage.html)

!!! note
    Above links are external to [docs.kasten.io](https://docs.kasten.io).
