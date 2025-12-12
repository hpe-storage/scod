# Overview

At this point the CSI driver should be installed and configured.

!!! important
    Most examples below assumes there's a `Secret` named "hpe-file-backend" in the "hpe-storage" `Namespace`. Learn how to add `Secrets` in the [Deployment section](deployment.md#add_a_storage_backend).

[TOC]

## PVC Access Modes

The HPE GreenLake File Storage CSI Driver is primarily a `ReadWriteMany` (RWX) CSI implementation for file based storage. The CSI driver also supports `ReadWriteOnce` (RWO) and `ReadOnlyMany` (ROX).

| Access Mode      | Abbreviation | Use Case |
| ---------------- | ------------ | -------- |
| ReadWriteOnce    | RWO          | For high performance `Pods` where access to the PVC is exclusive to one host at a time. |
| ReadWriteOncePod | RWOP         | Exclusive access by a single `Pod`. Not currently supported by the CSI driver. |
| ReadWriteMany    | RWX          | For shared filesystems where multiple `Pods` in the same `Namespace` need simultaneous access to a PVC across multiple nodes. |
| ReadOnlyMany     | ROX          | Read-only representation of RWX. |

!!! seealso "ReadWriteOnce and access by multiple Pods"
    `Pods` that require access to the same "ReadWriteOnce" (RWO) PVC needs to reside on the same node and `Namespace` by using [selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) or [affinity scheduling rules](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) applied when deployed. If not configured correctly, the `Pod` will fail to start and will throw a "Multi-Attach" error in the event log if the PVC is already attached to a `Pod` that has been scheduled on a different node within the cluster.

## Enabling CSI Snapshots

Support for `VolumeSnapshotClasses` and `VolumeSnapshots` is available from Kubernetes 1.17+. The snapshot CRDs and the common snapshot controller needs to be installed manually. As per Kubernetes TAG Storage, these should not be installed as part of a CSI driver and should be deployed by the Kubernetes cluster vendor or user.

Ensure the snapshot CRDs and common snapshot controller hasn't been installed already.

```text
kubectl get crd volumesnapshots.snapshot.storage.k8s.io \
  volumesnapshotcontents.snapshot.storage.k8s.io \
  volumesnapshotclasses.snapshot.storage.k8s.io
```

Vendors may package, name and deploy the common snapshot controller using their own naming conventions. Run the command below and look for workload names that contain "snapshot".

```text
kubectl get sts,deploy -A
```

If no prior CRDs or controllers exist, install the snapshot CRDs and common snapshot controller (once per Kubernetes cluster, independent of any CSI drivers).

```text fct_label="HPE GreenLake for File Storage CSI Driver v2.6.4"
# Kubernetes 1.30-1.34
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v8.4 -b hpe-greenlake-for-file-csi-driver-v2.6.4
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

```text fct_label="v1.0.0-beta"
# Kubernetes 1.28-1.31
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v8.0.1 -b hpe-greenlake-for-file-csi-driver-v1.0.0-beta
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

