# Introduction

The HPE Alletra 5000/6000 and Nimble Storage Container Storage Provider ("CSP") for Kubernetes is the reference implementation for the HPE CSI Driver for Kubernetes. The CSP abstracts the data management capabilities of the array for use by Kubernetes. The documentation found herein is mainly geared towards day-2 operations and reference documentation for the `StorageClass` and `VolumeSnapshotClass` parameters but also contains important array setup requirements.

!!! caution "Important"
    For a successful deployment, it's important to understand the array platform requirements found within the [CSI driver](../../index.md#compatibility_and_support) (compute node OS and Kubernetes versions) and the CSP.

[TOC]

!!! seealso
    There's a brief introduction on [how to use HPE Nimble Storage](../../../learn/video_gallery/index.md#using_the_hpe_csi_driver_with_hpe_nimble_storage) with the HPE CSI Driver in the Video Gallery. It also applies broadly to HPE Alletra 5000/6000.

## Platform Requirements

Always check the corresponding CSI driver version in [compatibility and support](../../index.md#compatibility_and_support) for the required array Operating System ("OS") version for a particular release of the driver. If a certain feature is gated against a certain version of the array OS it will be called out where applicable.

!!! tip
    The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities.

### Setting Up the Array

How to deploy an HPE storage array is beyond the scope of this document. Please refer to [HPE InfoSight](https://infosight.hpe.com) for further reading.

!!! error "Important"
    The HPE Nimble Storage Linux Toolkit (NLT) is **not** compatible with the HPE CSI Driver for Kubernetes. Do not install NLT on Kubernetes compute nodes. It may be installed on Kubernetes control plane nodes if they use iSCSI or FC storage from the array.

#### Network Requirements

The following ports needs to be reachable from the CSP running on the Kubernetes cluster to the management interface of the array.

| Port | Protocol | Description |
| ---- | -------- | ----------- |
| 443<sup>1</sup>  | HTTPS | Multitenant REST API |
| 5392 | HTTPS | REST API |

<small>
 <sup>1</sup> = The CSP will probe for multitenancy first, even if the platform doesn't support it. Leaving port 443 accessible is highly recommended to prevent slow control plane operations.
</small>

!!! info "Data networks"
    The data networks expects a flat network between the compute nodes and the array. Routing of iSCSI traffic is not supported by CSP.

#### Single Tenant Deployment

The CSP requires access to a user with either `poweruser` or the `administrator` role. It's recommended to use the `poweruser` role for least privilege practices.

!!! tip
    It's highly recommended to deploy a multitenant setup.

#### Multitenant Deployment

In array OS 6.0.0 and newer it's possible to create separate tenants using the `tenantadmin` CLI to assign folders to a tenant. This creates a secure and logical separation of storage resources between Kubernetes clusters.

No special configuration is needed on the Kubernetes cluster when using a tenant account or a regular user account. It's important to understand from a provisioning perspective that if the tenant account being used has been assigned multiple folders, the CSP will pick the folder with the most space available. If this is not desirable and a 1:1 `StorageClass` to Folder mapping is needed, the "folder" parameter needs to be called out in the `StorageClass`.

For reference, as of array OS 6.0.0, this is the `tenantadmin` command synopsis.

```text
$ tenantadmin --help
Usage: tenantadmin [options]
Manage Tenants.

Available options are:
  --help                           Program help.

  --list                           List Tenants.

  --info name                      Tenant info.

  --add tenant_name                Add a tenant.
    --folders folders              List of folder paths (comma separated
                                   pool_name:fqn) the tenant will be able to
                                   access (mandatory).

  --remove name                    Remove a tenant.

  --add_folder tenant_name         Add a folder path for tenant access.
    --name folder_name             Name of the folder path (pool_name:fqn) to
                                   be added (mandatory).

  --remove_folder tenant_name      Remove a folder path from tenant access.
    --name folder_name             Name of the folder path (pool_name:fqn) to
                                   be removed (mandatory).

  --passwd                         Change tenant's login password.
    --tenant name                  Change a specific tenant's login password
                                   (mandatory).
```

!!! caution
    The `tenantadmin` command may only be run by local array OS administrators. LDAP or Active Directory accounts, regardless of role, are not supported.

- Visit the array admin guide on [HPE InfoSight](https://infosight.hpe.com) to learn more about how to use the `tenantadmin` CLI.

##### Tenant Limitations

Some features may be limited and restricted in a multitenant deployment, such as arbitrarily import volumes in folders from the array the tenant isn't a user of, here are a few less obvious limitations.

- CHAP is configured globally for the CSI driver. The CSI driver is contracted to create the CHAP user if it doesn't exist. It's important that the CHAP user does not exist prior when used with a tenant, as tenant may not share CHAP users among themselves or the admin account.
- Both port 443 and 5392 needs to be exposed to the Kubernetes cluster in multitenant deployments.

!!! seealso
    An in-depth tutorial on how to use multitenancy and the `tenantadmin` CLI is available on HPE Developer: [Multitenancy for Kubernetes clusters using HPE Alletra 5000/6000 and Nimble Storage](https://developer.hpe.com/blog/multitenancy-for-kubernetes-clusters-using-hpe-alletra-6000-and-nimble-storage/). There's also a high level overview of multitenancy available as a [lightboard presentation on YouTube](https://www.youtube.com/watch?v=47txpSAGEZs).

### Limitations

Consult the [compatibility and support](../../index.md#compatibility_and_support) table for supported array OS versions. CSI and CSP specific limitations are listed below.

- Striped volumes on grouped arrays are not supported by the CSI driver.
- The CSP is not capable of provisioning or importing volumes protected by Peer Persistence.
- When using an FC only array and provisioning RWX block volumes, the "multi_initiator" attribute won't get set properly on the volume. The workaround is to run `group --edit --iscsi_enabled yes` on the Array OS CLI.
- iSCSI does not work and is not supported on routed data networks. The REST API traffic may be routed and is supported.

## StorageClass Parameters

A `StorageClass` is used to provision or clone a persistent volume. It can also be used to import an existing volume or clone a snapshot into the Kubernetes cluster. The parameters are grouped below by those same workflows.

- [Common parameters for provisioning and cloning](#common_parameters_for_provisioning_and_cloning)
- [Provisioning Parameters](#provisioning_parameters)
- [Cloning Parameters](#cloning_parameters)
- [Import Parameters](#import_parameters)
- [Pod Inline Volume Parameters (Local Ephemeral Volumes)](#pod_inline_volume_parameters_local_ephemeral_volumes)
- [VolumeGroupClass Parameters](#volumegroupclass_parameters)
- [VolumeSnapshotClass Parameters](#volumesnapshotclass_parameters)

Backward compatibility with the HPE Nimble Storage FlexVolume driver is being honored to a certain degree. `StorageClass` API objects needs be rewritten and parameters need to be updated regardless.

Please see [using the HPE CSI Driver](../../using.md#base_storageclass_parameters) for base `StorageClass` examples. All parameters enumerated reflects the current version and may contain unannounced features and capabilities.

Example default `StorageClass` ([download](examples/storageclass.yaml)):

```yaml
{% include "csi_driver/container_storage_provider/hpe_alletra_6000/examples/storageclass.yaml" %}```

!!! hint
    If all mutable parameters have values provided during provisioning of the `PersistentVolumes`, the [Volume Mutator](../../using.md#using_volume_mutations) will later allow changes if needed.

### Common Parameters for Provisioning and Cloning

These parameters are mutable between a parent volume and creating a clone from a snapshot.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| accessProtocol<sup>1</sup>     | Text    | The access protocol to use when accessing the persistent volume ("fc" or "iscsi").  Defaults to "iscsi" when unspecified. |
| destroyOnDelete                | Boolean | Indicates the backing Nimble volume (including snapshots) should be destroyed when the PVC is deleted. Defaults to "false" which means volumes needs to be pruned manually. |
| limitIops                      | Integer | The IOPS limit of the volume. The IOPS limit should be in the range 256 to 4294967294, or -1 for unlimited (default). |
| limitMbps                      | Integer | The MB/s throughput limit for the volume between 1 and 4294967294, or -1 for unlimited (default).|
| description                    | Text    | Text to be added to the volume's description on the array. Empty string by default. |
| performancePolicy<sup>2</sup>  | Text    | The name of the performance policy to assign to the volume. Default example performance policies include "Backup Repository", "Exchange 2003 data store", "Exchange 2007 data store", "Exchange 2010 data store", "Exchange log", "Oracle OLTP", "Other Workloads", "SharePoint", "SQL Server", "SQL Server 2012", "SQL Server Logs". Defaults to the "default" performance policy. |
| protectionTemplate<sup>4</sup> | Text    | The name of the protection template to assign to the volume. Default examples of protection templates include "Retain-30Daily", "Retain-48Hourly-30Daily-52Weekly", and "Retain-90Daily". |
| folder                         | Text    | The name of the folder in which to place the volume. Defaults to the root of the "default" pool. |
| thick                          | Boolean | Indicates that the volume should be thick provisioned. Defaults to "false" |
| dedupeEnabled<sup>3</sup>      | Boolean | Indicates that the volume should enable deduplication. Defaults to "true" when available. |
| syncOnDetach                   | Boolean | Indicates that a snapshot of the volume should be synced to the replication partner each time it is detached from a node. Defaults to "false". |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameter is immutable and can't be altered after provisioning/cloning.
 <br /><sup>2</sup> = Performance policies may only be mutated between performance polices with the same block size.
 <br /><sup>3</sup> = Deduplication may only be mutated within the same performance policy application category and block size.
 <br /><sup>4</sup> = This parameter was removed in HPE CSI Driver 1.4.0 and replaced with [`VolumeGroupClasses`](#volumegroupclass_parameters).
</small>

!!! note
    Performance Policies, Folders and Protection Templates are array OS specific constructs that can be created on the array itself to address particular requirements or workloads. Please consult with the storage admin or read the admin guide found on [HPE InfoSight](https://infosight.hpe.com).

### Provisioning Parameters

These parameters are immutable for both volumes and clones once created, clones will inherit parent attributes.

| Parameter       | String         | Description |
| --------------- | -------------- | ----------- |
| encrypted       | Boolean        | Indicates that the volume should be encrypted. Defaults to "false". |
| pool            | Text           | The name of the pool in which to place the volume. Defaults to the "default" pool. |

### Cloning Parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a PVC in the current namespace or use `importVolAsClone` and reference an array volume name to clone and import to Kubernetes.

| Parameter        | String  | Description |
| ---------------- | ------- | ----------- |
| cloneOf          | Text    | The name of the PV to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text    | The name of the array volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| snapshot         | Text    | The name of the snapshot to base the clone on. This is optional. If not specified, a new snapshot is created. |
| createSnapshot   | Boolean | Indicates that a new snapshot of the volume should be taken matching the name provided in the `snapshot` parameter. If the `snapshot` parameter is not specified, a default name will be created. |

### Import Parameters

Importing volumes to Kubernetes requires the source array volume to be offline. In case of reverse replication, the upstream volume should be in offline state. All previous Access Control Records and Initiator Groups will be stripped from the volume when put under control of the HPE CSI Driver.

| Parameter          | String  | Description |
| ------------------ | ------- | ----------- |
| importVolumeName   | Text    | The name of the array volume to import. |
| snapshot           | Text    | The name of the array snapshot to restore the imported volume to after takeover. If not specified, the volume will not be restored. |
| takeover           | Boolean | Indicates the current group will takeover ownership of the array volume and volume collection. This should be performed against a downstream replica. |
| reverseReplication | Boolean | Reverses the replication direction so that writes to the array volume are replicated back to the group where it was replicated from. |
| forceImport        | Boolean | Forces the import of a volume that is not owned by the group and is not part of a volume collection. If the volume is part of a volume collection, use takeover instead. |

!!! seealso
    In this [HPE Developer blog post](https://developer.hpe.com/blog/lift-and-transform-apps-with-hpe-csi-driver-for-kubernetes/) you'll learn how to use the import parameters to lift and transform applications from traditional infrastructure to Kubernetes using the HPE CSI Driver.

## Pod Inline Volume Parameters (Local Ephemeral Volumes)

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

## VolumeGroupClass Parameters

If basic data protection is required and performed on the array, `VolumeGroups` needs to be created, even it's just a single volume that needs data protection using snapshots and replication. Learn more about `VolumeGroups` in the [provisioning concepts documentation](../../using.md#volume_groups).

| Parameter          | String  | Description |
| ------------------ | ------- | ----------- |
| description        | Text    | Text to be added to the volume collection description on the array. Empty by default. |
| protectionTemplate | Text    | The name of the protection template to assign to the volume collection. Default examples of protection templates include "Retain-30Daily", "Retain-48Hourly-30Daily-52Weekly", and "Retain-90Daily". Empty by default, meaning no array snapshots are performed on the `VolumeGroups`. |

!!! tip "New feature"
    `VolumeGroupClasses` were introduced with version 1.4.0 of the CSI driver. Learn more in the [Using section](../../using.md#volume_groups).

## VolumeSnapshotClass Parameters

These parametes are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../using.md#enabling_csi_snapshots) for more information.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../using.md#using_csi_snapshots).

| Parameter   | String  | Description |
| ----------- | ------  | ----------- |
| description | Text    | Text to be added to the snapshot's description on the array. |
| writable    | Boolean | Indicates if the snapshot is writable on the array. Defaults to "false". |
| online      | Boolean | Indicates if the snapshot is set to online on the array. Defaults to "false". |

## Static Provisioning

Static provisioning of `PVs` and `PVCs` may be used when absolute control over physical volumes are required by the storage administrator. This CSP also supports importing volumes and clones of volumes using the [import parameters](#import_parameters) in a `StorageClass`.

### Persistent Volume

Create a `PV` referencing an existing 10GiB volume on the array, replace `.spec.csi.volumeHandle` with the array volume ID.

!!! warning
    If a filesystem can't be detected on the device a new filesystem will be created. If the volume contains data, make sure the data reside in a whole device filesystem.

```text
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-static-pv-1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  csi:
    volumeHandle: <insert volume ID here>
    driver: csi.hpe.com
    fsType: xfs
    volumeAttributes:
      volumeAccessMode: mount
      fsType: xfs
    controllerPublishSecretRef:
      name: hpe-backend
      namespace: hpe-storage
    nodePublishSecretRef:
      name: hpe-backend
      namespace: hpe-storage
    controllerExpandSecretRef:
      name: hpe-backend
      namespace: hpe-storage
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
```

!!! tip
    Remove `.spec.csi.controllerExpandSecretRef` to disallow volume expansion.

### Persistent Volume Claim

Now, a user may claim the static `PV` by creating a `PVC` referencing the `PV` name in `.spec.volumeName`.

```text
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: my-static-pv-1
  storageClassName: ""
```
