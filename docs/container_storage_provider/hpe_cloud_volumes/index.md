!!! error "Expired content"
    The documentation described on this page may be obsolete and contain references to unsupported and deprecated software. Please reach out to your HPE representative if you think you need any of the components referenced within.

# Introduction

The HPE Cloud Volumes CSP integrates seamlessly with the HPE Cloud Volumes Block service in the public cloud. The CSP abstracts the data management capabilities of the storage service for use by Kubernetes. The documentation found herein is mainly geared towards day-2 operations and reference documentation for the `StorageClass` and `VolumeSnapshotClass` parameters but also contains important HPE Cloud Volumes Block configuration details.

!!! important 
    The HPE Cloud Volumes CSP is currently in **beta** and available as a Tech Preview on Amazon EKS only. Please see the [1.5.0-beta Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver/1.5.0-beta).

[TOC]

!!! seealso
    There's a Tech Preview available in the [Video Gallery](../../learn/video_gallery/index.md#tech_preview_hybrid_cloud_kubernetes_using_hpe_cloud_volumes) on how to get started with the HPE Cloud Volumes CSP with the HPE CSI Driver.

## Cloud requirements

Always check the corresponding CSI driver version in [compatibility and support](../../csi_driver/index.md#compatibility_and_support) for basic requirements (such as supported Kubernetes version and cloud instance OS). If a certain feature is gated against any particular cloud provider it will be called out where applicable.

| Hyperscaler         | Managed Kubernetes               | BYO Kubernetes | Status       |
| ------------------- | -------------------------------- | -------------- | ------------ |
| Amazon Web Services | Elastic Kubernetes Service (EKS) | N/A            | Tech Preview | 
| Microsoft Azure     | Azure Kubernetes Service (AKS)   | TBA            | TBA          | 
| Google Cloud        | Google Kubernetes Engine (GKE)   | TBA            | TBA          | 

Additional hyperscaler support and BYO capabilities may become available in a future release of the CSP.

### Instance metadata

Kubernetes compute nodes will need to have access to the cloud provider's metadata services. This varies by cloud provider and is taken care of automatically by the HPE Cloud Volume CSP. The provided values may be overridden in the `StorageClass`, see [common parameters](#common_parameters_for_provisioning_and_cloning) for more information.

### Available regions

The HPE Cloud Volumes CSP may be deployed in the regions where the managed Kubernetes service control planes intersect with the HPE Cloud Volumes Block service.

| Region         | EKS                  | Azure | Google |
| -------------- | -------------------- | ----- | ------ |
| Americas       | us-east-1, us-west-2 | TBA   | TBA    |
| Europe         | eu-west-1, eu-west-2 | TBA   | TBA    |
| Asia Pacific   | ap-northest-1        | TBA   | TBA    |

Consider this table a snapshot of a particular moment in time and consult with the respective hyperscalers and the HPE Cloud Volumes Block service for definitive availability.

!!! note
    In other regions where HPE Cloud Volumes provide services, such as us-west-1, but cloud providers has no managed Kubernetes service; BYO Kubernetes is the only available option when it becomes available as a supported feature of the CSP.

## Limitations

Consult the [compatibility and support](../../csi_driver/index.md#compatibility_and_support) table for generic limitations and requirements. CSI and CSP specific limitations with HPE Cloud Volumes Block is listed below.

- The Volume Group Provisioner and Volume Group Snapshotter sidecars are currently not implemented in the HPE Cloud Volumes CSP.
- The base CSI driver parameter `description` is ignored by the CSP.
- In some cases, your a "regionID" needs to be supplied in the `StorageClass` and in conjunction with Ephemeral Inline Volumes. Your "regionID" may only be found in the APIs. Join us on [Slack](https://slack.hpedev.io/) if you're hitting this issue (it can be seen in the CSP logs).

!!! tip
    While not a limitation, iSCSI CHAP is mandatory with HPE Cloud Volumes but does not need to be configured within the CSI driver. The CHAP credentials are queried through the REST APIs from the HPE Cloud Volumes account session and applied automatically during runtime.

## StorageClass parameters

A `StorageClass` is used to provision or clone an HPE Cloud Volumes Block-backed persistent volume. It can also be used to import an existing Cloud Volume or clone of a snapshot into the Kubernetes cluster. The parameters are grouped below by those same workflows.

- [Common parameters for provisioning and cloning](#common_parameters_for_provisioning_and_cloning)
- [Provisioning parameters](#provisioning_parameters)
- [Pod inline volume parameters (Local Ephemeral Volumes)](#pod_inline_volume_parameters_local_ephemeral_volumes)
- [Cloning parameters](#cloning_parameters)
- [Import parameters](#import_parameters)
- [VolumeSnapshotClass parameters](#volumesnapshotclass_parameters)

Please see [using the HPE CSI Driver](../../csi_driver/using.md#base_storageclass_parameters) for base `StorageClass` examples. All parameters enumerated reflects the current version and may contain unannounced features and capabilities.

!!! note
    All parameters are optional unless documented as mandatory for a particular use case.

### Common parameters for provisioning and cloning

These parameters are mutable between a parent volume and creating a clone from a snapshot.

| Parameter                       | String  | Description |
| ------------------------------- | ------- | ----------- |
| destroyOnDelete                 | Boolean | Indicates the backing Cloud Volume (including snapshots) should be destroyed when the PVC is deleted. Defaults to "false" which means volumes needs to be pruned manually in the Cloud Volume service. |
| limitIops                       | Integer | The IOPS limit of the volume. The IOPS limit should be in the range 300 (default) to 20000. |
| performancePolicy<sup>1</sup>   | Text    | The name of the performance policy to assign to the volume. Available performance policies: "Exchange", "Oracle", "SharePoint", "SQL", "Windows File Server". Defaults to "Other Workloads". |
| schedule                        | Text    | Snapshot schedule to assign to the volumes. Available schedules: "hourly", "daily", "twicedaily", "weekly", "monthly", "none". Defaults to "daily". |
| retentionPolicy                 | Integer | Retention policy to assign to the `schedule`. The parameter must be paired properly with the `schedule`. <br /><br /><ul><li>hourly: 6, 12, 24<li>daily: 3, 7, 14<li>twicedaily: 4, 8, 14<li>weekly: 2, 4, 8<li>monthly: 3, 6, 12</ul>Defaults to "3" paired with the "daily" `retentionPolicy`.
| privateCloud<sup>1</sup>        | Text    | Override the compute instance provided VPC/VNET. |
| existingCloudSubnet<sup>1</sup> | Text    | Override the compute instance provided subnet. |
| automatedConnection<sup>1</sup> | Boolean | Override the HPE Cloud Volumes configured setting for connection automation. Connections between HPE Cloud Volumes and the desired VPC/VNET needs to be provisioned manually if set to "false". |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../csi_driver/using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameter is immutable and can't be altered after provisioning/cloning.
</small>

### Provisioning parameters

These parameters are immutable for both volumes and clones once created, clones will inherit parent attributes.

| Parameter       | String         | Description      |
| --------------- | -------------- | ---------------- |
| volumeType      | Text           | Volume type, General Purpose Flash ("GPF") or Premium Flash ("PF"). Defaults to "PF" |

### Pod inline volume parameters (Local Ephemeral Volumes)

These parameters are applicable only for Pod inline volumes and to be specified within Pod spec.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| csi.storage.k8s.io/ephemeral   | Boolean | Indicates that the request is for ephemeral inline volume. This is a mandatory parameter and must be set to "true".|
| inline-volume-secret-name      | Text    | A reference to the secret object containing sensitive information to pass to the CSI driver to complete the CSI NodePublishVolume call.|
| inline-volume-secret-namespace | Text    | The namespace of `inline-volume-secret-name` for ephemeral inline volume.|
| size                           | Text    | The size of ephemeral volume specified in MiB or GiB. If unspecified, a default value will be used.|

!!! important
    All parameters are **required** for inline ephemeral volumes.

### Cloning parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a PVC in the current namespace or use `importVolAsClone` and reference a Cloud Volume name to clone and import to Kubernetes.

| Parameter        | String  | Description |
| ---------------- | ------- | ----------- |
| cloneOf          | Text    | The name of the PV to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text    | The name of the Cloud Volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| snapshot         | Text    | The name of the snapshot to base the clone on. This is optional. If not specified, a new snapshot is created. |
| createSnapshot   | Boolean | Indicates that a new snapshot of the volume should be taken matching the name provided in the `snapshot` parameter. If the `snapshot` parameter is not specified, a default name will be created. |
| replStore        | Text    | Name of the Cloud Volume Replication Store to look for volumes, defaults to look outside of Replication Stores |

### Import parameters

Importing volumes to Kubernetes requires the source Cloud Volume to be disconnected. 

| Parameter          | String  | Description |
| ------------------ | ------- | ----------- |
| importVolumeName   | Text    | The name of the Cloud Volume to import. |
| forceImport        | Boolean | Allows import of volumes created on a different Kubernetes cluster other than the one importing the volume to. |

## VolumeSnapshotClass parameters

These parametes are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter   | String  | Description |
| ----------- | ------  | ----------- |
| description | Text    | Text to be added to the snapshot's description in the Cloud Volume service (optional) |
