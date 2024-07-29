# Overview
At this point the CSI driver and CSP should be installed and configured.

!!! important
    Most examples below assumes there's a `Secret` named "hpe-backend" in the "hpe-storage" `Namespace`. Learn how to add `Secrets` in the [Deployment section](deployment.md#add_an_hpe_storage_backend).

[TOC]

!!! tip
    If you're familiar with the basic concepts of persistent storage on Kubernetes and are looking for an overview of example YAML declarations for different object types supported by the HPE CSI driver, [visit the source code repo](https://github.com/hpe-storage/csi-driver/tree/master/examples/kubernetes) on GitHub.

## PVC Access Modes

The HPE CSI Driver for Kubernetes is primarily a `ReadWriteOnce` (RWO) CSI implementation for block based storage. The CSI driver also supports `ReadWriteMany` (RWX) and `ReadOnlyMany` (ROX) using a NFS Server Provisioner. It's enabled by [transparently deploying a NFS server](#using_the_nfs_server_provisioner) for each Persistent Volume Claim (PVC) against a `StorageClass` where it's enabled, that in turn is backed by a traditional RWO claim. Most of the examples featured on SCOD are illustrated as RWO using block based storage, but many of the examples apply in the majority of use cases.

| Access Mode      | Abbreviation | Use Case |
| ---------------- | ------------ | -------- |
| ReadWriteOnce    | RWO          | For high performance `Pods` where access to the PVC is exclusive to one host at a time. May use either block based storage or the NFS Server Provisioner where connectivity to the data fabric is limited to a few worker nodes in the Kubernetes cluster. |
| ReadWriteOncePod | RWOP         | Exclusive access by a single `Pod`. Not currently supported by the HPE CSI Driver. |
| ReadWriteMany    | RWX          | For shared filesystems where multiple `Pods` in the same `Namespace` need simultaneous access to a PVC across multiple nodes. |
| ReadOnlyMany     | ROX          | Read-only representation of RWX. |

!!! seealso "ReadWriteOnce and access by multiple Pods"
    `Pods` that require access to the same "ReadWriteOnce" (RWO) PVC needs to reside on the same node and `Namespace` by using [selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) or [affinity scheduling rules](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) applied when deployed. If not configured correctly, the `Pod` will fail to start and will throw a "Multi-Attach" error in the event log if the PVC is already attached to a `Pod` that has been scheduled on a different node within the cluster.

The NFS Server Provisioner is not enabled by the default `StorageClass` and needs a custom `StorageClass`. The following sections are tailored to help understand the NFS Server Provisioner capabilities.

