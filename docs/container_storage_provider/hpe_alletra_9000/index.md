# Introduction

The HPE Alletra 9000 and Primera and 3PAR Storage Container Storage Provider (CSP) for Kubernetes is part of the [HPE CSI Driver for Kubernetes](../../csi_driver/index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes. 

[TOC]

!!! note
    For help getting started with deploying the HPE CSI Driver using HPE Alletra 9000, Primera or 3PAR storage, check out the [tutorial over at HPE DEV](https://developer.hpe.com/blog/9o7zJkqlX5cErkrzgopL/tutorial-how-to-get-started-with-the-hpe-csi-driver-and-hpe-primera-and-).

## Platform Requirements

Check the corresponding CSI driver version in the [compatibility and support](../../csi_driver/index.md#compatibility_and_support) table for the latest updates on supported Kubernetes version, orchestrators and host OS.

Refer to the HPE Single Point of Connectivity Knowledge (SPOCK) for specific platform details (requires an HPE Passport account) of the CSP. The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities. 

* [HPE Alletra 9000](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_alletra.html)
* [HPE Primera](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_primera.html)
* [HPE 3PAR](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_3par.html)

### User Role Requirements

The CSP requires access to a user with either `edit` or the `super` role. It's recommended to use the `edit` role for security best practices.

## StorageClass Parameters

All parameters enumerated reflects the current version and may contain unannounced features and capabilities. 

### Common Provisioning Parameters

| Parameter  | &nbsp;&nbsp;Option&nbsp;&nbsp;  | Description |
| ---------- | ------- | ----------- |
| accessProtocol <br /> (**Required**)  | fc | The access protocol to use when accessing the persistent volume. |
|                                       | iscsi | The access protocol to use when accessing the persistent volume. |
| cpg <sup>1</sup> | Text | The name of existing CPG to be used for volume provisioning. If the cpg parameter is not specified, the CSP will automatically set cpg parameter based upon a CPG available to the array. |
| snapCpg <sup>1</sup> | Text | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. |
| compression <sup>1</sup> | Boolean | Indicates that the volume should be compressed. (3PAR only) |
| provisioningType <sup>1</sup> <br /> (**Default: tpvv**) | tpvv | Indicates Thin provisioned volume type. |
|                               | full <sup>3</sup> | Indicates Full provisioned volume type. |
|                               | dedup <sup>3</sup> | Indicates Thin Deduplication volume type. |
|                               | reduce <sup>4</sup> | Indicates Data Reduction volume type. |
| hostSeesVLUN | Boolean | Enable "host sees" VLUN template. |
| importVol | Text | Name of the volume to import. |
| importVolAsClone | Text | Name of the volume to clone and import. |
| cloneOf <sup>2</sup> | Text | Name of the `PersistentVolumeClaim` to clone. |
| virtualCopyOf <sup>2</sup> | Text | Name of the `PersistentVolumeClaim` to snapshot. |
| qosName | Text | Name of the volume set which has QoS rules applied. |
| remoteCopyGroup <sup>1</sup> | Text | Name of a new or existing Remote Copy group on the array. |
| replicationDevices | Text | Indicates name of custom resource of type `hpereplicationdeviceinfos`. |
| allowBatchReplicatedVolumeCreation | Boolean | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single Remote Copy group. <br /> During this process, the Remote Copy group is stopped and started once. |
| oneRcgPerPvc | Boolean | Creates a dedicated Remote Copy group per persistent volume. |
| iscsiPortalIps | Text | Comma separated list of the array iSCSI port IPs. |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../csi_driver/using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameters that are editable after provisioning.
 <br /><sup>2</sup> = Volumes with snapshots/clones can't be modified.
 <br /><sup>3</sup> = HPE 3PAR only parameter
 <br /><sup>4</sup> = HPE Primera/Alletra 9000 only parameter
</small>

Please see [using the HPE CSI Driver](../../csi_driver/using.md#base_storageclass_parameters) for additional `StorageClass` examples like CSI snapshots and clones. 

!!! Important
    The HPE CSI Driver allows the `PersistentVolumeClaim` to override the `StorageClass` parameters by annotating the `PersistentVolumeClaim`. Please see [Using PVC Overrides](../../csi_driver/using.md#using_pvc_overrides) for more details.

### Ephemeral Inline Volumes

These parameters are applicable only for Pod inline volumes and to be specified within Pod spec.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| csi.storage.k8s.io/ephemeral   | Boolean | Indicates that the request is for ephemeral inline volume. This is a mandatory parameter and must be set to "true".|
| inline-volume-secret-name      | Text    | A reference to the secret object containing sensitive information to pass to the CSI driver to complete the CSI NodePublishVolume call.|
| inline-volume-secret-namespace | Text    | The namespace of `inline-volume-secret-name` for ephemeral inline volume.|
| size                           | Text    | The size of ephemeral volume specified in MiB or GiB. If unspecified, a default value will be used.|
| accessProtocol                 | Text    | Storage access protocol to use, "iscsi" or "fc".|

!!! important
    All parameters are **required** for inline ephemeral volumes.

### VLUN Templates

A VLUN template enables the export of a virtual volume as a VLUN to hosts. For more information, see the [HPE Primera OS Commmand Line Interface - Installation and Reference Guide](https://support.hpe.com/hpesc/public/docDisplay?docId=a00105286en_us&page=createvlun.html).

The HPE CSI Driver supports the following types of VLUN templates:

* **Matched set** - (Default) The VLUN is visible to initiators with the host's WWNs only on the specified port(s). 
* **Host sees** - The VLUN is visible to the initiators with any of the host's WWNs.

| Parameter    | String  | Description |
| ------------ | ------- | ----------- |
| hostSeesVLUN | Boolean | Enable "host sees" VLUN template. |

!!! note
    `hostSeeVLUN` is a mutable parameter. To modify an existing `PVC`, `hostSeesVLUN` needs to be specified with the `allowMutations` parameter along with editing the `PVC` with annotation `csi.hpe.com/hostSeesVLUN: "true"`. The HPE CSI Driver creates the vlun template based upon the `hostSeesVLUN` parameter during the volume publish operation. For the change to take effect, the `Pod` will need to be scheduled on another node by either deleting the `Pod` or draining the node.

### Importing Volumes

During the import volume process, any legacy (non-container volumes) or existing docker/k8s volume defined in the **ImportVol** parameter, within a `StorageClass`, will be renamed to match the `PersistentVolumeClaim` that leverages the `StorageClass`. The new volumes will be exposed through the HPE CSI Driver and made available to the Kubernetes cluster. **Note:** All previous Access Control Records and Initiator Groups will be removed from the volume when it is imported. The `hostSeesVLUN` parameter improves the performance in clusters where many pods are being migrated or created.

| Parameter      | Option      | Description |
| -------------- | ----------- | ----------- |
| accessProtocol | fc or iscsi | The access protocol to use when accessing the persistent volume. |
| importVol      | Text        | The name of the array volume to import. |

!!! important
    • **No other parameters** are required in the `StorageClass` when importing a volume outside of those parameters listed in the table above.<br />
    • Support for `importVol` is available from HPE CSI Driver 1.2.0+.

### Cloning Persistent Volumes

Cloning supports two modes of cloning. Either use `cloneOf` and reference a `PersistentVolumeClaim` in the current namespace to clone or use `importVolAsClone` and reference an array volume name to clone and import into the Kubernetes cluster. Volumes with clones are immutable once created.

| Parameter        | Option      | Description |
| ---------------- | ----------- | ----------- |
| cloneOf          | Text        | The name of the `PersistentVolumeClaim` to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text        | The name of the array volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| accessProtocol   | fc or iscsi | The access protocol to use when accessing the persistent volume. |

!!! important
    • **No other parameters** are required in the `StorageClass` while cloning outside of those parameters listed in the table above.<br />
    • Cloning using above parameters is independent of snapshot `CRD` availability on Kubernetes and it can be performed on any supported Kubernetes version.<br />
    • Support for `importVolAsClone` and `cloneOf` is available from HPE CSI Driver 1.3.0+.

### Creating Snapshots of Persistent Volumes

During the snapshotting process, any existing `PersistentVolumeClaim` defined in the `virtualCopyOf` parameter within a `StorageClass`, will be snapped as `PersistentVolumeClaim` and exposed through the HPE CSI Driver and made available to the Kubernetes cluster. Volumes with snapshots are immutable once created.

| Parameter          | Option      | Description |
| ------------------ | ----------- | ----------- |
| accessProtocol     | fc or iscsi | The access protocol to use when accessing the persistent volume. |
| virtualCopyOf      | Text        | The name of existing `PersistentVolumeClaim` to be snapped |

!!! important
    • **No other parameters** are required in the `StorageClass` when snapshotting a volume outside of those parameters listed in the table above.<br />
    • Snapshotting using `virtualCopyOf` is independent of snapshot `CRD` availability on Kubernetes and it can be performed on any supported Kubernetes version.<br />
    • Support for `virtualCopyOf` is available from HPE CSI Driver 1.3.0+.

### Remote Copy with Peer Persistence Synchronous Replication Parameters

To enable replication within the HPE CSI Driver, the following steps must be completed:

* Create `Secrets` for both primary and target arrays. Refer to [Configuring Additional Storage Backends](../../csi_driver/deployment.md#configuring_additional_storage_backends).
* Create replication custom resource.
* Create replication enabled `StorageClass`.

For a tutorial on how to enable replication, check out the blog [Enabling Remote Copy using the HPE CSI Driver for Kubernetes on HPE Primera](https://developer.hpe.com/blog/ppPAlQ807Ah8QGMNl1YE/tutorial-enabling-remote-copy-using-the-hpe-csi-driver-for-kubernetes-on)

A Custom Resource Definition (CRD) of type `hpereplicationdeviceinfos.storage.hpe.com`  must be created to define the target array information. The CRD object name will be used to define the `StorageClass` parameter **replicationDevices**. CRD mandatory parameters: `targetCpg`, `targetName`, `targetSecret` and `targetSecretNamespace`.


```yaml fct_label="HPE CSI Driver v2.1.0 and later"
apiVersion: storage.hpe.com/v2
kind: HPEReplicationDeviceInfo
metadata:
  name: r1
spec:
  target_array_details:
  - targetCpg: <cpg_name>
    targetSnapCpg: <snapcpg_name> #optional.
    targetName: <target_array_name>
    targetSecret: <target_secret_name>
    targetSecretNamespace: hpe-storage
```

```yaml fct_label="HPE CSI Driver v2.0.0"
apiVersion: storage.hpe.com/v1
kind: HPEReplicationDeviceInfo
metadata:
  name: r1
spec:
  target_array_details:
  - targetCpg: <cpg_name>
    targetSnapCpg: <snapcpg_name> #optional.
    targetName: <target_array_name>
    targetSecret: <target_secret_name>
    targetSecretNamespace: hpe-storage
```

!!! important
    **The HPE CSI Driver only supports Remote Copy Peer Persistence mode.**

These parameters are applicable only for replication. Both parameters are mandatory. If the Remote Copy volume group (RCG) name, as defined within the `StorageClass`, does not exist on the array, then a new RCG will be created.

| Parameter       | Option  | Description |
| --------------- | ------- | ----------- |
| remoteCopyGroup | Text    | Name of new or existing Remote Copy group on the array. |
| replicationDevices | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |
| allowBatchReplicatedVolumeCreation | Boolean | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single Remote Copy group. (Optional) <br /> During this process, the Remote Copy group is stopped and started once. |
| oneRcgPerPvc                       | Boolean | Creates a dedicated Remote Copy group per persistent volume. (Optional) |

!!! important
    HPE CSI Driver version 2.0 and before, the **Auto synchronize** and **Auto recover** policies for replicated volumes are not set automatically. To configure these policies, create a new Remote Copy group with **Auto Synchronize** and **Auto recover** enabled in SSMC or via CLI with the following command: <br/ > `setrcopygroup pol auto_recover,auto_synchronize <group_name>`

### Add Non-Replicated Volume to Remote Copy group

To add a non-replicated volume to an existing Remote Copy group, `allowMutations: description` at minimum must be defined within the `StorageClass`. Refer to [Remote Copy with Peer Persistence Replication](#remote_copy_with_peer_persistence_synchronous_replication_parameters) for more details.

Edit the non-replicated PVC and annotate the following parameters:

| Parameter       | Option  | Description |
| --------------- | ------- | ----------- |
| remoteCopyGroup | Text    | Name of existing Remote Copy group. |
| oneRcgPerPvc    | Boolean | Creates a dedicated Remote Copy group per persistent volume. (Optional) |
| replicationDevices | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |

!!! note
    `remoteCopyGroup` and `oneRcgPerPvc` parameters are mutually exclusive and cannot be added together when editing a `PVC`.

### Specifying iSCSI Target Portal IPs

This parameter allows the ability to specify a subset of the array iSCSI ports for iSCSI sessions. By default, the HPE CSI Driver uses all available iSCSI ports.

| Parameter      | Option | Description |
| -------------- | ------ | ----------- |
| iscsiPortalIps | Text   | Comma separated list of target portal IPs. |

### Creating a VolumeSnapshotClass

These parameters are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information. Volumes with snapshots are immutable.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter | String  | Description |
| --------- | ------  | ----------- |
| read_only | Boolean | Indicates if the snapshot is writable on the array. |

### Import a Snapshot

During the import snapshot process, any legacy (non-container snapshot) or an existing docker/k8s snapshot defined in the **ImportVol** parameter, within a `VolumeSnapshotClass`, will be renamed with the prefix "snapshot-". The new snapshot will be exposed through the HPE CSI Driver and made available to the Kubernetes cluster. **Note:** All previous Access Control Records and Initiator Groups will be removed from the snapshot when it is imported.

| Parameter | Option | Description |
| --------- | ------ | ----------- |
| importVol | Text   | The name of the array snapshot to import. |

### Creating a QoS StorageClass

In the array, QoS rules are applied to a volume set. To use an existing volume set with QoS rules on a `PersistentVolumeClaim`, set the `qosName` parameter within a `StorageClass` to the name of the existing array volume set.

| Parameter | Option | Description |
| --------- | ------ | ----------- |
| qosName   | Text   | Name of the HPE Primera or 3PAR volume set which has QoS rules. This parameter is optional. If specified, the `PersistentVolumeClaim` will be associated with the array volume set, for purposes of applying the QoS rules. |

### Creating a SnapshotGroupClass

These parameters are for `SnapshotGroupClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information. Volumes with snapshots are immutable.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter | String  | Description |
| --------- | ------- | ----------- |
| read_only | Boolean | Indicates if the snapshot is writable on the array. |

### Creating a VolumeGroupClass with QoS Settings

In the HPE CSI Driver version 1.4.0+, a volume set with QoS settings can be created dynamically using the QoS parameters for the `VolumeGroupClass`. The following parameters are available for a `VolumeGroup` on the array. Learn more about `VolumeGroups` in the [provisioning concepts documentation](../../csi_driver/using.md#volume_groups).

| Parameter    | String | Description |
| ------------ | ------ | ----------- |
| priority     | Text   | The priority level for the target volume set. Example: "low", "normal", "high"|
| ioMinGoal    | Text   | IOPS minimum goal for the target volume set. Example: "300" |
| ioMaxLimit   | Text   | IOPS maximum limit for the target volume set. Example: "10000" |
| bwMinGoalKb  | Text   | Bandwidth minimum goal in kilobytes per second for the target volume set. Example: "300" |
| bwMaxLimitKb | Text   | Bandwidth maximum limit in kilobytes per second for the target volume set. Example: "30000" |
| latencyGoal  | Text   | Latency goal in milliseconds (ms) or microseconds(us) for the target volume set. Example: "300ms" or "500us" |
| domain       | Text   | The array Virtual Domain, with which the volume group and related objects are associated with. Example: "sample_domain" |

!!! note
    QoS settings are mandatory when creating a VolumeGroupClass on the array.

### Specifying Fibre Channel Ports

Use this parameter to specify a subset of Fibre Channel (FC) ports on the array to create vluns. By default, the HPE CSI Driver uses all available FC ports.

| Parameter   | Option | Description |
| ----------- | ------ | ----------- |
| fcPortsList | Text   | Comma separated list of available FC ports. |

### Support

Please refer to the HPE Alletra 9000 and Primera and 3PAR Storage CSP [support statement](../../legal/support/index.md#hpe_primera_and_hpe_3par_container_storage_provider_support).
