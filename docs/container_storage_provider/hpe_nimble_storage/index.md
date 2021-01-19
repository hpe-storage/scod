# Introduction

The HPE Nimble Storage CSP is the reference implementation for the HPE CSI Driver for Kubernetes. The CSP abstracts the data management capabilities of the array for use by Kubernetes. The documentation found herein is mainly geared towards day two operations and reference documentation for the `StorageClass` and `VolumeSnapshotClass` parameters but also contains important Nimble array setup requirements.

!!! caution "Important"
    For a successful deployment, it's important to understand the Nimble platform requirements found within the [CSI driver](../../csi_driver/index.md#compatibility_and_support) (worker host OS and Kubernetes versions) and the CSP.

[TOC]

!!! seealso
    There's a brief introduction on [how to use HPE Nimble Storage](../../learn/video_gallery/index.md#using_the_hpe_csi_driver_with_hpe_nimble_storage) with the HPE CSI Driver in the Video Gallery.

## Platform requirements

Always check the corresponding CSI driver version in [compatibility and support](../../csi_driver/index.md#compatibility_and_support) for the required NimbleOS version for a particular release of the driver. If a certain feature is gated against a certain version of NimbleOS it will be called out where applicable.

!!! tip
    The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities.

### Setting up the Nimble array

How to deploy an HPE Nimble Storage array is beyond the scope of this document. Please refer to [HPE InfoSight](https://infosight.hpe.com) for further reading.

#### Single tenant deployment

The CSP requires access to a user with either `poweruser` or the `administrator` role. It's recommended to use the `poweruser` role for least privilege practices.

## StorageClass parameters

A `StorageClass` is used to provision or clone an HPE Nimble Storage-backed persistent volume. It can also be used to import an existing HPE Nimble Storage volume or clone of a snapshot into the Kubernetes cluster. The parameters are grouped below by those same workflows.

- [Common parameters for provisioning and cloning](#common_parameters_for_provisioning_and_cloning)
- [Provisioning parameters](#provisioning_parameters)
- [Pod inline volume parameters (Local Ephemeral Volumes)](#pod_inline_volume_parameters_local_ephemeral_volumes)
- [Cloning parameters](#cloning_parameters)
- [Import parameters](#import_parameters)
- [VolumeSnapshotClass parameters](#volumesnapshotclass_parameters)
- [VolumeGroupClass parameters](#volumesnapshotclass_parameters)

Backward compatibility with the HPE Nimble Storage FlexVolume driver is being honored to a certain degree. `StorageClass` API objects needs be rewritten and parameters need to be updated regardless.

Please see [using the HPE CSI Driver](../../csi_driver/using.md#base_storageclass_parameters) for base `StorageClass` examples. All parameters enumerated reflects the current version and may contain unannounced features and capabilities.

!!! note
    These are optional parameters unless specified.

### Common parameters for provisioning and cloning

These parameters are mutable between a parent volume and creating a clone from a snapshot.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| accessProtocol<sup>1</sup>     | Text    | The access protocol to use when accessing the persistent volume ("fc" or "iscsi").  Defaults to "iscsi" when unspecified. |
| destroyOnDelete                | Boolean | Indicates the backing Nimble volume (including snapshots) should be destroyed when the PVC is deleted. Defaults to "false" which means volumes needs to be pruned manually. |
| limitIops                      | Integer | The IOPS limit of the volume. The IOPS limit should be in the range 256 to 4294967294, or -1 for unlimited (default). |
| limitMbps                      | Integer | The MB/s throughput limit for the volume between 1 and 4294967294, or -1 for unlimited (default).|
| description                    | Text    | Text to be added to the volume's description on the Nimble array. Empty string by default. |
| performancePolicy<sup>2</sup>  | Text    | The name of the performance policy to assign to the volume. Default example performance policies include "Backup Repository", "Exchange 2003 data store", "Exchange 2007 data store", "Exchange 2010 data store", "Exchange log", "Oracle OLTP", "Other Workloads", "SharePoint", "SQL Server", "SQL Server 2012", "SQL Server Logs". Defaults to the "default" performance policy. |
| protectionTemplate<sup>1</sup> | Text    | The name of the protection template to assign to the volume. Default examples of protection templates include "Retain-30Daily", "Retain-48Hourly-30Daily-52Weekly", and "Retain-90Daily". |
| folder                         | Text    | The name of the Nimble folder in which to place the volume. Defaults to the root of the "default" pool. |
| thick                          | Boolean | Indicates that the volume should be thick provisioned. Defaults to "false" |
| dedupeEnabled<sup>3</sup>      | Boolean | Indicates that the volume should enable deduplication. Defaults to "true" when available. |
| syncOnDetach                   | Boolean | Indicates that a snapshot of the volume should be synced to the replication partner each time it is detached from a node. Defaults to "false". |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../csi_driver/using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameter is immutable and can't be altered after provisioning/cloning. This parameter is removed in HPE CSI Driver 1.4.0 and replaced with `[VolumeGroupClasses](#volumegroupclass_parameters)`.
 <br /><sup>2</sup> = Performance policies may only be mutated between performance polices with the same block size.
 <br /><sup>3</sup> = Deduplication may only be mutated within the same performance policy application category and block size.
</small>

!!! note
    Performance Policies, Folders and Protection Templates are Nimble specific constructs that can be created on the Nimble array itself to address particular requirements or workloads. Please consult with the storage admin or read the admin guide found on [HPE InfoSight](https://infosight.hpe.com).

### Provisioning parameters

These parameters are immutable for both volumes and clones once created, clones will inherit parent attributes.

| Parameter       | String         | Description |
| --------------- | -------------- | ----------- |
| encrypted       | Boolean        | Indicates that the volume should be encrypted. Defaults to "false". |
| pool            | Text           | The name of the pool in which to place the volume. Defaults to the "default" pool. |

### Pod inline volume parameters (Local Ephemeral Volumes)

These parameters are applicable only for Pod inline volumes and to be specified within Pod spec.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| csi.storage.k8s.io/ephemeral   | Boolean | Indicates that the request is for ephemeral inline volume. This is a mandatory parameter and must be set to "true".|
| inline-volume-secret-name      | Text    | A reference to the secret object containing sensitive information to pass to the CSI driver to complete the CSI NodePublishVolume call.|
| inline-volume-secret-namespace | Text    | The namespace of `inline-volume-secret-name` for ephemeral inline volume.|
| size                           | Text    | The size of ephemeral volume specified in MiB or GiB. If unspecified, a default value will be used.|
| accessProtocol                 | Text    | Storage access protocol to use, "iscsi" or "fc".

!!! important
    All parameters are **required** for inline ephemeral volumes.

### Cloning parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a PVC in the current namespace or use `importVolAsClone` and reference a Nimble volume name to clone and import to Kubernetes.

| Parameter        | String  | Description |
| ---------------- | ------- | ----------- |
| cloneOf          | Text    | The name of the PV to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text    | The name of the Nimble volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| snapshot         | Text    | The name of the snapshot to base the clone on. This is optional. If not specified, a new snapshot is created. |
| createSnapshot   | Boolean | Indicates that a new snapshot of the volume should be taken matching the name provided in the `snapshot` parameter. If the `snapshot` parameter is not specified, a default name will be created. |

### Import parameters

Importing volumes to Kubernetes requires the source Nimble volume to be offline. In case of reverse replication, the upstream volume should be in offline state. All previous Access Control Records and Initiator Groups will be stripped from the volume when put under control of the HPE CSI Driver.

| Parameter          | String  | Description |
| ------------------ | ------- | ----------- |
| importVolumeName   | Text    | The name of the Nimble volume to import. |
| snapshot           | Text    | The name of the Nimble snapshot to restore the imported volume to after takeover. If not specified, the volume will not be restored. |
| takeover           | Boolean | Indicates the current group will takeover ownership of the Nimble volume and volume collection. This should be performed against a downstream replica. |
| reverseReplication | Boolean | Reverses the replication direction so that writes to the Nimble volume are replicated back to the group where it was replicated from. |
| forceImport        | Boolean | Forces the import of a volume that is not owned by the group and is not part of a volume collection. If the volume is part of a volume collection, use takeover instead. |

## VolumeGroupClass parameters

If basic data protection is required and performed on the Nimble array, `VolumeGroups` needs to be created, even it's just a single volume that needs data protection using snapshots and replication. Learn more about `VolumeGroups` in the [provisioning concepts documentation](../../csi_driver/using.md#volume_groups).

| Parameter          | String  | Description |
| ------------------ | ------- | ----------- |
| description        | Text    | Text to be added to the volume collection description on the Nimble array. Empty by default. |
| protectionTemplate | Text    | The name of the protection template to assign to the volume collection. Default examples of protection templates include "Retain-30Daily", "Retain-48Hourly-30Daily-52Weekly", and "Retain-90Daily". Empty by default, meaning no array snapshots are performed on the `VolumeGroups`. |

!!! tip "New feature"
    `VolumeGroupClasses` were introduced with version 1.4.0 of the CSI driver. Learn more in the [Using section](../../csi_driver/using.md#volume_groups).

## VolumeSnapshotClass parameters

These parametes are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter   | String  | Description |
| ----------- | ------  | ----------- |
| description | Text    | Text to be added to the snapshot's description on the Nimble array. |
| writable    | Boolean | Indicates if the snapshot is writable on the Nimble array. Defaults to "false". |
| online      | Boolean | Indicates if the snapshot is set to online on the Nimble array. Defaults to "false". |