!!! tip
    The [provisioning](#provisioning_concepts) section contains examples on how to create a `VolumeSnapshotClass` and `VolumeSnapshot` objects.

## Base StorageClass Parameters

This serve as a base `StorageClass` using the most common scenario.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-file-standard
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: filex.csi.hpe.com
parameters:
  csi.storage.k8s.io/provisioner-secret-name: hpe-file-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-file-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-file-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-expand-secret-name: hpe-file-backend
  csi.storage.k8s.io/controller-expand-secret-namespace: hpe-storage
  root_export: /my-volumes
  view_policy: my-view-policy-1
  vip_pool_name: my-pool-1
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```

!!! important "Important"
    Replace "hpe-file-backend" with a `Secret` relevant to the backend being referenced. See [Deployment](deployment.md#secret_parameters) on how to create a `Secret`.<br />

HPE GreenLake for File Storage CSI Driver `StorageClass` parameters.

| Parameter              | Required | String   | Description |
| ---------------------- | -------- | -------- | ----------- |
| root_export            | Yes      | Text     | Folder on the appliance to create new `PersistentVolumes` in. |
| view_policy            | Yes      | Text     | Existing View Policy on the appliance. |
| vip_pool_name          | No<sup>&ast;</sup>  | Text     | Existing VIP Pool on the appliance. |
| vip_pool_fqdn          | No<sup>&ast;</sup>  | Text     | Existing DNS name with multiple `A` records that maps to IP addresses in a VIP Pool. |
| use_local_ip_for_mount | No       | Text     | Hard coded IP address for the NFS server. Has no effect if vip_pool_name or vip_pool_fqdn is set. |
| qos_policy             | No       | Text     | Existing QoS Policy on the appliance. |

<small><sup>&ast;</sup> = Mutually exclusive.</small>

!!! tip
    Guidance on how to create the necessary resources such as a VIP Pool, View Policy, and QoS Policy on the appliance, check out the official [HPE GreenLake for File Storage: Cluster Administrator Guide](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00002658en_us)

### NFS Protocol Version

The CSI driver will by default mount NFS exports with version 3. To use version 4.1, add `nfsvers=4` to "mountOptions" in the `StorageClass`.

```yaml
mountOptions:
  - nfsvers=4
```

## Provisioning Concepts

These instructions are provided as an example on how to use common Kubernetes resources with the CSI driver.

- [Create a PersistentVolumeClaim from a StorageClass](#create_a_persistentvolumeclaim_from_a_storageclass)
- [Using CSI Snapshots](#using_csi_snapshots)
- [Expanding PVCs](#expanding_pvcs)

!!! tip "New to Kubernetes?"
    There's a basic tutorial of how dynamic provisioning of persistent storage on Kubernetes works in the [Video Gallery](../learn/video_gallery/index.md#dynamic_provisioning_of_persistent_storage_on_kubernetes).

### Create a PersistentVolumeClaim from a StorageClass

The steps in the HPE CSI Driver for Kubernetes section of SCOD outlines the basic concepts of [creating a PVC from a `StorageClass`](../csi_driver/using.md#create_a_persistentvolumeclaim_from_a_storageclass). Skip the steps creating a HPE CSI Driver for Kubernetes `StorageClass`.

### Using CSI Snapshots

CSI introduces snapshots as native objects in Kubernetes that allows end-users to provision `VolumeSnapshot` objects from an existing `PersistentVolumeClaim`. New PVCs may then be created using the snapshot as a source.

!!! tip
    Ensure [CSI snapshots are enabled](#enabling_csi_snapshots).
    <br />There's a [tutorial in the Video Gallery](../learn/video_gallery/index.md#using_the_hpe_csi_driver_to_create_csi_snapshots_and_clones) on how to use CSI snapshots and clones.

Start by creating a `VolumeSnapshotClass` referencing the `Secret` and defining additional snapshot parameters.

```yaml
apiVersion: snapshot.storage.k8s.io/v1
driver: filex.csi.hpe.com
deletionPolicy: Delete
kind: VolumeSnapshotClass
metadata:
  name: hpe-file-snapshot
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
parameters:
  csi.storage.k8s.io/snapshotter-secret-name: hpe-file-backend
  csi.storage.k8s.io/snapshotter-secret-namespace: hpe-storage
  csi.storage.k8s.io/snapshotter-list-secret-name: hpe-file-backend
  csi.storage.k8s.io/snapshotter-list-secret-namespace: hpe-storage
```

Once a `VolumeSnapshotClass` has been created, follow the steps outlined in the HPE CSI Driver section for [using CSI snapshots](../csi_driver/using.md#using_csi_snapshots).

### Expanding PVCs

Instructions on how to expand an existing PVC is available in the HPE CSI Driver section for [expanding PVCs](../csi_driver/using.md#expanding_pvcs).

## Further Reading

The [official Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/) contains comprehensive documentation on how to markup `PersistentVolumeClaim` and `StorageClass` resources to tweak certain behaviors. Including `volumeBindingMode` and `mountOptions`.
