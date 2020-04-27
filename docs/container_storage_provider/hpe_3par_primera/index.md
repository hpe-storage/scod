# Introduction

The HPE 3PAR and Primera Container Storage Provider integrates as part of the [HPE CSI Driver for Kubernetes](../../csi_driver/index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes. 

[TOC]

## Platform requirements

Always check the corresponding CSI driver version in [compatibility and support](../../csi_driver/index.md#compatibility_and_support) and [SPOCK](#spock) for latest support matrix for the HPE 3PAR and Primera Container Storage Provider.

|  CSI   |   CSP  | Linux OS | OpenShift | Kubernetes | 3PAR and Primera OS |
| ------ | ------ | -------- | --------- | ---------- | ------------------- |
| v1.1.1 | v1.0.0 | - CentOS: 7.7 <br /> - RHEL: 7.6, 7.7 / RHCOS | OpenShift 4.2 with RHEL 7.6 or 7.7 or RHCOS as worker nodes| K8s 1.16, 1.17 | - 3PAR OS: 3.3.1 (FC & iSCSI) <br /> - Primera OS: 4.0.0, 4.1.0 (FC only) |

!!! important
    &emsp;- Minimum 2 iSCSI IP ports should be in ready state<br>
    &emsp;- FC array should be in ready state and zoned with initiator hosts<br>
    &emsp;- FC supported only on Bare metal and Fabric SAN

#### SPOCK
* [3PAR](https://spock.corp.int.hpe.com/SPOCK/Pages.internal/spock2Html.aspx?htmlFile=hw_3par.internal.html) 
* [Primera](https://spock.corp.int.hpe.com/SPOCK/Pages.internal/spock2Html.aspx?htmlFile=hw_primera.internal.html) 

!!! tip
    The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities.

#### User role requirements

The CSP requires access to a user with either `edit` or the `super` role. It's recommended to use the `edit` role for least privilege practices.

## StorageClass example

A `StorageClass` is used to provision an HPE 3PAR or Primera Storage-backed persistent volume. Please see [using the HPE CSI Driver](../../csi_driver/using.md#base_storageclass_parameters) for additional base `StorageClass` examples like CSI snapshots and clones. 

Here is an example of a `StorageClass` using the HPE 3PAR and Primera CSP. Please see [common parameters](#common_parameters_for_provisioning) for additional parameter options.

```yaml
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
| --------------------------------- | ------- | ----------- | :--: | :-----: |
| accessProtocol <br /> (required)    | fc      | The access protocol to use when accessing the persistent volume. | **X** | **X** |
|                                     | iscsi   | The access protocol to use when accessing the persistent volume. | **X** |   |
| cpg <br /> (required)               | Text    | The name of existing CPG to be used for volume provisioning. | **X** | **X** | 
| snap_cpg                            | Text    | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. | **X** | **X** |
| compression                         | Boolean | Indicates that the volume should be compressed. | **X** |   |
| provisioning_type <br /> (required) | tpvv    | Indicates Thin provisioned volume type. | **X** | **X** |
|                                     | full    | Indicates Full provisioned volume type. | **X** |   |
|                                     | dedup   | Indicates Thin Deduplication volume type. | **X** |   |
|                                     | reduce  | Indicates Thin Deduplication/Compression volume type. |   | **X** |

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
| accessProtocol                 | Text    | Storage access protocol to use, "iscsi" or "fc".

!!! important
    All parameters are **required** for inline ephemeral volumes.

## VolumeSnapshotClass parameters

These parametes are for `VolumeSnapshotClass` objects when using CSI snapshots. Please see [using CSI snapshots](../../csi_driver/using.md#using_csi_snapshots) for more details.

| Parameter   | String  | Description |
| ----------- | ------  | ----------- |
| read_only   | Boolean | Indicates if the snapshot is writable on the 3PAR or Primera array. |

## Support

Please refer to the HPE 3PAR and Primera Container Storage Provider [support statement](../../legal/support/index.md#hpe_3par_and_primera_container_storage_provider_support).