* [Using the NFS Server Provisioner](#using_the_nfs_server_provisioner)
* [NFS Server Provisioner `StorageClass` parameters](#base_storageclass_parameters)
* [Diagnosing the NFS Server Provisioner issues](diagnostics.md#nfs_server_provisioner_resources)
* [Limitations and Considerations for the NFS Server Provisioner](using.md#limitations_and_considerations_for_the_nfs_server_provisioner)

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

```text fct_label="HPE CSI Driver v2.5.0"
# Kubernetes 1.27-1.30
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v8.0.1 -b hpe-csi-driver-v2.5.0
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

```text fct_label="v2.4.2"
# Kubernetes 1.26-1.29
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v6.3.3 -b hpe-csi-driver-v2.4.2
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

```text fct_label="v2.4.1"
# Kubernetes 1.26-1.29
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v6.3.3 -b hpe-csi-driver-v2.4.1
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

```text fct_label="v2.4.0"
# Kubernetes 1.25-1.28
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v6.2.2 -b hpe-csi-driver-v2.4.0
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

```text fct_label="v2.3.0"
# Kubernetes 1.23-1.26
git clone https://github.com/kubernetes-csi/external-snapshotter
cd external-snapshotter
git checkout tags/v5.0.1 -b hpe-csi-driver-v2.3.0
kubectl kustomize client/config/crd | kubectl create -f-
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f-
```

!!! tip
    The [provisioning](#provisioning_concepts) section contains examples on how to create `VolumeSnapshotClass` and `VolumeSnapshot` objects.

## Base StorageClass Parameters

Each CSP has its own set of unique parameters to control the provisioning behavior. These examples serve as a base `StorageClass` example for each version of Kubernetes. See the respective [CSP](../container_storage_provider/index.md) for more elaborate examples.

```yaml
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

!!! important "Important"
    Replace "hpe-backend" with a `Secret` relevant to the backend being referenced.<br />

Common HPE CSI Driver `StorageClass` parameters across CSPs.

| Parameter                     | String         | Description |
| ----------------------------- | -------------- | ----------- |
| accessProtocol                | Text           | The access protocol to use when accessing the persistent volume ("fc" or "iscsi").  Default: "iscsi" |
| chapSecretName                | Text           | Name of `Secret` to use for iSCSI CHAP. |
| chapSecretNamespace           | Text           | Namespace of `Secret` to use for iSCSI CHAP. |
| description<sup>1</sup>       | Text           | Text to be added to the volume PV metadata on the backend CSP. Default: "" |
| csi.storage.k8s.io/fstype     | Text           | Filesystem to format new volumes with. XFS is preferred, ext3, ext4 and btrfs is supported. Defaults to "ext4" if omitted. |
| fsOwner                       | userId:groupId | The user id and group id that should own the root directory of the filesystem. |
| fsMode                        | Octal digits   | 1 to 4 octal digits that represent the file mode to be applied to the root directory of the filesystem. |
| fsCreateOptions               | Text           | A string to be passed to the mkfs command.  These flags are opaque to CSI and are therefore not validated.  To protect the node, only the following characters are allowed:  ```[a-zA-Z0-9=, \-]```. |
| fsRepair                      | Boolean        | When set to "true", if a mount fails and filesystem corruption is detected, this parameter will control if an actual repair will be attempted. Default: "false". <br/><br />**Note:** `fsRepair` is unable to detect or remedy corrupted filesystems that are already mounted. **Data loss may occur during the attempt to repair the filesystem.** |
| nfsResources                  | Boolean        | When set to "true", requests against the `StorageClass` will create resources for the NFS Server Provisioner (`Deployment`, RWO `PVC` and `Service`). Required parameter for ReadWriteMany and ReadOnlyMany accessModes. Default: "false" |
| nfsForeignStorageClass        | Text           | Provision NFS servers on `PVCs` from a different `StorageClass`. See [Using a Foreign StorageClass](#using_a_foreign_storageclass) |
| nfsNamespace                  | Text           | Resources are by default created in the "hpe-nfs" `Namespace`. If CSI `VolumeSnapshotClass` and `dataSource` functionality is required on the requesting claim, requesting and backing PVC need to exist in the requesting `Namespace`. A value of "csi.storage.k8s.io/pvc/namespace" will provision resources in the requesting `PVC` `Namespace`. |
| nfsNodeSelector               | Text           | Customize the `nodeSelector` label value for the NFS `Pod`. The default behavior is to omit the `nodeSelector`. |
| nfsMountOptions               | Text           | Customize NFS mount options for the `Pods` to the server `Deployment`. Uses `mount` command defaults from the node. |
| nfsProvisionerImage           | Text           | Customize provisioner image for the server `Deployment`. Default: Official build from "hpestorage/nfs-provisioner" repo |
| nfsResourceRequestsCpuM         | Text           | Specify CPU requests for the server `Deployment` in milli CPU. Default: "500m". Example: "4000m" |
| nfsResourceRequestsMemoryMi     | Text           | Specify memory requests (in megabytes) for the server `Deployment`. Default: "512Mi". Example: "4096Mi". |
| nfsResourceLimitsCpuM         | Text           | Specify CPU limits for the server `Deployment` in milli CPU. Default: "1000m". Example: "4000m" |
| nfsResourceLimitsMemoryMi     | Text           | Specify memory limits (in megabytes) for the server `Deployment`. Default: "2048Mi". Example: "500Mi". Recommended minimum: "2048Mi". |
| hostEncryption                | Boolean        | Direct the CSI driver to invoke Linux Unified Key Setup (LUKS) via the `dm-crypt` kernel module. Default: "false". See [Volume encryption](#using_volume_encryption) to learn more. |
| hostEncryptionSecretName      | Text           | Name of the `Secret` to use for the volume encryption. Mandatory if "hostEncryption" is enabled. Default: "" |
| hostEncryptionSecretNamespace | Text           | `Namespace` where to find "hostEncryptionSecretName". Default: "" |

<small><sup>1</sup> = Parameter is mutable using the [CSI Volume Mutator](#using_volume_mutations).</small>

!!! note
    All common HPE CSI Driver parameters are optional.

## Enabling iSCSI CHAP

Familiarize yourself with the [iSCSI CHAP Considerations](index.md#iscsi_chap_considerations) before proceeding. This section describes how to enable iSCSI CHAP with HPE CSI Driver 2.5.0 and later.

Create an iSCSI CHAP `Secret`. The referenced CHAP account does not need to exist on the storage backend, it will be created by the CSP if it doesn't exist.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-chap-secret
  namespace: hpe-storage
stringData:
  # Up to 64 characters including \-:., must start with an alpha-numeric character.
  chapUser: "my-chap-user"
  # Between 12 to 16 alpha-numeric characters.
  chapPassword: "my-chap-password"
```

Once the `Secret` has been created, there are two methods available to use it depending on the situation, cluster-wide or per `StorageClass`.

### Cluster-wide iSCSI CHAP Credentials

The cluster-wide iSCSI CHAP credentials will be used by all iSCSI-based `PersistentVolumes` regardless of backend and `StorageClass`. The CHAP `Secret` is simply referenced during install of the HPE CSI Driver for Kubernetes Helm Chart. The `Secret` and `Namespace` needs to exist prior to install.

Example:

```text
helm install my-hpe-csi-driver -n hpe-storage \
  hpe-storage/hpe-csi-driver \
  --set iscsi.chapSecretName=my-chap-secret
```

!!! important
    Once a `PersistentVolume` has been provisioned with cluster-wide iSCSI CHAP credentials it's not possible to switch over to per `StorageClass` iSCSI CHAP credentials.<br /><br />If CSI driver 2.4.2 or earlier has been used, cluster-wide iSCSI CHAP credentials is the only way to provide the credentials for volumes provisioned with 2.4.2 or earlier.

### Per StorageClass iSCSI CHAP Credentials

The CHAP `Secret` may be referenced in a `StorageClass`.

```yaml
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
  chapSecretNamespace: hpe-storage
  chapSecretName: my-chap-secret
reclaimPolicy: Delete
allowVolumeExpansion: true
```

!!! warning
    The iSCSI CHAP credentials are in reality per iSCSI Target. Do **NOT** create multiple `StorageClasses` referencing different CHAP `Secrets` with different credentials for the same backend. It will result in a data outage with conflicting sessions.<br /><br />Ensure the same `Secret` is referenced in all `StorageClasses` using a particular backend.

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

```text
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

```text
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

```text
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
```yaml
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

```yaml
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
    For more elaborate use cases around ephemeral inline volumes, check out the tutorial on HPE Developer: [Using Ephemeral Inline Volumes on Kubernetes](https://developer.hpe.com/blog/EE2QnZBXXwi4o7X0E4M0/using-raw-block-and-ephemeral-inline-volumes-on-kubernetes)

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

!!! note
    The `accessModes` may be set to `ReadWriteOnce`, `ReadWriteMany` or `ReadOnlyMany`. It's expected that the application handles read/write IO, volume locking and access in the event of concurrent block access from multiple nodes.

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
    There's an in-depth tutorial available on HPE Developer that covers raw block volumes: [Using Raw Block Volumes on Kubernetes](https://developer.hpe.com/blog/EE2QnZBXXwi4o7X0E4M0/using-raw-block-and-ephemeral-inline-volumes-on-kubernetes)

### Using CSI Snapshots

CSI introduces snapshots as native objects in Kubernetes that allows end-users to provision `VolumeSnapshot` objects from an existing `PersistentVolumeClaim`. New PVCs may then be created using the snapshot as a source.

!!! tip
    Ensure [CSI snapshots are enabled](#enabling_csi_snapshots).
    <br />There's a [tutorial in the Video Gallery](../learn/video_gallery/index.md#using_the_hpe_csi_driver_to_create_csi_snapshots_and_clones) on how to use CSI snapshots and clones.

Start by creating a `VolumeSnapshotClass` referencing the `Secret` and defining additional snapshot parameters.

```yaml
apiVersion: snapshot.storage.k8s.io/v1
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

!!! note
    [Container Storage Providers](../container_storage_provider) may have optional parameters to the `VolumeSnapshotClass`.

Create a `VolumeSnapshot`. This will create a new snapshot of the volume.

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  source:
    persistentVolumeClaimName: my-pvc
```

!!! tip
    If a specific `VolumeSnapshotClass` is desired, use `.spec.volumeSnapshotClassName` to call it out.

Check that a new `VolumeSnapshot` is created based on your claim:

```text
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
    For a more comprehensive tutorial on how to use snapshots and clones with CSI on Kubernetes 1.17, see [HPE CSI Driver for Kubernetes: Snapshots, Clones and Volume Expansion](https://developer.hpe.com/blog/PklOy39w8NtX6M2RvAxW/hpe-csi-driver-for-kubernetes-snapshots-clones-and-volume-expansion) on HPE Developer.

### Volume Groups

`PersistentVolumeClaims` created in a particular `Namespace` from the same storage backend may be grouped together in a `VolumeGroup`. A `VolumeGroup` is what may be known as a "consistency group" in other storage infrastructure systems. This allows certain attributes to be managed on a abstract group and attributes then applies to all member volumes in the group instead of managing each volume individually. One such aspect is creating snapshots with referential integrity between volumes or setting a performance attribute that would have accounting made on the logical group rather than the individual volume.

!!! tip
    A tutorial on how to use `VolumeGroups` and `SnapshotGroups` is available in the [Video Gallery](../learn/video_gallery/index.md#synchronize_volume_snapshots_for_distributed_workloads).

Before grouping `PeristentVolumeClaims` there needs to be a `VolumeGroupClass` created. It needs to reference a `Secret` that corresponds to the same backend the `PersistentVolumeClaims` were created on. A `VolumeGroupClass` is a cluster resource that needs administrative privileges to create.

```yaml
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

```yaml
apiVersion: storage.hpe.com/v1
kind: VolumeGroup
metadata:
  name: my-volume-group
spec:
  volumeGroupClassName: my-volume-group-class
```

Depending on the CSP being used, the `VolumeGroup` may reference an object that corresponds to the Kubernetes API object. It's not until users annotates their `PersistentVolumeClaims` the `VolumeGroup` gets populated.

Adding a `PersistentVolumeClaim` to a `VolumeGroup`:

```text
kubectl annotate pvc/my-pvc csi.hpe.com/volume-group=my-volume-group
```

Removing a `PersistentVolumeClaim` from a `VolumeGroup`:

```text
kubectl annotate pvc/my-pvc csi.hpe.com/volume-group-
```

!!! tip
    While adding the `PersistentVolumeClaim` to the `VolumeGroup` is instant, removal require one reconciliation loop and might not immediately be reflected on the `VolumeGroup` object.

### Snapshot Groups

Being able to create snapshots of the `VolumeGroup` require the CSI external-snapshotter to be [installed](#enabling_csi_snapshots) and also require a `VolumeSnapshotClass` [configured](#using_csi_snapshots) using the same storage backend as the `VolumeGroup`. Once those pieces are in place, a `SnapshotGroupClass` needs to be created. `SnapshotGroupClasses` are cluster objects created by an administrator.

```yaml
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

```yaml
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

```text
kubectl get volumesnapshots
```

If no `VolumeSnapshots` are being enumerated, check the [diagnostics](diagnostics.md#volume_and_snapshot_groups) on how to check the component logs and such.

!!! tip "New feature!"
    Volume Groups and Snapshot Groups got introduced in HPE CSI Driver for Kubernetes 1.4.0.

### Expanding PVCs

To perform expansion operations on Kubernetes 1.14+, you must enhance your `StorageClass` with the `.allowVolumeExpansion: true` key. Please see [base `StorageClass` parameters](#base_storageclass_parameters) for additional information.

Then, a volume provisioned by a `StorageClass` with expansion attributes may have its `PersistentVolumeClaim` expanded by altering the `.spec.resources.requests.storage` key of the `PersistentVolumeClaim`.

This may be done by the `kubectl patch` command.

```text
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
    In order to mutate a `StorageClass` parameter it needs to have a default value set in the `StorageClass`. In the example below we'll allow mutatating "description". If the parameter "description" wasn't set when the `PersistentVolume` was provisioned, no subsequent mutations are allowed. The CSP may set defaults for certain parameters during provisioning, if those are mutable, the mutation will be performed.

```yaml
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

```yaml
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

Enabling the NFS Server Provisioner to allow "ReadWriteMany" and "ReadOnlyMany" access mode for a `PVC` is straightforward. Create a new `StorageClass` and set `.parameters.nfsResources` to `"true"`. Any subsequent claim to the `StorageClass` will create a NFS server `Deployment` on the cluster with the associated objects running on top of a "ReadWriteOnce" `PVC`.

Any "RWO" claim made against the `StorageClass` will also create a NFS server `Deployment`. This allows diverse connectivity options among the Kubernetes worker nodes as the HPE CSI Driver will look for nodes labelled `csi.hpe.com/hpe-nfs=true` (or using a custom value specified in `.parameters.nfsNodeSelector`) before submitting the workload for scheduling. This allows dedicated NFS worker nodes without user workloads using taints and tolerations. The NFS server `Pod` is armed with a `csi.hpe.com/hpe-nfs` toleration. It's required to taint dedicated NFS worker nodes if they truly need to be dedicated.

By default, the NFS Server Provisioner deploy resources in the "hpe-nfs" `Namespace`. This makes it easy to manage and diagnose. However, to use CSI data management capabilities (`VolumeSnapshots` and `.spec.dataSource`) on the PVCs, the NFS resources need to be deployed in the same `Namespace` as the "RWX"/"ROX" requesting `PVC`. This is controlled by the `nfsNamespace` `StorageClass` parameter. See [base `StorageClass` parameters](#base_storageclass_parameters) for more information.

!!! tip
    A comprehensive [tutorial is available](https://developer.hpe.com/blog/xABwJY56qEfNGMEo1lDj/introducing-a-nfs-server-provisioner-and-pod-monitor-for-the-hpe-csi-dri) on HPE Developer on how to get started with the NFS Server Provisioner and the HPE CSI Driver for Kubernetes. There's also a brief tutorial available in the [Video Gallery](../learn/video_gallery/index.md#multi-writer_workloads_using_the_nfs_server_provisioner).

Example `StorageClass` with "nfsResources" enabled. No CSP specific parameters for clarity.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-standard-file
provisioner: csi.hpe.com
parameters:
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
  description: "NFS backend volume created by the HPE CSI Driver for Kubernetes"
  csi.storage.k8s.io/fstype: ext4
  nfsResources: "true"
reclaimPolicy: Delete
allowVolumeExpansion: true
```

!!! note
    Using XFS may result in stale NFS handles during node failures and outages. Always use ext4 for NFS `PVCs`. While "allowVolumeExpansion" isn't supported on the NFS `PVC`, the backend "RWO" `PVC` does.

Example use of `accessModes`:

```yaml fct_label="ReadWriteOnce"
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

```yaml fct_label="ReadWriteMany"
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

```yaml fct_label="ReadOnlyMany"
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

In the case of declaring a "ROX" `PVC`, the requesting `Pod` specification needs to request the `PVC` as read-only. Example:

```yaml
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

#### Using a Foreign StorageClass

Since HPE CSI Driver for Kubernetes version 2.4.1 it's possible to provision NFS servers on top of non-HPE CSI Driver `StorageClasses`. The most prominent use case for this functionality is to coexist with the vSphere CSI Driver (VMware vSphere Container Storage Plug-in) in FC environments and provide "RWX" `PVCs`.

##### Example StorageClass using a foreign StorageClass

The HPE CSI Driver only manages the NFS server `Deployment`, `Service` and `PVC`. There must be an existing `StorageClass` capable of provisioning "RWO" filesystem `PVCs`.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-nfs-servers
provisioner: csi.hpe.com
parameters:
  nfsResources: "true"
  nfsForeignStorageClass: "my-foreign-storageclass-name"
reclaimPolicy: Delete
allowVolumeExpansion: false
```

Next, provision "RWO" or "RWX" claims from the "hpe-nfs-servers" `StorageClass`. An NFS server will be provisioned on a "RWO" `PVC` from the `StorageClass` "my-foreign-storageclass-name".

!!! note
    Only `StorageClasses` that uses HPE storage proxied by partner CSI drivers are supported by HPE.

#### Limitations and Considerations for the NFS Server Provisioner

These are some common issues and gotchas that are useful to know about when planning to use the NFS Server Provisioner.

- The current tested and supported limit for the NFS Server Provisioner is 32 NFS servers per Kubernetes worker node.
- The two `StorageClass` parameters "nfsResourceLimitsCpuM" and "nfsResourceLimitsMemoryMi" control how much CPU and memory it may consume. Tests show that the NFS server consumes about 150MiB at instantiation and 2GiB is the recommended minimum for most workloads. The NFS server `Pod` is by default limited to 2GiB or memory and 1000 milli CPU.
- The NFS `PVC` can **NOT** be expanded. If more capacity is needed, expand the "ReadWriteOnce" `PVC` backing the NFS Server Provisioner. This will result in inaccurate space reporting.
- Due to the fact that the NFS Server Provisioner deploys a number of different resources on the hosting cluster per `PVC`, provisioning times may differ greatly between clusters. On an idle cluster with the NFS Server Provisioning image cached, less than 30 seconds is the most common sighting but it may exceed 30 seconds which may trigger warnings on the requesting `PVC`. This is normal behavior.
- The HPE CSI Driver includes a Pod Monitor to delete `Pods` that have become unavailable due to the Pod status changing to `NodeLost` or a node becoming unreachable that the `Pod` runs on. By default the Pod Monitor only watches the NFS Server Provisioner `Deployments`. It may be used for any `Deployment`. See [Pod Monitor](monitor.md) on how to use it, especially the [limitations](monitor.md#limitations).
- Certain CNIs may have issues to gracefully restore access from the NFS clients to the NFS export. Flannel have exhibited this problem and the most consistent performance have been observed with Calico.
- The [Volume Mutation](#using_volume_mutations) feature does not work on the NFS `PVC`. If changes are needed, perform the change on the backing "ReadWriteOnce" `PVC`.
- As outlined in [Using the NFS Server Provisioner](#using_the_nfs_server_provisioner), CSI snapshots and cloning of NFS `PVCs` requires the CSI snapshot and NFS server to reside in the same `Namespace`. This also applies when using third-party backup software such as Kasten K10. Use the "nfsNamespace" `StorageClass` parameter to control where to provision resources.
- [VolumeGroups](using.md#volume_groups) and [SnapshotGroups](using.md#snapshot_groups) are only supported on the backing "ReadWriteOnce" `PVC`. The "volume-group" annotation may be set at the initial creation of the NFS `PVC` but will have adverse effect on logging as the Volume Group Provisioner tries to add the NFS `PVC` to the backend consistency group indefinitely.
- The NFS servers deployed by the HPE CSI Driver are not managed during CSI driver upgrades. Manual [upgrade is required](operations.md#upgrade_nfs_servers).
- Using the same network interface for NFS and block IO has shown suboptimal performance. Use FC for the block storage for the best performance.
- A single NFS server instance is capable of 100GigE wirespeed with large sequential workloads and up to 200,000 IOPS with small IO using bare-metal nodes and multiple clients.
- Using ext4 as the backing filesystem has shown better performance with simultaneous writers to the same file.
- Additional configuration and considerations may be required when using the NFS Server Provisioner with Red Hat OpenShift. See [NFS Server Provisioner Considerations](../partners/redhat_openshift/index.md#nfs_server_provisioner_considerations) for OpenShift.
- XFS has proven troublesome to use as a backend "RWO" volume filesystem, leaving stale NFS handles for clients. Use ext4 as the "csi.storage.k8s.io/fstype" `StorageClass` parameter for best results.

See [diagnosing NFS Server Provisioner issues](diagnostics.md#nfs_server_provisioner_resources) for further details.

### Using Volume Encryption

From version 2.0.0 and onwards of the CSI driver supports host-based volume encryption for any of the CSPs supported by the CSI driver.

Host-based volume encryption is controlled by `StorageClass` parameters configured by the Kubernetes administrator and may be configured to be overridden by Kubernetes users. In the below example, a single `Secret` is used to encrypt and decrypt all volumes provisioned by the `StorageClass`.

First, create a `Secret`, in this example we'll use the "hpe-storage" `Namespace`.

```yaml
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

```yaml
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

```yaml
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
        claimName: my-encrypted-pvc
```

Once the `Pod` comes up, verify that the volume is encrypted.

```text
$ kubectl exec -it my-pod -c pod-datelog-1 -- df -h /data
Filesystem              Size  Used Avail Use% Mounted on
/dev/mapper/enc-mpatha  100G   33M  100G   1% /data
```

Host-based volume encryption is in effect if the "enc" prefix is seen on the multipath device name.

!!! seealso 
    For an in-depth tutorial and more advanced use cases for host-based volume encryption, check out this blog post on HPE Developer: [Host-based Volume Encryption with HPE CSI Driver for Kubernetes](https://developer.hpe.com/blog/host-based-volume-encryption-with-hpe-csi-driver-for-kubernetes/)

### Topology and volumeBindingMode

With CSI driver v2.5.0 and newer, basic CSI topology information can be associated with a single backend from a `StorageClass`. For backwards compatibility, only `volumeBindingMode: WaitForFirstConsumer` require topology labels assigned to compute nodes. Using the default `volumeBindingMode` of `Immediate` will preserve the behavior prior to v2.5.0.

!!! tip
    The "csi-provisioner" is deployed with `--feature-gates Topology=true` and `--immediate-topology=false`. It's impact on volume provisioning and accessibility can be found [here](https://github.com/kubernetes-csi/external-provisioner?tab=readme-ov-file#topology-support).

Assume a simple use case where only a handful of nodes in a Kubernetes cluster have Fibre Channel adapters installed. Workloads with persistent storage requirements from a particular `StorageClass` should be deployed onto those nodes only.

#### Label Compute Nodes

Nodes with the label `csi.hpe.com/zone` are considered during topology accessibility assessments. Assume three nodes in the cluster have FC adapters.

```text
kubectl label node/my-node{1..3} csi.hpe.com/zone=fc --overwrite
```

If the CSI driver is already installed on the cluster, the CSI node driver needs to be restarted for the node labels to propagate.

```text
kubectl rollout restart -n hpe-storage ds/hpe-csi-node
```

#### Create StorageClass with Topology Information

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: hpe-standard-fc
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
  accessProtocol: fc
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: csi.hpe.com/zone
    values:
    - fc
```

Any workload provisioning `PVCs` from the above `StorageClass` will now be scheduled on nodes labeled `csi.hpe.com/zone=fc`.

!!! note
    The `allowedTopologies` key may be omitted if there's only a single topology applied to a subset of nodes. The nodes always need to be labeled when using `volumeBindingMode: WaitForFirstConsumer`. If all nodes have access to a backend, set `volumeBindingMode: Immediate` and omit `allowedTopologies`.

## Further Reading

The [official Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/) contains comprehensive documentation on how to markup `PersistentVolumeClaim` and `StorageClass` API objects to tweak certain behaviors.

Each CSP has a set of unique `StorageClass` parameters that may be tweaked to accommodate a wide variety of use cases. Please see the documentation of [the respective CSP for more details](../container_storage_provider/index.md).
