# Introduction

The HPE Alletra Storage MP, Alletra 9000 and Primera and 3PAR Storage Container Storage Provider (CSP) for Kubernetes is part of the [HPE CSI Driver for Kubernetes](../../csi_driver/index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes. 

!!! note
    The HPE CSI Driver for Kubernetes is only compatible with HPE Alletra Storage MP running with block services, such as HPE GreenLake for Block Storage.

[TOC]

!!! note
    For help getting started with deploying the HPE CSI Driver using HPE Alletra Storage MP, Alletra 9000, Primera or 3PAR storage, check out the [tutorial over at HPE Developer](https://developer.hpe.com/blog/9o7zJkqlX5cErkrzgopL/tutorial-how-to-get-started-with-the-hpe-csi-driver-and-hpe-primera-and-).

## Platform Requirements

Check the corresponding CSI driver version in the [compatibility and support](../../csi_driver/index.md#compatibility_and_support) table for the latest updates on supported Kubernetes version, orchestrators and host OS.
<!-- Refer to the HPE Single Point of Connectivity Knowledge (SPOCK) for specific platform details (requires an HPE Passport account) of the CSP. The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities.

* [HPE Alletra Storage MP](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_greenlake_block.html)
* [HPE Alletra 9000](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_alletra.html)
* [HPE Primera](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_primera.html)
* [HPE 3PAR](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_3par.html) -->

### Network Port Requirements

The HPE Alletra Storage MP, Alletra 9000, Primera and 3PAR Container Storage Provider requires the following TCP ports to be open inbound to the array from the Kubernetes cluster worker nodes running the HPE CSI Driver for Kubernetes.

| Port | Protocol | Description |
| ---- | -------- | ----------- |
| 443 | HTTPS | WSAPI (HPE Alletra Storage MP, Alletra 9000/Primera) |
| 8080 | HTTPS | WSAPI (HPE 3PAR) |
| 22 | SSH | Array communication |

### User Role Requirements

The CSP requires access to a local user with either `edit` or the `super` role. It's recommended to use the `edit` role for security best practices.

!!! note
    LDAP users are not supported by the CSP.

### Virtual Domains

Virtual Domains are not yet fully supported by the CSP. From HPE CSI Driver v2.5.0, it's possible to manually create the Kubernetes hosts connecting to storage within the Virtual Domain. Once the hosts have been created, deploy the CSI driver with the Helm chart using the "disableHostDeletion" parameter set to "true". The Virtual Domain user may create the hosts through the Virtual Domain if the "AllowDomainUsersAffectNoDomain" parameter is set to either "hostonly" or "yes" on the array.

!!! note
    Remote Copy Groups managed by the CSP have not been tested with Virtual Domains at this time.

## VLUN Templates

A VLUN template enables the export of a virtual volume as a VLUN to hosts. For more information, see the [HPE Primera OS Commmand Line Interface - Installation and Reference Guide](https://support.hpe.com/hpesc/public/docDisplay?docId=a00105286en_us&page=createvlun.html).

The CSP supports the following types of VLUN templates:

| Template | Description |
| -------- | ----------- |
| Matched&nbsp;set | The default VLUN template. The VLUN is visible to initiators with the host's WWNs only on the specified port(s). |
| Host sees | The VLUN is visible to the initiators with any of the host's WWNs. |

The boolean string "hostSeesVLUN" `StorageClass` parameter controls which VLUN template to use.

!!! warning "Recommendation"
    In most scenarios, "hostSeesVLUN" should be set to "true".

### Change VLUN Template for existing PVCs

To modify an existing `PVC`, "hostSeesVLUN" needs to be specified with the "allowMutations" parameter along with adding the `PVC` annotation "csi.hpe.com/hostSeesVLUN" with the string values of either "true" or "false". The HPE CSI Driver creates the VLUN template based upon the `hostSeesVLUN` parameter during the volume publish operation. For the change to take effect, the `Pod` will need to be scheduled on another node by either deleting the `Pod` or draining the node.

## StorageClass Parameters

All parameters enumerated reflects the current version and may contain unannounced features and capabilities. 

### Common Provisioning Parameters

<!--
| cpg <sup>1</sup> | Text | The name of existing CPG to be used for volume provisioning. If the `cpg` parameter is not specified, the CSP will automatically set `cpg` parameter based upon a CPG available to the array. |
| snapCpg <sup>1</sup> | Text | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. |
| compression <sup>1</sup> | Boolean | Indicates that the volume should be compressed. (3PAR only) |
-->

| Parameter  | &nbsp;&nbsp;Option&nbsp;&nbsp;  | Description |
| ---------- | ------- | ----------- |
| accessProtocol (**Required**)  | fc or iscsi | The access protocol to use when attaching the persistent volume. |
| cpg <sup>1</sup> | Text | The name of existing CPG to be used for volume provisioning. If the `cpg` parameter is not specified, the CSP will select a CPG available to the array. |
| snapCpg <sup>1</sup> | Text | The name of the snapshot CPG to be used for volume provisioning. **Needs to be set if any kind of `VolumeSnapshots` or `PVC` cloning parameters are used.** |
| compression <sup>1</sup> | Boolean | Indicates that the volume should be compressed. (3PAR only) |
| provisioningType <sup>1</sup> | tpvv | Default. Indicates Thin provisioned volume type. |
|                               | full <sup>3</sup> | Indicates Full provisioned volume type. |
|                               | dedup <sup>3</sup> | Indicates Thin Deduplication volume type. |
|                               | reduce <sup>4</sup> | Indicates Data Reduction volume type. |
| hostSeesVLUN | Boolean | Enable "host sees" VLUN template. |
| importVolumeName | Text | Name of the volume to import. |
| importVolAsClone | Text | Name of the volume to clone and import. |
| cloneOf <sup>2</sup> | Text | Name of the `PersistentVolumeClaim` to clone. |
| virtualCopyOf <sup>2</sup> | Text | Name of the `PersistentVolumeClaim` to snapshot. |
| qosName | Text | Name of the volume set which has QoS rules applied. |
| remoteCopyGroup <sup>1</sup> | Text | Name of a new or existing Remote Copy group on the array. |
| replicationDevices | Text | Indicates name of custom resource of type `hpereplicationdeviceinfos`. |
| allowBatchReplicatedVolumeCreation | Boolean | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single Remote Copy group. <br /> During this process, the Remote Copy group is stopped and started once. |
| oneRcgPerPvc | Boolean | Creates a dedicated Remote Copy group per persistent volume. |
| iscsiPortalIps | Text | Comma separated list of the array iSCSI port IPs. |
| fcPortsList | Text   | Comma separated list of available FC ports. Example: "0:5:1,1:4:2,2:4:1,3:4:2" Default: Use all available ports. |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../csi_driver/using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameters that are editable after provisioning.
 <br /><sup>2</sup> = Volumes with snapshots/clones can't be modified.
 <br /><sup>3</sup> = HPE 3PAR only parameter
 <br /><sup>4</sup> = HPE Primera/Alletra 9000 only parameter
</small>

Please see [using the HPE CSI Driver](../../csi_driver/using.md#base_storageclass_parameters) for additional `StorageClass` examples like CSI snapshots and clones. 

!!! important
    The HPE CSI Driver allows the `PersistentVolumeClaim` to override the `StorageClass` parameters by annotating the `PersistentVolumeClaim`. Please see [Using PVC Overrides](../../csi_driver/using.md#using_pvc_overrides) for more details.

### Cloning Parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a `PersistentVolumeClaim` in the current namespace to clone or use `importVolAsClone` and reference an array volume name to clone and import into the Kubernetes cluster. Volumes with clones are immutable once created.

| Parameter        | Option      | Description |
| ---------------- | ----------- | ----------- |
| cloneOf          | Text        | The name of the `PersistentVolumeClaim` to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text        | The name of the array volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| accessProtocol   | fc or iscsi | The access protocol to use when attaching the cloned volume. |

!!! important
    • **No other parameters** are required in the `StorageClass` while cloning outside of those parameters listed in the table above.<br />
    • Cloning using above parameters is independent of snapshot `CRD` availability on Kubernetes and it can be performed on any supported Kubernetes version.<br />
    • Support for `importVolAsClone` and `cloneOf` is available from HPE CSI Driver 1.3.0+.

### Array Snapshot Parameters

During the snapshotting process, any existing `PersistentVolumeClaim` defined in the `virtualCopyOf` parameter within a `StorageClass`, will be snapped as `PersistentVolumeClaim` and exposed through the HPE CSI Driver and made available to the Kubernetes cluster. Volumes with snapshots are immutable once created.

| Parameter          | Option      | Description |
| ------------------ | ----------- | ----------- |
| accessProtocol     | fc or iscsi | The access protocol to use when attaching the snapshot volume. |
| virtualCopyOf      | Text        | The name of existing `PersistentVolumeClaim` to be snapped |

!!! important
    • **No other parameters** are required in the `StorageClass` when snapshotting a volume outside of those parameters listed in the table above.<br />
    • Snapshotting using `virtualCopyOf` is independent of snapshot `CRD` availability on Kubernetes and it can be performed on any supported Kubernetes version.<br />
    • Support for `virtualCopyOf` is available from HPE CSI Driver 1.3.0+.

### Import Parameters

During the import volume process, any legacy (non-container volumes) defined in the **ImportVol** parameter, within a `StorageClass`, will be renamed to match the `PersistentVolumeClaim` that leverages the `StorageClass`. The new volumes will be exposed through the HPE CSI Driver and made available to the Kubernetes cluster. **Note:** All previous Access Control Records and Initiator Groups will be removed from the volume when it is imported.

| Parameter      | Option      | Description |
| -------------- | ----------- | ----------- |
| accessProtocol | fc or iscsi | The access protocol to use when importing the volume. |
| importVolumeName | Text        | The name of the array volume to import. |

!!! important
    • **No other parameters** are required in the `StorageClass` when importing a volume outside of those parameters listed in the table above.<br />
    • Support for `importVolumeName` is available from HPE CSI Driver 1.2.0+.

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
| remoteCopyGroup | Text    | Name of new or existing Remote Copy group <sup>1</sup> on the array. |
| replicationDevices | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |
| allowBatchReplicatedVolumeCreation | Boolean | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single Remote Copy group. (Optional) <br /> During this process, the Remote Copy group is stopped and started once. |
| oneRcgPerPvc                       | Boolean | Creates a dedicated Remote Copy group per persistent volume. (Optional) |

<small>
 Remote Copy additional details: 
 <br /><sup>1</sup> = Existing RCG must have CPG and Copy CPG configured.
 <br />Link to [HPE Primera OS: Configuring data replication using Remote Copy](https://techhub.hpe.com/eginfolib/storage/docs/Primera/RemoteCopy/RCconfig/index.html#GUID-F6C62635-5398-48E5-B1A6-3C8D8806C0D8.html)
</small>

!!! important
    
    Remote Copy groups (RCG) created by the HPE CSI driver 2.1 and later have the **Auto synchronize** and **Auto recover** policies applied. <br /> To add or remove these policies from RCGs, modify the existing RCG using the SSMC or CLI with the following command: <br/ > <br /> **Add** <br /> `setrcopygroup pol auto_recover,auto_synchronize <group_name>` <br/ > **Remove** <br /> `setrcopygroup pol no_auto_recover,no_auto_synchronize <group_name>`


#### Add Non-Replicated Volume to Remote Copy group

To add a non-replicated volume to an existing Remote Copy group, `allowMutations: description` at minimum must be defined within the `StorageClass`. Refer to [Remote Copy with Peer Persistence Replication](#remote_copy_with_peer_persistence_synchronous_replication_parameters) for more details.

Edit the non-replicated PVC and annotate the following parameters:

| Parameter       | Option  | Description |
| --------------- | ------- | ----------- |
| remoteCopyGroup | Text    | Name of existing Remote Copy group. |
| oneRcgPerPvc    | Boolean | Creates a dedicated Remote Copy group per persistent volume. (Optional) |
| replicationDevices | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |

!!! note
    `remoteCopyGroup` and `oneRcgPerPvc` parameters are mutually exclusive and cannot be added together when editing a `PVC`.



## VolumeSnapshotClass Parameters

These parameters are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information. Volumes with snapshots are immutable.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter | String  | Description |
| --------- | ------  | ----------- |
| read_only | Boolean | Indicates if the snapshot is writable on the array. |

## VolumeGroupClass Parameters

In the HPE CSI Driver version 1.4.0+, a volume set with QoS settings can be created dynamically using the QoS parameters for the `VolumeGroupClass`. The following parameters are available for a `VolumeGroup` on the array. Learn more about `VolumeGroups` in the [provisioning concepts documentation](../../csi_driver/using.md#volume_groups).

| Parameter    | String | Description |
| ------------ | ------ | ----------- |
| description  | Text   | An identifier to describe the `VolumeGroupClass`. Example: "My VolumeGroupClass" |
| priority     | Text   | The priority level for the target volume set. Example: "low", "normal", "high"|
| ioMinGoal    | Text   | IOPS minimum goal for the target volume set. Example: "300" |
| ioMaxLimit   | Text   | IOPS maximum limit for the target volume set. Example: "10000" |
| bwMinGoalKb  | Text   | Bandwidth minimum goal in kilobytes per second for the target volume set. Example: "300" |
| bwMaxLimitKb | Text   | Bandwidth maximum limit in kilobytes per second for the target volume set. Example: "30000" |
| latencyGoal  | Text   | Latency goal in milliseconds (ms) or microseconds(us) for the target volume set. Example: "300ms" or "500us" |
| domain       | Text   | The array Virtual Domain, with which the volume group and related objects are associated with. Example: "sample_domain" |

!!! caution "Important"
    All QoS parameters are mandatory when creating a `VolumeGroupClass` on the array.

Example:

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
  priority: normal
  ioMinGoal: "300"
  ioMaxLimit: "10000"
  bwMinGoalKb: "3000"
  bwMaxLimitKb: "30000"
  latencyGoal: "300ms"
```


## SnapshotGroupClass Parameters

These parameters are for `SnapshotGroupClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information. Volumes with snapshots are immutable.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter | String  | Description |
| --------- | ------- | ----------- |
| read_only | Boolean | Indicates if the snapshot is writable on the array. |

## Static Provisioning

Static provisioning of `PVs` and `PVCs` may be used when absolute control over physical volumes are required by the storage administrator. This CSP also supports importing volumes and clones of volumes using the [import parameters](#import_parameters) in a `StorageClass`.

### Prerequisites

The CSP expects a certain naming convention for `PersistentVolumes` and Virtual Volumes on the array.

- Persistent Volume: `pvc-00000000-0000-0000-0000-000000000000`
- Virtual Volume: `pvc-00000000-0000-0000-0000-000`

!!! note
    The zeroes are used as examples. They can be replaced with any hexadecimal from `0` to `f`. Establishing a scheme may be important if static provisioning is going to be the main method of providing persistent storage to workloads.

The following example uses the above scheme as a naming convention. Have a storage administrator rename the existing Virtual Volume on the array:

```text
setvv -name pvc-00000000-0000-0000-0000-000 my-existing-virtual-volume
```

### HPEVolumeInfo

Create a new `HPEVolumeInfo` resource. 

```text
apiVersion: storage.hpe.com/v2
kind: HPEVolumeInfo
metadata:
  name: pvc-00000000-0000-0000-0000-000000000000
spec:
  record:
    Id: pvc-00000000-0000-0000-0000-000000000000
    Name: pvc-00000000-0000-0000-0000-000
  uuid: pvc-00000000-0000-0000-0000-000000000000
```

### Persistent Volume

Create a `PV` referencing the `HPEVolumeInfo` resource.

!!! warning
    If a filesystem can't be detected on the device a new filesystem will be created. If the volume contains data, make sure the data reside in a whole device filesystem.

```text
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvc-00000000-0000-0000-0000-000000000000
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 16Gi
  csi:
    volumeHandle: pvc-00000000-0000-0000-0000-000000000000
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
      storage: 16Gi
  volumeName: my-static-pv-1
  storageClassName: ""
```

## Support

Please refer to the HPE Alletra Storage MP, Alletra 9000 and Primera and 3PAR Storage CSP [support statement](../../legal/support/index.md#hpe_primera_and_hpe_3par_container_storage_provider_support).
