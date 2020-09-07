# Introduction

The HPE 3PAR and Primera Container Storage Provider integrates as part of the [HPE CSI Driver for Kubernetes](../../csi_driver/index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes. 

[TOC]

!!! note
    For help getting started with deploying the HPE CSI Driver using HPE Primera or 3PAR Storage, check out the [tutorial over at HPE DEV](https://developer.hpe.com/blog/9o7zJkqlX5cErkrzgopL/tutorial-how-to-get-started-with-the-hpe-csi-driver-and-hpe-primera-and-).

## Platform requirements

Always check the corresponding CSI driver version in [compatibility and support](../../csi_driver/index.md#compatibility_and_support) and [SPOCK](#spock) for latest support matrix for the HPE 3PAR and Primera Container Storage Provider.

|  CSI   | Linux OS | OpenShift | Kubernetes | 3PAR and Primera OS |
| ------ | -------- | --------- | ---------- | ------------------- |
| v1.3.0 | - CentOS: 7.7 <br /> - RHEL: 7.6, 7.7 / RHCOS | OpenShift 4.2/4.3 with RHEL 7.6 or 7.7 or RHCOS as worker nodes| K8s 1.16, 1.17, 1.18 | - 3PAR OS: 3.3.1 (FC & iSCSI) <br /> - Primera OS: 4.0.0, 4.1.0 (FC only), 4.2.0 |

!!! important
    • Minimum 2 iSCSI IP ports should be in ready state<br />
    • FC array should be in ready state and zoned with initiator hosts<br />
    • FC supported only on Bare metal and Fabric SAN

#### SPOCK
Refer to Hewlett Packard Enterprise Single Point of Connectivity Knowledge (SPOCK) for HPE storage products for specific platform details (requires a HPE Passport account).

* [3PAR](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_3par.html)
* [Primera](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_primera.html)

!!! tip
    The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities.

#### User role requirements

The CSP requires access to a user with either `edit` or the `super` role. It's recommended to use the `edit` role for least privilege practices.

## StorageClass example

A `StorageClass` is used to provision an HPE 3PAR or Primera Storage-backed persistent volume. Please see [using the HPE CSI Driver](../../csi_driver/using.md#base_storageclass_parameters) for additional base `StorageClass` examples like CSI snapshots and clones. 

Here is an example of a `StorageClass` using the HPE 3PAR and Primera CSP. Please see [common parameters](#common_parameters_for_provisioning) for additional parameter options.

```markdown
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: 3par-sc
provisioner: csi.hpe.com
allowVolumeExpansion: true
parameters:
  csi.storage.k8s.io/fstype: ext4
  csi.storage.k8s.io/provisioner-secret-name: primera3par-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: primera3par-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: primera3par-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: primera3par-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/resizer-secret-name: primera3par-secret
  csi.storage.k8s.io/resizer-secret-namespace: kube-system
  csi.storage.k8s.io/controller-expand-secret-name: primera3par-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
  cpg: FC_r6
  provisioning_type: tpvv
  accessProtocol: iscsi
```

## StorageClass parameters

All parameters enumerated reflects the current version and may contain unannounced features and capabilities. 

### Common parameters for provisioning

These parameters are used for volume provisioning and supported platforms.

| Parameter                         | Option  | Description | 3PAR | Primera |
| --------------------------------- | ------- | ----------- | ---- | ------- |
| accessProtocol <br /> (required)    | fc      | The access protocol to use when accessing the persistent volume. | **X** | **X** |
|                                     | iscsi   | The access protocol to use when accessing the persistent volume. | **X** |   |
| cpg <br />                | Text    | The name of existing CPG to be used for volume provisioning. If the cpg parameter is not specified, the CSP will automatically set cpg parameter based upon a CPG available to 3PAR or Primera array.| **X** | **X** | 
| snap_cpg                            | Text    | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. | **X** | **X** |
| compression                         | Boolean | Indicates that the volume should be compressed. | **X** |   |
| provisioning_type <br />  | tpvv    | Indicates Thin provisioned volume type. Default: tpvv | **X** | **X** |
|                                     | full    | Indicates Full provisioned volume type. | **X** |   |
|                                     | dedup   | Indicates Thin Deduplication volume type. | **X** |   |
|                                     | reduce  | Indicates Thin Deduplication/Compression volume type. |   | **X** |
| importVol   | Text      | Name of the volume to import. | **X** | **X** |
| importVolAsClone  | Text      | Name of the volume to clone and import. | **X** | **X** |
| cloneOf  | Text      | Name of the `PersistentVolumeClaim` to clone. | **X** | **X** |
| virtualCopyOf  | Text      | Name of the `PersistentVolumeClaim` to snapshot. | **X** | **X** |
| qosName  | Text      | Name of the volume set which has QoS rules applied. | **X** | **X** |
| remoteCopyGroup <br /> | Text | Name of a new or existing remote copy group on HPE Primera/3PAR array. | **X** | **X** |
| replicationDevices <br /> | Text <br /> | Indicates name of custom resource of type `hpereplicationdeviceinfos`. | **X** | **X** |
| iscsiPortalIps <br /> | Text <br /> | Comma separated list of HPE Primera/3PAR iSCSI port IPs. | **X** | **X** |

!!! Important
    The HPE CSI Driver allows the `PersistentVolumeClaim` to override the `StorageClass` parameters by annotating the `PersistentVolumeClaim`. Please see [Using PVC Overrides](../../csi_driver/using.md#using_pvc_overrides) for more details.

### Primera Data Reduction volumes

These parameters are used to create Primera Data Reduction (Thinly Provisioned with Deduplication/Compression enabled) volumes. Please see [Common Parameters](#common_parameters_for_provisioning) for more details.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| accessProtocol     | fc      | The access protocol to use when accessing the persistent volume. |
| cpg                | Text    | The name of existing CPG to be used for volume provisioning. |
| snap_cpg           | Text    | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. |
| provisioning_type  | reduce  | **Required** |

Example `StorageClass` for provisioning Primera Data Reduction volumes

```markdown
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: primera-reduce-sc
provisioner: csi.hpe.com
allowVolumeExpansion: true
parameters:
  csi.storage.k8s.io/fstype: ext4
  csi.storage.k8s.io/provisioner-secret-name: primera3par-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: primera3par-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: primera3par-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: primera3par-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/resizer-secret-name: primera3par-secret
  csi.storage.k8s.io/resizer-secret-namespace: kube-system
  csi.storage.k8s.io/controller-expand-secret-name: primera3par-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
  cpg: SSD_r6
  provisioning_type: reduce
  accessProtocol: fc
```  


### Pod inline volume parameters (Local Ephemeral Volumes)
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

### Import volume
During the import volume process, any legacy (non-container volumes) or existing docker/k8s volume defined in the **ImportVol** parameter, within a `StorageClass`, will be renamed to match the `PersistentVolumeClaim` that leverages the `StorageClass`. The new volumes will be exposed through the HPE CSI Driver and made available to the Kubernetes cluster. **Note:** All previous Access Control Records and Initiator Groups will be removed from the volume when it is imported.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| accessProtocol     | fc or iscsi  | The access protocol to use when accessing the persistent volume. |
| importVol          | Text    | The name of the HPE Primera or 3PAR volume to import. |

!!! important
    • **No other parameters** are required in the `StorageClass` when importing a volume outside of those parameters listed in the table above.<br />
    • Support for `importVol` is available from HPE CSI Driver 1.2.0.

### Cloning parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a `PersistentVolumeClaim` in the current namespace to clone or use `importVolAsClone` and reference a HPE Primera or 3PAR volume name to clone and import to Kubernetes.

| Parameter        | Option  | Description |
| ---------------- | ------- | ----------- |
| cloneOf          | Text    | The name of the `PersistentVolumeClaim` to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text    | The name of the HPE Primera or 3PAR volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| accessProtocol     | fc or iscsi  | The access protocol to use when accessing the persistent volume. |

!!! important
    • **No other parameters** are required in the `StorageClass` while cloning outside of those parameters listed in the table above.<br />
    • Cloning using above parameters is independent of snapshot `CRD` availability on Kubernetes and it can be performed on any supported Kubernetes version.<br />
    • Support for `importVolAsClone` and `cloneOf` is available from HPE CSI Driver 1.3.0.

### Snapshotting a volume
During snapshotting process, any existing `PersistentVolumeClaim` defined in the `virtualCopyOf` parameter within a `StorageClass`, will be snapped as `PersistentVolumeClaim` and exposed through the HPE CSI Driver and made available to the Kubernetes cluster.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| accessProtocol     | fc or iscsi  | The access protocol to use when accessing the persistent volume. |
| virtualCopyOf      | Text         | The name of existing `PersistentVolumeClaim` to be snapped |

!!! important
    • **No other parameters** are required in the `StorageClass` when snapshotting a volume outside of those parameters listed in the table above.<br />
    • Snapshotting using `virtualCopyOf` is independent of snapshot `CRD` availability on Kubernetes and it can be performed on any supported Kubernetes version.<br />
    • Support for `virtualCopyOf` is available from HPE CSI Driver 1.3.0.

### Applying QOS Rules (qosName)
In the HPE Primera or 3PAR Storage system, the QoS rules are applied to a volume set. To apply QoS rules to a `PersistentVolumeClaim`, create a volume set in the HPE Primera or 3PAR, apply QoS rules to the volume set and set the `qosName` parameter within a `StorageClass` to the name of the HPE Primera or 3PAR volume set.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| qosName      | Text         | Name of the HPE Primera or 3PAR volume set which has QoS rules. This parameter is optional. If specified, the `PersistentVolumeClaim` will be associated with the HPE Primera or 3PAR volume set, for purposes of applying the QoS rules. |

### Remote Copy with Peer Persistence synchronous replication

To enable replication within the HPE CSI Driver, the following steps must be completed:

* Create `Secrets` for both primary and target HPE Primera or 3PAR arrays. Refer to [Adding additional backends](../../csi_driver/deployment.md#adding_additional_backends).
* Create replication custom resource.
* Create replication enabled `StorageClass`.

For a tutorial on how to enable replication, check out the blog [Enabling Remote Copy using the HPE CSI Driver for Kubernetes on HPE Primera](https://developer.hpe.com/blog/ppPAlQ807Ah8QGMNl1YE/tutorial-enabling-remote-copy-using-the-hpe-csi-driver-for-kubernetes-on)

A Custom Resource Definition (CRD) of type `hpereplicationdeviceinfos.storage.hpe.com`  must be created to define the target array information. The CRD object name will used to define the `StorageClass` parameter **replicationDevices**. 
```yaml
apiVersion: storage.hpe.com/v1
kind: HPEReplicationDeviceInfo
metadata:
  name: r1
spec:
  target_array_details:
  - targetCpg: <cpg_name>
    targetSnapCpg: <snapcpg_name> #optional
    targetName: <target_array_name>
    targetSecret: <target_secret_name>
    targetSecretNamespace: kube-system
```

!!! important
    • targetCpg, targetName, targetSecret and targetSecretNamespace are mandatory for `HPEReplicationDeviceInfo` CRD.<br />
    • Currently, the HPE CSI Driver only supports Remote Copy Peer Persistence mode. Async support will be added in a future release.<br />

These parameters are applicable only for replication. Both parameters are mandatory. If the remote copy volume group (RCG) name, as defined within the `StorageClass`, does not exist on the HPE Primera or 3PAR array, then a new RCG will be created.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| remoteCopyGroup    | Text    | Name of new or existing remote copy group on the HPE Primera/3PAR array. |
| replicationDevices | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |

### Target Portal IPs

This parameter allows the ability to specify a subset of HPE Primera/3PAR iSCSI ports for iSCSI sessions. By default, the HPE CSI Driver uses all available iSCSI ports.

| Parameter      | Option  | Description |
| -------------- | ------- | ----------- |
| iscsiPortalIps |   Text  | Comma separated list of target portal IPs. |

## VolumeSnapshotClass parameters

These parameters are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../csi_driver/using.md#enabling_csi_snapshots) for more information.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots).

| Parameter   | String  | Description |
| ----------- | ------  | ----------- |
| read_only   | Boolean | Indicates if the snapshot is writable on the 3PAR or Primera array. |

### Import Snapshot
During the import snapshot process, any legacy (non-container snapshot) or an existing docker/k8s snapshot defined in the **ImportVol** parameter, within a `VolumeSnapshotClass`, will be renamed with the prefix "snapshot-". The new snapshot will be exposed through the HPE CSI Driver and made available to the Kubernetes cluster. **Note:** All previous Access Control Records and Initiator Groups will be removed from the snapshot when it is imported.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| importVol          | Text    | The name of the 3PAR or Primera snapshot to import. |

## Support

Please refer to the HPE 3PAR and Primera Container Storage Provider [support statement](../../legal/support/index.md#hpe_3par_and_primera_container_storage_provider_support).
