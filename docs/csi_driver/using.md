# Overview
At this point the CSI driver and CSP should be installed and configured.

!!! important
    Most examples below assumes there's a `Secret` named "hpe-backend" in the "hpe-storage" `Namespace`. Learn how to add `Secrets` in the [Deployment section](deployment.md#add_an_hpe_storage_backend).

[TOC]

!!! tip
    If you're familiar with the basic concepts of persistent storage on Kubernetes and are looking for an overview of example YAML declarations for different object types supported by the HPE CSI driver, [visit the source code repo](https://github.com/hpe-storage/csi-driver/tree/master/examples/kubernetes) on GitHub.

## PVC Access Modes

The HPE CSI Driver for Kubernetes is primarily a `ReadWriteOnce` (RWO) CSI implementation for block based storage. The CSI driver also supports `ReadWriteMany` (RWX) and `ReadOnlyMany` (ROX) using a NFS Server Provisioner. It's enabled by transparently deploying a NFS server for each Persistent Volume Claim (PVC) against a `StorageClass` where it's enabled, that in turn is backed by a traditional RWO claim. Most of the examples featured on SCOD are illustrated as RWO using block based storage, but many of the examples apply in the majority of use cases.

| Access Mode   | Abbreviation | Use Case |
| ------------- | ------------ | -------- |
| ReadWriteOnce | RWO          | For high performance `Pods` where access to the PVC is exclusive to one host at a time. May use either block based storage or the NFS Server Provisioner where connectivity to the data fabric is limited to a few worker nodes in the Kubernetes cluster. |
| ReadWriteMany | RWX          | For shared filesystems where multiple `Pods` in the same `Namespace` need simultaneous access to a PVC across multiple nodes. |
| ReadOnlyMany  | ROX          | Read-only representation of RWX. |

!!! seealso "ReadWriteOnce and access by multiple Pods"
    `Pods` that require access to the same "ReadWriteOnce" (RWO) PVC needs to reside on the same node and `Namespace` by using [selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) or [affinity scheduling rules](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) applied when deployed. If not configured correctly, the `Pod` will fail to start and will throw a "Multi-Attach" error in the event log if the PVC is already attached to a `Pod` that has been scheduled on a different node within the cluster.

The NFS Server Provisioner is not enabled by the default `StorageClass` and needs a custom `StorageClass`. The following sections are tailored to help understand the NFS Server Provisioner capabilities.

