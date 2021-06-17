# Introduction

The HPE Alletra 9000 and Primera and 3PAR Storage Container Storage Provider (CSP) for Kubernetes is part of the [HPE CSI Driver for Kubernetes](../../csi_driver/index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes. 

[TOC]

!!! note
    For help getting started with deploying the HPE CSI Driver using HPE Alletra 9000, Primera or 3PAR storage, check out the [tutorial over at HPE DEV](https://developer.hpe.com/blog/9o7zJkqlX5cErkrzgopL/tutorial-how-to-get-started-with-the-hpe-csi-driver-and-hpe-primera-and-).

## Platform Requirements

The following has been tested and validated for HPE CSI driver version with HPE Alletra 9000, HPE Primera and 3PAR. Always check the corresponding CSI driver version in the [compatibility and support](../../csi_driver/index.md#compatibility_and_support) table.

| Version | Protocols | Host OS | Container Orchestrator | HPE Alletra 9000, Primera and 3PAR OS |
| ------ | ------------------- |-------- | --------- | ------------------- |
| v2.0.0 | iSCSI & FC | CentOS 8.x <br /> RHEL 8.x <br /> CoreOS | Kubernetes 1.18-1.21 <br /> Red Hat OpenShift 4.4, 4.6 <br /> SUSE CaaSP 4.2 | 3PAR OS 3.3.1+ <br /> Primera OS 4.0+ |
| v1.4.0 | iSCSI & FC | CentOS 8.1 <br /> RHEL 8.1 <br /> CoreOS | Kubernetes 1.17-1.20 <br /> Red Hat OpenShift 4.4, 4.6 <br /> SUSE CaaSP 4.2 | 3PAR OS 3.3.1+ <br /> Primera OS 4.0+ |
| v1.3.0 | iSCSI & FC | CentOS 7.6, 7.7 <br /> RHEL 7.6, 7.7 <br /> CoreOS | Kubernetes 1.16-1.19 <br /> Red Hat OpenShift 4.2, 4.3 | 3PAR OS 3.3.1+ <br /> Primera OS 4.0+ |
| v1.2.0 | iSCSI & FC | CentOS 7.6, 7.7 <br /> RHEL 7.6, 7.7 <br /> CoreOS | Kubernetes 1.16-1.18 <br /> Red Hat OpenShift 4.2, 4.3 | 3PAR OS 3.3.1+ <br /> Primera OS 4.0, 4.1 |

Refer to the HPE Single Point of Connectivity Knowledge (SPOCK) for specific platform details (requires an HPE Passport account) of the CSP. The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities. 

* [HPE Alletra 9000](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_alletra.html)
* [HPE Primera](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_primera.html)
* [HPE 3PAR](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_3par.html)

!!! important
    Ubuntu is not a supported OS option for the HPE CSI Driver when using HPE Alletra 9000 and HPE Primera. Please refer to SPOCK for the list of Platform supported OS.

### User Role Requirements

The CSP requires access to a user with either `edit` or the `super` role. It's recommended to use the `edit` role for security best practices.

## StorageClass Parameters

All parameters enumerated reflects the current version and may contain unannounced features and capabilities. 

### Common Provisioning Parameters

| Parameter                         | Option  | Description | 3PAR | HPE Alletra 9000 and Primera |
| --------------------------------- | ------- | ----------- | ---- | ------- |
| accessProtocol <br /> (required)    | fc      | The access protocol to use when accessing the persistent volume. | **X** | **X** |
|                                     | iscsi   | The access protocol to use when accessing the persistent volume. | **X** | **X** |
| cpg<sup>1</sup> <br />                | Text    | The name of existing CPG to be used for volume provisioning. If the cpg parameter is not specified, the CSP will automatically set cpg parameter based upon a CPG available to the array.| **X** | **X** | 
| snapCpg<sup>1</sup>                            | Text    | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. | **X** | **X** |
| compression<sup>1</sup>                         | Boolean | Indicates that the volume should be compressed. | **X** |   |
| provisioningType<sup>1</sup> <br />  | tpvv    | Indicates Thin provisioned volume type. Default: tpvv | **X** | **X** |
|                                     | full    | Indicates Full provisioned volume type. | **X** |   |
|                                     | dedup   | Indicates Thin Deduplication volume type. | **X** |   |
|                                     | reduce  | Indicates Data Reduction volume type. |   | **X** |
| importVol   | Text      | Name of the volume to import. | **X** | **X** |
| importVolAsClone  | Text      | Name of the volume to clone and import. | **X** | **X** |
| cloneOf<sup>2</sup>  | Text      | Name of the `PersistentVolumeClaim` to clone. | **X** | **X** |
| virtualCopyOf<sup>2</sup>  | Text      | Name of the `PersistentVolumeClaim` to snapshot. | **X** | **X** |
| qosName  | Text      | Name of the volume set which has QoS rules applied. | **X** | **X** |
| remoteCopyGroup <br /> | Text | Name of a new or existing remote copy group on the array. | **X** | **X** |
| replicationDevices <br /> | Text <br /> | Indicates name of custom resource of type `hpereplicationdeviceinfos`. | **X** | **X** |
| allowBatchReplicatedVolumeCreation <br /> | Boolean <br />  | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single remote copy group. <br /> During this process, the remote copy group is stopped and started once. | **X** | **X** |
| oneRcgPerPvc <br /> | Boolean <br /> | Creates a dedicated Remote Copy Group per persistent volume. | **X** | **X** |
| iscsiPortalIps <br /> | Text <br /> | Comma separated list of the array iSCSI port IPs. | **X** | **X** |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../csi_driver/using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameters that are editable after provisioning.
 <br /><sup>2</sup> = Volumes with snapshots/clones can't be modified.
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

### Importing Volumes

During the import volume process, any legacy (non-container volumes) or existing docker/k8s volume defined in the **ImportVol** parameter, within a `StorageClass`, will be renamed to match the `PersistentVolumeClaim` that leverages the `StorageClass`. The new volumes will be exposed through the HPE CSI Driver and made available to the Kubernetes cluster. **Note:** All previous Access Control Records and Initiator Groups will be removed from the volume when it is imported.

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

!!! important
    Due to a limitation within the HPE Primera/3PAR WSAPI, the **auto-synchronize** policy for replicated volumes is not available. Due to this, any automatically create remote copy groups (RCG) through the HPE CSI Driver will require a manual recovery and restore once both HPE Primera or 3PAR systems are online and both Remote Copy links are **up**. This will be addressed in an upcoming release of the HPE Primera and 3PAR WSAPI.
    <br /><br />
    For more information on **Auto synchronize**: <br />
    • "Remote Copy group policies" in [HPE Primera OS: Configuring data replicationusing Remote Copy over IP](https://support.hpe.com/hpesc/public/docDisplay?docLocale=en_US&docId=emr_na-a00088914en_us).<br />
    • "How Remote Copy recovers synchronous Remote Copy groups following replication link failure" in [Disaster-Tolerant Solutions with HPE 3PAR Remote Copy](https://h20195.www2.hpe.com/v2/GetPDF.aspx/4AA3-8318ENW.pdf).<br />
    <br /><br />**Workaround:**<br /> It is recommended to **pre-create** the remote copy group with "Auto Synchronize" and "Auto Recover" enabled using either the SSMC or with the CLI using: `setrcopygroup pol auto_failover,auto_synchronize <group_name>`. Set the `remoteCopyGroup` parameter to the existing remote copy group name.

For a tutorial on how to enable replication, check out the blog [Enabling Remote Copy using the HPE CSI Driver for Kubernetes on HPE Primera](https://developer.hpe.com/blog/ppPAlQ807Ah8QGMNl1YE/tutorial-enabling-remote-copy-using-the-hpe-csi-driver-for-kubernetes-on)

A Custom Resource Definition (CRD) of type `hpereplicationdeviceinfos.storage.hpe.com`  must be created to define the target array information. The CRD object name will be used to define the `StorageClass` parameter **replicationDevices**. 
```yaml
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
    • targetCpg, targetName, targetSecret and targetSecretNamespace are mandatory for `HPEReplicationDeviceInfo` CRD.<br />
    • Currently, the HPE CSI Driver only supports Remote Copy Peer Persistence mode. Async support will be added in a future release.<br />

These parameters are applicable only for replication. Both parameters are mandatory. If the remote copy volume group (RCG) name, as defined within the `StorageClass`, does not exist on the array, then a new RCG will be created.

| Parameter          | Option  | Description |
| ---------------------------------- | ------- | ----------- |
| remoteCopyGroup                    | Text    | Name of new or existing remote copy group on the array. |
| replicationDevices                 | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |
| allowBatchReplicatedVolumeCreation | Boolean | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single remote copy group. (Optional) <br /> During this process, the remote copy group is stopped and started once. |
| oneRcgPerPvc                       | Boolean | Creates a dedicated Remote Copy Group per persistent volume. (Optional) |

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