* [Using the NFS Server Provisioner](#using_the_nfs_server_provisioner)
* [NFS Server Provisioner `StorageClass` parameters](#base_storageclass_parameters)
* [Diagnosing the NFS Server Provisioner issues](diagnostics.md#nfs_server_provisioner_resources)

## Enabling CSI Snapshots

Support for `VolumeSnapshotClasses` and `VolumeSnapshots` is available from Kubernetes 1.17+. The snapshot beta CRDs and the common snapshot controller needs to be installed manually. As per Kubernetes SIG Storage, these should not be installed as part of a CSI driver and should be deployed by the Kubernetes cluster vendor or user.

Ensure the snapshot CRDs and common snapshot controller hasn't been installed already.

```markdown
kubectl get crd volumesnapshots.snapshot.storage.k8s.io \
  volumesnapshotcontents.snapshot.storage.k8s.io \
  volumesnapshotclasses.snapshot.storage.k8s.io
```

Vendors may package, name and deploy the common snapshot controller using their own naming conventions. Run the command below and look for workload names that contain "snapshot".

```markdown
kubectl get sts,deploy -A
```

If no prior CRDs or controllers exist, install the snapshot CRDs and common snapshot controller (once per Kubernetes cluster, independent of any CSI drivers).

```markdown fct_label="HPE CSI Driver v2.0.0"
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter

# Kubernetes 1.20 and newer
git checkout tags/v4.0.0 -b release-4.0

# Kubernetes 1.18 and 1.19
git checkout tags/v3.0.3 -b release-3.0

# All versions
kubectl apply -f client/config/crd -f deploy/kubernetes/snapshot-controller
```

```markdown fct_label="HPE CSI Driver v1.4.0"
# Kubernetes 1.17-1.19
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v3.0.3 -b release-3.0
kubectl apply -f client/config/crd -f deploy/kubernetes/snapshot-controller
```

```markdown fct_label="HPE CSI Driver v1.3.0"
# Kubernetes 1.17-1.19
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v3.0.3 -b release-3.0
kubectl apply -f config/crd -f deploy/kubernetes/snapshot-controller
```

!!! tip
    The [provisioning](#provisioning_concepts) section contains examples on how to create `VolumeSnapshotClass` and `VolumeSnapshot` objects.

## Base StorageClass Parameters

Each CSP has its own set of unique parameters to control the provisioning behavior. These examples serve as a base `StorageClass` example for each version of Kubernetes. See the respective [CSP](../container_storage_provider/index.md) for more elaborate examples.

```markdown fct_label="K8s 1.15+"
# Renamed csi.storage.k8s.io/resizer-secret-name to
# csi.storage.k8s.io/controller-expand-secret-name
#
# Alpha features: PVC cloning and Pod inline ephemeral volumes
#
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: hpe-standard
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/controller-expand-secret-name: hpe-backend
  csi.storage.k8s.io/controller-expand-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  description: "Volume created by the HPE CSI Driver for Kubernetes"
reclaimPolicy: Delete
allowVolumeExpansion: true
```

```markdown fct_label="K8s 1.14"
# Alpha feature: volume expansion
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: hpe-standard
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/resizer-secret-name: hpe-backend
  csi.storage.k8s.io/resizer-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  description: "Volume created by the HPE CSI Driver for Kubernetes"
reclaimPolicy: Delete
# Required to allow volume expansion
allowVolumeExpansion: true
```

!!! important "Important"
    Replace "hpe-backend" with a `Secret` relevant to the backend being referenced.<br />

Common HPE CSI Driver `StorageClass` parameters across CSPs.

| Parameter                     | String         | Description |
| ----------------------------- | -------------- | ----------- |
| accessProtocol                | Text           | The access protocol to use when accessing the persistent volume ("fc" or "iscsi").  Default: "iscsi" |
| description<sup>1</sup>       | Text           | Text to be added to the volume PV metadata on the backend CSP. Default: "" |
| csi.storage.k8s.io/fstype     | Text           | Filesystem to format new volumes with. XFS is preferred, ext3, ext4 and btrfs is supported. Defaults to "ext4" if omitted. |
| fsOwner                       | userId:groupId | The user id and group id that should own the root directory of the filesystem. |
| fsMode                        | Octal digits   | 1 to 4 octal digits that represent the file mode to be applied to the root directory of the filesystem. |
| fsCreateOptions               | Text           | A string to be passed to the mkfs command.  These flags are opaque to CSI and are therefore not validated.  To protect the node, only the following characters are allowed:  ```[a-zA-Z0-9=, \-]```. |
| nfsResources                  | Boolean        | When set to "true", requests against the `StorageClass` will create resources for the NFS Server Provisioner (`Deployment`, RWO `PVC` and `Service`). Required parameter for ReadWriteMany and ReadOnlyMany accessModes. Default: "false" |
| nfsNamespace                  | Text           | Resources are by default created in the "hpe-nfs" `Namespace`. If CSI `VolumeSnapshotClass` and `dataSource` functionality is required on the requesting claim, requesting and backing PVC need to exist in the requesting `Namespace`. |
| nfsMountOptions               | Text           | Customize NFS mount options for the `Pods` to the server `Deployment`. Default: "nolock, hard,vers=4" |
| nfsProvisionerImage           | Text           | Customize provisioner image for the server `Deployment`. Default: Official build from "hpestorage/nfs-provisioner" repo |
| nfsResourceLimitsCpuM         | Text           | Specify CPU limits for the server `Deployment` in milli CPU. Default: no limits applied. Example: "500m" |
| nfsResourceLimitsMemoryMi     | Text           | Specify memory limits (in megabytes) for the server `Deployment`. Default: no limits applied. Example: "500Mi" |
| hostEncryption                | Boolean        | Direct the CSI driver to invoke Linux Unified Key Setup (LUKS) via the `dm-crypt` kernel module. Default: "false". See [Volume encryption](#using_volume_encryption) to learn more. |
| hostEncryptionSecretName      | Text           | Name of the `Secret` to use for the volume encryption. Mandatory if "hostEncryption" is enabled. Default: "" |
| hostEncryptionSecretNamespace | Text           | `Namespace` where to find "hostEncryptionSecretName". Default: "" |

<small><sup>1</sup> = Parameter is mutable using the [CSI Volume Mutator](#using_volume_mutations).</small>

!!! note
    All common HPE CSI Driver parameters are optional.

## Provisioning Concepts

These instructions are provided as an example on how to use the HPE CSI Driver with a [CSP](../container_storage_provider/index.md) supported by HPE.

- [Create a PersistentVolumeClaim from a StorageClass](#create_a_persistentvolumeclaim_from_a_storageclass)
- [Ephemeral inline volumes](#ephemeral_inline_volumes)
- [Raw Block Volumes](#raw_block_volumes)
- [Using CSI Snapshots](#using_csi_snapshots)
- [Volume Groups](#volume_groups)
- [Snapshot Groups](#snapshot_groups)
- [Expanding PVCs](#expanding_pvcs)
- [Using PVC Overrides](#using_pvc_overrides)
- [Using Volume Mutations](#using_volume_mutations)
- [Using Volume Encryption](#using_volume_encryption)
- [Using the NFS Server Provisioner](#using_the_nfs_server_provisioner)
- [Using volume encryption](#using_volume_encryption)

!!! tip "New to Kubernetes?"
    There's a basic tutorial of how dynamic provisioning of persistent storage on Kubernetes works in the [Video Gallery](../learn/video_gallery/index.md#dynamic_provisioning_of_persistent_storage_on_kubernetes).

### Create a PersistentVolumeClaim from a StorageClass

The below YAML declarations are meant to be created with `kubectl create`. Either copy the content to a file on the host where `kubectl` is being executed, or copy & paste into the terminal, like this:

```markdown
kubectl create -f-
< paste the YAML >
^D (CTRL + D)
```

To get started, create a `StorageClass` API object referencing the CSI driver `Secret` relevant to the backend.

These examples are for Kubernetes 1.15+

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-scod
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/controller-expand-secret-name: hpe-backend
  csi.storage.k8s.io/controller-expand-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  description: "Volume created by the HPE CSI Driver for Kubernetes"
  accessProtocol: iscsi
reclaimPolicy: Delete
allowVolumeExpansion: true
```

Create a `PersistentVolumeClaim`. This object declaration ensures a `PersistentVolume` is created and provisioned on your behalf, make sure to reference the correct `.spec.storageClassName`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-first-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-scod
```

!!! note
    In most environments, there is a default `StorageClass` declared on the cluster. In such a scenario, the `.spec.storageClassName` can be omitted. The default `StorageClass` is controlled by an annotation: `.metadata.annotations.storageclass.kubernetes.io/is-default-class` set to either `"true"` or `"false"`.

After the `PersistentVolumeClaim` has been declared, check that a new `PersistentVolume` is created based on your claim:

```markdown
kubectl get pv
NAME              CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM                STORAGECLASS AGE
pvc-13336da3-7... 32Gi     RWO          Delete         Bound  default/my-first-pvc hpe-scod     3s
```

The above output means that the HPE CSI Driver successfully provisioned a new volume. The volume is not attached to any node yet. It will only be attached to a node if a scheduled workload requests the `PersistentVolumeClaim`. Now, let us create a `Pod` that refers to the above volume. When the `Pod` is created, the volume will be attached, formatted and mounted according to the specification.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-pod
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
    - name: pod-datelog-2
      image: debian
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
  volumes:
    - name: export1
      persistentVolumeClaim:
        claimName: my-first-pvc
```

Check if the `Pod` is running successfully.

```markdown
kubectl get pod my-pod
NAME        READY   STATUS    RESTARTS   AGE
my-pod      2/2     Running   0          2m29s
```

!!! tip
    A simple `Pod` does not provide any automatic recovery if the node the `Pod` is scheduled on crashes or become unresponsive. Please see [the official Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/) for different workload types that provide automatic recovery. A shortlist of recommended workload types that are suitable for persistent storage is available in [this blog post](https://datamattsson.tumblr.com/post/182297931146/highly-available-stateful-workloads-on-kubernetes) and best practices are outlined in [this blog post](https://datamattsson.tumblr.com/post/185031432701/best-practices-for-stateful-workloads-on).

### Ephemeral Inline Volumes

It's possible to declare a volume "inline" a `Pod` specification. The volume is ephemeral and only persists as long as the `Pod` is running. If the `Pod` gets rescheduled, deleted or upgraded, the volume is deleted and a new volume gets provisioned if it gets restarted.

Ephemeral inline volumes are not associated with a `StorageClass`, hence a `Secret` needs to be provided inline with the volume.


!!! warning
    Allowing user `Pods` to access the CSP `Secret` gives them the same privileges on the backend system as the HPE CSI Driver.

There are two ways to declare the `Secret` with ephemeral inline volumes, either the `Secret` is in the same `Namespace` as the workload being declared or it resides in a foreign `Namespace`.

Local `Secret`:
```markdown
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-inline-mount-1
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: my-volume-1
          mountPath: /data
  volumes:
    - name: my-volume-1
      csi:
       driver: csi.hpe.com
       nodePublishSecretRef:
         name: hpe-backend
       fsType: ext3
       volumeAttributes:
         csi.storage.k8s.io/ephemeral: "true"
         accessProtocol: "iscsi"
         size: "5Gi"
```

Foreign `Secret`:

```markdown
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-inline-mount-2
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: my-volume-1
          mountPath: /data
  volumes:
    - name: my-volume-1
      csi:
       driver: csi.hpe.com
       fsType: ext3
       volumeAttributes:
         csi.storage.k8s.io/ephemeral: "true"
         inline-volume-secret-name: hpe-backend
         inline-volume-secret-namespace: hpe-storage
         accessProtocol: "iscsi"
         size: "7Gi"
```

The parameters used in the examples are the bare minimum required parameters. Any parameters supported by the HPE CSI Driver and backend CSP may be used for ephemeral inline volumes. See the [base StorageClass parameters](#base_storageclass_parameters) or the respective CSP being used.

!!! seealso
    For more elaborate use cases around ephemeral inline volumes, check out the tutorial on HPE DEV: [Using Ephemeral Inline Volumes on Kubernetes](https://developer.hpe.com/blog/EE2QnZBXXwi4o7X0E4M0/using-raw-block-and-ephemeral-inline-volumes-on-kubernetes)

### Raw Block Volumes

The default `volumeMode` for a `PersistentVolumeClaim` is `Filesystem`. If a raw block volume is desired, `volumeMode` needs to be set to `Block`. No filesystem will be created. Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-block
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-scod
  volumeMode: Block
```

Mapping the device in a `Pod` specification is slightly different than using regular filesystems as a `volumeDevices` section is added instead of a `volumeMounts` stanza:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-block
spec:
  containers:
    - name: my-null-pod
      image: fedora:31
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: my-pvc-block
```

!!! seealso
    There's an in-depth tutorial available on HPE DEV that covers raw block volumes: [Using Raw Block Volumes on Kubernetes](https://developer.hpe.com/blog/EE2QnZBXXwi4o7X0E4M0/using-raw-block-and-ephemeral-inline-volumes-on-kubernetes)

### Using CSI Snapshots

CSI introduces snapshots as native objects in Kubernetes that allows end-users to provision `VolumeSnapshot` objects from an existing `PersistentVolumeClaim`. New PVCs may then be created using the snapshot as a source.

!!! tip
    Ensure [CSI snapshots are enabled](#enabling_csi_snapshots).
    <br />There's a [tutorial in the Video Gallery](../learn/video_gallery/index.md#using_the_hpe_csi_driver_to_create_csi_snapshots_and_clones) on how to use CSI snapshots and clones.

Start by creating a `VolumeSnapshotClass` referencing the `Secret` and defining additional snapshot parameters.

Kubernetes 1.17+ (CSI snapshots in beta)

```markdown fct_label="HPE CSI Driver v1.4.0"
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotClass
metadata:
  name: hpe-snapshot
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: csi.hpe.com
deletionPolicy: Delete
parameters:
  description: "Snapshot created by the HPE CSI Driver"
  csi.storage.k8s.io/snapshotter-secret-name: hpe-backend
  csi.storage.k8s.io/snapshotter-secret-namespace: hpe-storage
  csi.storage.k8s.io/snapshotter-list-secret-name: hpe-backend
  csi.storage.k8s.io/snapshotter-list-secret-namespace: hpe-storage
```

```markdown fct_label="HPE CSI Driver v1.3.0"
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotClass
metadata:
  name: hpe-snapshot
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: csi.hpe.com
deletionPolicy: Delete
parameters:
  description: "Snapshot created by the HPE CSI Driver"
  csi.storage.k8s.io/snapshotter-secret-name: hpe-backend
  csi.storage.k8s.io/snapshotter-secret-namespace: hpe-storage
```

Create a `VolumeSnapshot`. This will create a new snapshot of the volume.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  source:
    persistentVolumeClaimName: my-pvc
```

!!! tip
    If a specific `VolumeSnapshotClass` is desired, use `.spec.snapshotClassName` to call it out.

Check that a new `VolumeSnapshot` is created based on your claim:

```markdown
kubectl describe volumesnapshot my-snapshot
Name:         my-snapshot
Namespace:    default
...
Status:
  Creation Time:  2019-05-22T15:51:28Z
  Ready:          true
  Restore Size:   32Gi
```

It's now possible to create a new `PersistentVolumeClaim` from the `VolumeSnapshot`.

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-from-snapshot
spec:
  dataSource:
    name: my-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
```

!!! caution "Important"
    The size in `.spec.resources.requests.storage` must match the `.spec.dataSource` size.

The `.data.dataSource` attribute may also clone `PersistentVolumeClaim` directly, without creating a `VolumeSnapshot`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-from-pvc
spec:
  dataSource:
    name: my-pvc
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
```

Again, the size in `.spec.resources.requests.storage` must match the source `PersistentVolumeClaim`. This can get sticky from an automation perspective as volume expansion is being used on the source volume. It's recommended to inspect the source `PersistentVolumeClaim` or `VolumeSnapshot` size prior to creating a clone.

!!! seealso "Learn more"
    For a more comprehensive tutorial on how to use snapshots and clones with CSI on Kubernetes 1.17, see [HPE CSI Driver for Kubernetes: Snapshots, Clones and Volume Expansion](https://developer.hpe.com/blog/PklOy39w8NtX6M2RvAxW/hpe-csi-driver-for-kubernetes-snapshots-clones-and-volume-expansion) on HPE DEV.

### Volume Groups

`PersistentVolumeClaims` created in a particular `Namespace` from the same storage backend may be grouped together in a `VolumeGroup`. A `VolumeGroup` is what may be known as a "consistency group" in other storage infrastructure systems. This allows certain attributes to be managed on a abstract group and attributes then applies to all member volumes in the group instead of managing each volume individually. One such aspect is creating snapshots with referential integrity between volumes or setting a performance attribute that would have accounting made on the logical group rather than the individual volume.

!!! tip
    A tutorial on how to use `VolumeGroups` and `SnapshotGroups` is available in the [Video Gallery](../learn/video_gallery/index.md#synchronize_volume_snapshots_for_distributed_workloads).

Before grouping `PeristentVolumeClaims` there needs to be a `VolumeGroupClass` created. It needs to reference a `Secret` that corresponds to the same backend the `PersistentVolumeClaims` were created on. A `VolumeGroupClass` is a cluster resource that needs administrative privileges to create.

```markdown
---
apiVersion: storage.hpe.com/v1
kind: VolumeGroupClass
metadata:
  name: my-volume-group-class
provisioner: csi.hpe.com
deletionPolicy: Delete
parameters:
  description: "HPE CSI Driver for Kubernetes Volume Group"
  csi.hpe.com/volume-group-provisioner-secret-name: hpe-backend
  csi.hpe.com/volume-group-provisioner-secret-namespace: hpe-storage
```

!!! note
    The `VolumeGroupClass` `.parameters` may contain CSP specifc parameters. Check the documentation of the [Container Storage Provider](../container_storage_provider) being used.

Once the `VolumeGroupClass` is in place, users may create `VolumeGroups`. The `VolumeGroups` are just like `PersistentVolumeClaims` part of a `Namespace` and both resources need to be in the same `Namespace` for the grouping to be successful.

```markdown
---
apiVersion: storage.hpe.com/v1
kind: VolumeGroup
metadata:
  name: my-volume-group
spec:
  volumeGroupClassName: my-volume-group-class
```

Depending on the CSP being used, the `VolumeGroup` may reference an object that corresponds to the Kubernetes API object. It's not until users annotates their `PersistentVolumeClaims` the `VolumeGroup` gets populated.

Adding a `PersistentVolumeClaim` to a `VolumeGroup`:

```markdown
kubectl annotate pvc/my-pvc csi.hpe.com/volume-group=my-volume-group
```

Removing a `PersistentVolumeClaim` from a `VolumeGroup`:

```markdown
kubectl annotate pvc/my-pvc csi.hpe.com/volume-group-
```

!!! tip
    While adding the `PersistentVolumeClaim` to the `VolumeGroup` is instant, removal require one reconciliation loop and might not immediately be reflected on the `VolumeGroup` object.

### Snapshot Groups

Being able to create snapshots of the `VolumeGroup` require the CSI external-snapshotter to be [installed](#enabling_csi_snapshots) and also require a `VolumeSnapshotClass` [configured](#using_csi_snapshots) using the same storage backend as the `VolumeGroup`. Once those pieces are in place, a `SnapshotGroupClass` needs to be created. `SnapshotGroupClasses` are cluster objects created by an administrator.

```markdown
---
apiVersion: storage.hpe.com/v1
kind: SnapshotGroupClass
metadata:
  name: my-snapshot-group-class
snapshotter: csi.hpe.com
deletionPolicy: Delete
parameters:
  csi.hpe.com/snapshot-group-snapshotter-secret-name: hpe-backend
  csi.hpe.com/snapshot-group-snapshotter-secret-namespace: hpe-storage
```

Creating a `SnapshotGroup` is later performed using the `VolumeGroup` as a source while referencing a `SnapshotGroupClass` and a `VolumeSnapshotClass`.

```markdown
---
apiVersion: storage.hpe.com/v1
kind: SnapshotGroup
metadata:
  name: my-snapshot-group-1
spec:
  source:
    kind: VolumeGroup
    apiGroup: storage.hpe.com
    name: my-volume-group
  snapshotGroupClassName: my-snapshot-group-class
  volumeSnapshotClassName: hpe-snapshot
```

Once the `SnapshotGroup` has been successfully created, the individual `VolumeSnapshots` are now available in the `Namespace`.

List `VolumeSnapshots`:

```markdown
kubectl get volumesnapshots
```

If no `VolumeSnapshots` are being enumerated, check the [diagnostics](diagnostics.md#volume_and_snapshot_groups) on how to check the component logs and such.

!!! tip "New feature!"
    Volume Groups and Snapshot Groups got introduced in HPE CSI Driver for Kubernetes 1.4.0.

### Expanding PVCs

To perform expansion operations on Kubernetes 1.14+, you must enhance your `StorageClass` with some additional attributes. Please see [base `StorageClass` parameters](#base_storageclass_parameters).

Then, a volume provisioned by a `StorageClass` with expansion attributes may have its `PersistentVolumeClaim` expanded by altering the `.spec.resources.requests.storage` key of the `PersistentVolumeClaim`.

This may be done by the `kubectl patch` command.

```markdown
kubectl patch pvc/my-pvc --patch '{"spec": {"resources": {"requests": {"storage": "64Gi"}}}}'
persistentvolumeclaim/my-pvc patched
```

The new `PersistentVolumeClaim` size may be observed with `kubectl get pvc/my-pvc` after a few moments.

### Using PVC Overrides

The HPE CSI Driver allows the `PersistentVolumeClaim` to override the `StorageClass` parameters by annotating the `PersistentVolumeClaim`. Define the parameters allowed to be overridden in the `StorageClass` by setting the `allowOverrides` parameter:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-scod-override
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  description: "Volume provisioned by the HPE CSI Driver"
  accessProtocol: iscsi
  allowOverrides: description,accessProtocol
```

The end-user may now control those parameters (the `StorageClass` provides the default values).

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-override
  annotations:
    csi.hpe.com/description: "This is my custom description"
    csi.hpe.com/accessProtocol: fc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: hpe-scod-override
```

### Using Volume Mutations

The HPE CSI Driver (version 1.3.0 and later) allows the CSP backend volume to be mutated by annotating the `PersistentVolumeClaim`. Define the parameters allowed to be mutated in the `StorageClass` by setting the `allowMutations` parameter.

!!! tip
    There's a tutorial available on YouTube accessible through the [Video Gallery](../learn/video_gallery/index.md#adapt_stateful_workloads_dynamically_with_the_hpe_csi_driver_for_kubernetes) on how to use volume mutations to adapt stateful workloads with the HPE CSI Driver.

!!! caution "Important"
    In order to mutate a `StorageClass` parameter it needs to have a default value set in the `StorageClass`. In the example below we'll allow mutatating "description". If the parameter "description" isn't set when the volume was provisioned, no subsequent mutations are allowed.

```markdown
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-scod-mutation
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  description: "Volume provisioned by the HPE CSI Driver"
  allowMutations: description
```

!!! note
    The `allowMutations` parameter is a comma separated list of values defined by each of the CSPs parameters, except the `description` parameter, which is common across all CSPs. See the documentation for each [CSP](../container_storage_provider/index.md) on what parameters are mutable.

The end-user may now control those parameters by editing or patching the `PersistentVolumeClaim`.

```markdown
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-mutation
  annotations:
    csi.hpe.com/description: "My description needs to change"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: hpe-scod-mutation
```

!!! tip "Good to know"
    As the `.spec.csi.volumeAttributes` on the `PersistentVolume` are immutable, the mutations performed on the backend volume are also annotated on the `PersistentVolume` object.

### Using the NFS Server Provisioner

Enabling the NFS Server Provisioner to allow RWX and ROX access mode for a PVC is straightforward. Create a new `StorageClass` and set `.parameters.nfsResources` to `"true"`. Any subsequent claim to the `StorageClass` will create a NFS server `Deployment` on the cluster with the associated objects running on top of a RWO PVC.

Any RWO claim made against the `StorageClass` will also create a NFS server `Deployment`. This allows diverse connectivity options among the Kubernetes worker nodes as the HPE CSI Driver will look for nodes labelled `csi.hpe.com/hpe-nfs=true` before submitting the workload for scheduling. This allows dedicated NFS worker nodes without user workloads using taints and tolerations.

By default, the NFS Server Provisioner deploy resources in the "hpe-nfs" `Namespace`. This makes it easy to manage and diagnose. However, to use CSI data management capabilities on the PVCs, the NFS resources need to be deployed in the same `Namespace` as the RWX/ROX requesting PVC. This is controlled by the `nfsNamespace` `StorageClass` parameter. See [base `StorageClass` parameters](#base_storageclass_parameters) for more information.

!!! tip
    A comprehensive [tutorial is available](https://developer.hpe.com/blog/xABwJY56qEfNGMEo1lDj/introducing-a-nfs-server-provisioner-and-pod-monitor-for-the-hpe-csi-dri) on HPE DEV on how to get started with the NFS Server Provisioner and the HPE CSI Driver for Kubernetes. There's also a brief tutorial available in the [Video Gallery](../learn/video_gallery/index.md#multi-writer_workloads_using_the_nfs_server_provisioner).

Example use of `accessModes`:

```markdown fct_label="ReadWriteOnce"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-rwo-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-nfs
```

```markdown fct_label="ReadWriteMany"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-rwx-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-nfs
```

```markdown fct_label="ReadOnlyMany"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-rox-pvc
spec:
  accessModes:
  - ReadOnlyMany
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-nfs
```

In the case of declaring a ROX PVC, the requesting `Pod` specification needs to request the PVC as read-only. Example:

```markdown
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-rox
spec:
  containers:
  - image: busybox
    name: busybox
    command:
      - "sleep"
      - "300"
    volumeMounts:
    - mountPath: /data
      name: my-vol
      readOnly: true
  volumes:
  - name: my-vol
    persistentVolumeClaim:
      claimName: my-rox-pvc
      readOnly: true
```

Requesting an empty read-only volume might not seem practical. The primary use case is to source existing datasets into immutable applications, using either a backend CSP cloning capability or CSI data management feature such as [snapshots or existing PVCs](#using_csi_snapshots).

#### Limitations and Considerations for the NFS Server Provisioner

The current tested and supported limit for the NFS Server Provisioner is 20 NFS servers per Kubernetes worker node. The NFS server `Deployment` is currently setup in a completely unfettered resource mode where it will consume as much memory and CPU as it requests.

The two `StorageClass` parameters `nfsResourceLimitsCpuM` and `nfsResourceLimitsMemoryMi` control how much CPU and memory it may consume. Tests show that the NFS server consumes about 150MiB at instantiation.

The `PVC` with "ReadWriteMany" or "ReadOnlyMany" access mode can NOT be expanded. If more capacity is needed, expand the "ReadWriteOnce" `PVC` backing the NFS Server Provisioner. This will result in inaccurate space reporting.

The HPE CSI Driver includes a Pod Monitor to delete `Pods` that have become unavailable due to the Pod status changing to `NodeLost` or a node becoming unreachable that the `Pod` runs on. By default the Pod Monitor only watches the NFS Server Provisioner `Deployments`. It may be used for any `Deployment`. See [Pod Monitor](monitor.md) on how to use it, especially the [limitations](monitor.md#limitations).

See [diagnosing NFS Server Provisioner issues](diagnostics.md#nfs_server_provisioner_resources) for further details.

### Using Volume Encryption

From version 2.0.0 and onwards of the CSI driver supports host-based volume encryption for any of the CSPs supported by the CSI driver.

Host-based volume encryption is controlled by `StorageClass` parameters configured by the Kubernetes administrator and may be configured to be overridden by Kubernetes users. In the below example, a single `Secret` is used to encrypt and decrypt all volumes provisioned by the `StorageClass`.

First, create a `Secret`, in this example we'll use the "hpe-storage" `Namespace`.

```markdown
apiVersion: v1
kind: Secret
metadata:
  name: my-passphrase
  namespace: hpe-storage
stringData:
  hostEncryptionPassphrase: "HPE CSI Driver for Kubernetes 2.0.0 Rocks!"
```

!!! tip
    The "hostEncryptionPassphrase" can be up to 512 characters.

Next, incorporate the `Secret` into a `StorageClass`.

```markdown
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-encrypted
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  description: "Volume provisioned by the HPE CSI Driver"
  hostEncryption: "true"
  hostEncryptionSecretName: my-passphrase
  hostEncryptionSecretNamespace: hpe-storage
reclaimPolicy: Delete
allowVolumeExpansion: true
```

Next, create a `PersistentVolumeClaim` that uses the "hpe-encrypted" `StorageClass`:

```markdown
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-encrypted-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: hpe-encrypted
```

Attach a basic `Pod` to verify functionality.

```markdown
kind: Pod
apiVersion: v1
metadata:
  name: my-pod
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
    - name: pod-datelog-2
      image: debian
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
  volumes:
    - name: export1
      persistentVolumeClaim:
        claimName: my-encrypted-pvc
```

Once the `Pod` comes up, verify that the volume is encrypted.

```markdown
$ kubectl exec -it my-pod -c pod-datelog-1 -- df -h /data
Filesystem              Size  Used Avail Use% Mounted on
/dev/mapper/enc-mpatha  100G   33M  100G   1% /data
```

Host-based volume encryption is in effect if the "enc" prefix is seen on the multipath device name.

!!! seealso 
    For an in-depth tutorial and more advanced use cases for host-based volume encryption, check out this blog post on HPE DEV: [Host-based Volume Encryption with HPE CSI Driver for Kubernetes](https://developer.hpe.com/blog/host-based-volume-encryption-with-hpe-csi-driver-for-kubernetes/)

## Further Reading

The [official Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/) contains comprehensive documentation on how to markup `PersistentVolumeClaim` and `StorageClass` API objects to tweak certain behaviors.

Each CSP has a set of unique `StorageClass` parameters that may be tweaked to accommodate a wide variety of use cases. Please see the documentation of [the respective CSP for more details](../container_storage_provider/index.md).
