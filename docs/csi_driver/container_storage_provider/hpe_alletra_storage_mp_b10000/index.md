# Introduction

The HPE Alletra Storage MP B10000, Alletra 9000 and Primera and 3PAR Storage Container Storage Provider (CSP) for Kubernetes is part of the [HPE CSI Driver for Kubernetes](../../index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes.

!!! caution "Attention"
    The HPE Alletra Storage MP B10000 *File Service* has its own **[CSP](../hpe_alletra_storage_mp_b10000_file_service/index.md)** and does not share the same features and capabilities as the block protocol CSP on this page.

[TOC]

!!! note
    For help getting started with deploying the HPE CSI Driver using HPE Alletra Storage MP B10000, Alletra 9000, Primera or 3PAR storage, check out the [tutorial over at HPE Developer](https://developer.hpe.com/blog/9o7zJkqlX5cErkrzgopL/tutorial-how-to-get-started-with-the-hpe-csi-driver-and-hpe-primera-and-).

## Platform Requirements

Check the corresponding CSI driver version in the [compatibility and support](../../index.md#compatibility_and_support) table for the latest updates on supported Kubernetes version, orchestrators and host OS.
<!-- Refer to the HPE Single Point of Connectivity Knowledge (SPOCK) for specific platform details (requires an HPE Passport account) of the CSP. The documentation reflected here always corresponds to the latest supported version and may contain references to future features and capabilities.

* [HPE Alletra Storage MP B10000](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_greenlake_block.html)
* [HPE Alletra 9000](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_alletra.html)
* [HPE Primera](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_primera.html)
* [HPE 3PAR](https://h20272.www2.hpe.com/SPOCK/Pages/spock2Html.aspx?htmlFile=hw_3par.html) -->

### Network Port Requirements

The HPE Alletra Storage MP B10000, Alletra 9000, Primera and 3PAR Container Storage Provider requires the following TCP ports to be open inbound to the array from the Kubernetes cluster worker nodes running the HPE CSI Driver for Kubernetes.

| Port | Protocol | Description |
| ---- | -------- | ----------- |
| 443 | HTTPS | WSAPI (HPE Alletra Storage MP B10000, Alletra 9000 and Primera) |
| 8080 | HTTPS | WSAPI (HPE 3PAR) |
| 22 | SSH | Array communication (HPE 3PAR) |

!!! caution "HPE 3PAR"
    From HPE CSI Driver v2.5.2 onwards it's recommended to specify "&lt;IP Addr&gt;:443" in the backend `Secret` to avoid using SSH for any HPE Alletra Storage MP B10000 derived platform except 3PAR. See [Deployment](../../deployment.md#secret_parameters) for more information.

### User Role Requirements

The CSP requires access to a user with either `edit` or the `super` role. It's recommended to use the `edit` role for security best practices.

!!! note
    LDAP accounts may be used from HPE CSI Driver v2.5.2 onwards.

### Virtual Domains

Virtual Domains are not yet fully supported by the CSP. From HPE CSI Driver v2.5.0, it's possible to manually create the Kubernetes hosts connecting to storage within the Virtual Domain. Once the hosts have been created, deploy the CSI driver with the Helm chart using the "disableHostDeletion" parameter set to "true". The Virtual Domain user may create the hosts through the Virtual Domain if the "AllowDomainUsersAffectNoDomain" parameter is set to either "hostonly" or "yes" on the array.

#### Detailed steps to use Virtual Domains

These steps assumes access to the storage platform with privileges to create domains and change settings.

Login to the storage platform with SSH. Create an new domain:

```text
cli% createdomain -comment "This is a test domain." my-kubernetes-domain-0
```

Then, create a new user and assign to the domain. These credentials will be used by the CSI driver.

```text
cli% createuser -c my-password-0 domain-user-0 my-kubernetes-domain-0 edit
```

Next, make sure domain users are allowed to create hosts outside the domain.

```text
cli% setsys AllowDomainUsersAffectNoDomain hostonly
```

!!! tip
    Hosts can be created manually at any point. Make sure the name of the host matches the name of each of the compute (worker) nodes in the Kubernetes cluster.

The next steps involve installing the HPE CSI Driver for Kubernetes with `disableHostDeletion` set to `true`. The steps to supply the parameter depends on if the Helm chart or Operator is being used.

- Helm chart install from [ArtifactHub.io](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver).
- Operator install for [OpenShift](../../partners/redhat_openshift/index.md).

Once the CSI driver is installed and running, [add an HPE storage backend](../../deployment.md#add_an_hpe_storage_backend) with the credentials provided in the steps above.

!!! note
    Remote Copy Groups managed by the CSP have not been tested with Virtual Domains at this time.

### Limitations

These are the generally known limitation of the CSP.

- The CSP has been tested using iSCSI with up to 250 `VolumeAttachments` per compute node. HPE recommends not exceeding 200 `VolumeAttachments` per node and leave headroom for emergencies. It's always recommended to test the upper bounds before deploying to production. Increasing the "maxVolumesPerNode" parameter from the default of 100 is explained in the [Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver).
- Compute node hostnames may not exceed 27 characters. The storage platform limitation is 31 characters. Since HPE CSI Driver 3.0.0, the node name has a 4 character protocol prefix such as "iqn-" or "wwn-". Further, the CSP truncates the domain name from the Kubernetes node name.

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

Example default `StorageClass` ([download](examples/storageclass.yaml)):

```yaml
{% include "csi_driver/container_storage_provider/hpe_alletra_storage_mp_b10000/examples/storageclass.yaml" %}```

!!! hint
    If all mutable parameters have values provided during provisioning of the `PersistentVolumes`, the [Volume Mutator](../../using.md#using_volume_mutations) will later allow changes if needed.

### Common Provisioning Parameters

| Parameter  | &nbsp;&nbsp;Option&nbsp;&nbsp;  | Description |
| ---------- | ------- | ----------- |
| accessProtocol (**Required**)  | fc or iscsi | The access protocol to use when attaching the persistent volume. |
| cpg <sup>1</sup> | Text | The name of existing CPG to be used for volume provisioning. If the `cpg` parameter is not specified, the CSP will select a CPG available to the array. |
| snapCpg <sup>1</sup> | Text | The name of the snapshot CPG to be used for volume provisioning. Defaults to value of `cpg` if not specified. |
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
| iscsiPortalIps | Text | Comma separated list of the array iSCSI port IPs. |
| fcPortsList | Text   | Comma separated list of available FC ports. Example: "0:5:1,1:4:2,2:4:1,3:4:2" Default: Use all available ports. |

<small>
 Restrictions applicable when using the [CSI volume mutator](../../using.md#using_volume_mutations):
 <br /><sup>1</sup> = Parameters that are editable after provisioning.
 <br /><sup>2</sup> = Volumes with snapshots/clones can't be modified.
 <br /><sup>3</sup> = HPE 3PAR only parameter
 <br /><sup>4</sup> = HPE Primera/Alletra 9000 only parameter
</small>

Please see [using the HPE CSI Driver](../../using.md#base_storageclass_parameters) for additional `StorageClass` examples like CSI snapshots and clones.

!!! important
    The HPE CSI Driver allows the `PersistentVolumeClaim` to override the `StorageClass` parameters by annotating the `PersistentVolumeClaim`. Please see [Using PVC Overrides](../../using.md#using_pvc_overrides) for more details.

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

<a name="remote_copy_with_peer_persistence_synchronous_replication_parameters"></a>
### Peer Persistence Configuration

The HPE Alletra Storage MP B10000 CSP supports two modes of performing synchrounous replication between two arrays, Active Peer Persistence (APP) and Classic Peer Persistence (CPP) using Remote Copy Groups (RCGs).

| Mode | Supported Platforms | Description |
| ---- | ------------------- | ----------- |
| APP  | HPE&nbsp;Alletra&nbsp;Storage&nbsp;MP&nbsp;B10000 | Fully automated disaster recovery and workload failover with symmetric topology up to campus distance while requiring a third site for quorum. Restrictions apply, see [Active Peer Persistence Limitations](#active_peer_persistence_limitations) |
| CPP  | HPE Alletra 9000<br />HPE Primera<br />HPE 3PAR | Data path resillience only, no workload failover. See [Classic Peer Persistence Limitations](#classic_peer_persistence_limitations) |

To enable replication within the HPE CSI Driver, the following steps must be completed:

* Create `Secrets` for both primary and target array. Refer to [Configuring Additional Storage Backends](../../deployment.md#configuring_additional_storage_backends).
* Create a replication `HPEReplicationDeviceInfos` CRD.
* Create a replication `HPEReplicationMappings` CRD.
* Create a replication enabled `StorageClass`.

A `CustomResourceDefinition` (CRD) of type `hpereplicationdeviceinfos.storage.hpe.com` must be created to define the target array information. The `CRD` resource name will be used to define the `StorageClass` parameter "replicationDevices".

```yaml
apiVersion: storage.hpe.com/v2
kind: HPEReplicationDeviceInfo
metadata:
  name: my-peer-persistence-target
  namespace: hpe-storage
spec:
  target_array_details:
  - targetCpg: <CPG name>
    targetSnapCpg: <Snap CPG name> # optional
    targetName: <Target array name>
    targetSecret: <Target Secret name>
    targetSecretNamespace: <Target Secret Namespace>
```

!!! info
    The "targetCpg" and "targetSnapCpg" names might be difficult to find on newer systems. On those systems the default name is "SSD_r6", if multiple CPGs are present on the system, use `showcpg` in the CLI to list the CPGs. The "targetName" can be listed on the primary using `showrcopy targets`. The "targetSnapCpg" parameter is not applicable for HPE Alletra Storage MP B10000 and should be omitted.

A `CustomResourceDefinition` (CRD) of type `hpereplicationmappings.storage.hpe.com` must be created to define the replication relationship information.

```yaml
apiVersion: storage.hpe.com/v3
kind: HPEReplicationMapping
metadata:
  name: hpe-csi-<Primary IP address>-<Target IP address>
spec:
  primary_array_details:
    cpg: <Primary CPG name>
    name: <Primary IP address>:443
    secret: <Primary Secret name>
    secretNamespace: <Primary Secret Namespace>
  rcg_details:
    rcgnames:
    - <Existing RCG name>
  target_array_details:
    cpg: <Target CPG name>
    name: <Target IP address>:443
    secret: <Target Secret name>
    secretNamespace: <Target Secret Namespace>
```

!!! note
    The RCG does not have to exist at this point, but is needed prior to creating and annotating PVCs.

Next, review and perform the prerequisites for:

- [Active Peer Persistence](#active_peer_persistence_prerequisites)
- [Classic Peer Persistence](#classic_peer_persistence_prerequisites)

#### Active Peer Persistence Prerequisites

Explaining all the requirements for using Active Peer Persistence is beyond the scope of this document. Be understood with the [Active Peer Persistence limitations](#active_peer_persistence_limitations) with the HPE CSI Driver before proceeding.

When using Active Peer Persistence, the CSP can't be allowed to manage the hosts on the backend arrays. While it's capable of creating hosts, the host needs to be admitted to RCGs manually to ensure the correct parameters are applied (see [Manual Host and RCG Creation](#manual_host_and_rcg_creation)).

!!! important
    The rest of this section assumes familiarity with the [HPE Active Peer Persistence technical white paper](https://www.hpe.com/psnow/doc/a00115612enw) and the terminology used therein.

##### HPE CSI Driver installation parameters

To prevent the CSP from deleting unused hosts, the HPE CSI Driver needs to be installed with the Helm chart or Operator using the "disableHostDeletion" parameter set to "true".

Helm chart install example:

```text
helm install --create-namespace -n hpe-storage my-hpe-csi-driver --set disableHostDeletion=true hpe-storage/hpe-csi-driver
```

!!! tip
    When installing using the Operator, the parameter is part of the `HPECSIDriver` manifest.

##### Manual Host and RCG Creation

The CSP is at this time unable to add hosts with the correct proximity to RCGs. This is due to an API limitation, the limitation will be removed in a future version of the HPE CSI Driver.

Create the hosts (iSCSI example):

```text
createhost -iscsi iqn-my-compute-node-1 iqn.1994-05.com.redhat:my-compute-node-1
createhost -iscsi iqn-my-compute-node-2 iqn.1994-05.com.redhat:my-compute-node-2
createhost -iscsi iqn-my-compute-node-3 iqn.1994-05.com.redhat:my-compute-node-3
```

!!! important
    Since HPE CSI Driver 3.0.0 the initiator host name need to be prefixed with the protocol, i.e "iqn-" for iSCSI and "wwn-" for Fibre Channel.

Create the RCG and set the correct policy:

```text
creatercopygroup -usr_cpg SSD_r6 my-target-FC:SSD_r6 my-csi-rcg my-target-FC:sync
setrcopygroup pol active_active my-csi-rcg
```

Admit the hosts to the RCG:

```text
admitrcopyhost -proximity all my-csi-rcg iqn-my-compute-node-1
admitrcopyhost -proximity all my-csi-rcg iqn-my-compute-node-2
admitrcopyhost -proximity all my-csi-rcg iqn-my-compute-node-3
```

Verify that the hostset exist on both primary and target array:

```text
showhostset RH2_my-csi-rcg*
```

Example output:

```text fct_label="Primary"
 Id Name           Members
102 RH2_my-csi-rcg iqn-my-compute-node-1
                   iqn-my-compute-node-2
                   iqn-my-compute-node-3
----------------------------------------
  1 total          3
```

```text fct_label="Target"
 Id Name                   Members
374 RH2_my-csi-rcg.r123352 iqn-my-compute-node-1
                           iqn-my-compute-node-2
                           iqn-my-compute-node-3
----------------------------------------
  1 total                  3
```

It's highly desirable for an Active Peer Persistence protected workload to be rescheduled during a site outage. To allow Kubernetes to reschedule the workloads that ran on a partitioned host, the HPE CSI Driver Pod Monitor need to remove the `VolumeAttachment` from the partitioned host.

Learn how to label `Pods` to be monitored by the HPE CSI Driver:

- [HPE CSI Driver for Kubernetes Pod Monitor](../../monitor.md)

!!! hint
    This construct also applies to KubeVirt virtual machines, not just containers.

##### StorageClass Parameters for Active Peer Persistence

Due to an API limitation and the necessary manual steps to pre-create all the storage resources prior to creating PVCs, it's not currently recommended to add replication parameters directly in the `StorageClass`. Instead, PVCs are annotated with the necessary parameters, either during creation or afterwards.

This section will be updated in a future revision, meanwhile, [Add Non-Replicated Volume to Remote Copy Group](#add_non-replicated_volume_to_remote_copy_group).

#### Classic Peer Persistence Prerequisites

If using existing RCGs for replication, the RCGs need to have the correct sync mode policy applied, "path_management". Periodic sync mode is not supported at this time.

```text
setrcopygroup pol path_management my-csi-rcg
```

Be understood with the [limitations](#remote_copy_limitations) of Classic Peer Persistence integration with the HPE CSI Driver before proceeding.

!!! hint
    Learn about Remote Copy and Peer Persistence mode in the [HPE Alletra 9000: Getting started with data replication using Remote Copy and the CLI](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00001084en_us&page=index.html) document.

For a tutorial on how to enable Classic Peer Persistence, check out the blog [Enabling Remote Copy using the HPE CSI Driver for Kubernetes on HPE Primera](https://developer.hpe.com/blog/ppPAlQ807Ah8QGMNl1YE/tutorial-enabling-remote-copy-using-the-hpe-csi-driver-for-kubernetes-on)

##### StorageClass Parameters for Classic Peer Persistence

These `StorageClass` parameters are applicable only for replication, "remoteCopyGroup" and "replicationDevices" are mandatory. If the RCG defined in "remoteCopyGroup" doesn't exist on the array, then a new RCG will be created.

| Parameter         | Option  | Description |
| ----------------- | ------- | ----------- |
| remoteCopyGroup | Text    | Name of new or existing RCG<sup>1</sup> on the array. |
| replicationDevices | Text    | Indicates name of `hpereplicationdeviceinfos` Custom Resource Definition (CRD). |
| allowBatchReplicatedVolumeCreation | Boolean | Enable the batch processing of persistent volumes in 10 second intervals and add them to a single Remote Copy group. (Optional) <br /> During this process, the Remote Copy group is stopped and started once. |
| oneRcgPerPvc | Boolean | Creates a dedicated Remote Copy group per persistent volume. (Optional) |

<small><sup>1</sup> = Existing RCGs must have local CPG and target CPG configured along with the correct policy, "path_management".<br />
</small>

!!! important
    RCGs created by the HPE CSI Driver applies the correct policies according to the replication mode supported by the system. Any changes to those policies need to be changed manually by a storage administrator and is highly discouraged.

#### Add Non-Replicated Volume to Remote Copy Group

In order to add an existing PVC to a RCG, the `StorageClass` will utilize the [Volume Mutator](../../using.md#using_volume_mutations). [PVC overrides](../../using.md#using_pvc_overrides) may be used for creating PVCs with replication.

!!! important
    Using the either mutations or overrides is the recommended workflow for Active Peer Persistence.

In the "parameter" section of the `StorageClass`, add the following (example):

```text
...
paramters:
  allowOverrides: replicationPolicy,remoteCopyGroup,replicationDevices
  allowMutations: replicationPolicy,remoteCopyGroup,replicationDevices
...
```

!!! tip
    It's entirely possible to only use "allowOverrides" to prevent users to tamper with existing PVCs.

Description of the parameters.

| Parameter          | Option  | Description |
| ------------------ | ------- | ----------- |
| replicationPolicy  | Text    | Set to "active" for Active Peer Persistence and omit or change to empty string if using Classic Peer Persistence. |
| remoteCopyGroup    | Text    | Name of existing RCG. |
| replicationDevices | Text    | Name of `HPEReplicationDeviceInfo` `CRD`. |
| oneRcgPerPvc       | Boolean | Creates a dedicated RCG per `PVC`. (Optional) |

!!! note
    "remoteCopyGroup" and "oneRcgPerPvc" parameters are mutually exclusive and cannot be added together when editing a `PVC`.

Learn in the using section how to apply overrides and mutations in more detail, as an example for replication, this PVC, when either created or edited, add the underlying `PersistentVolume` to "my-csi-rcg" RCG using Active Peer Persistence using the "my-peer-persistence-target" `HPEReplicationDeviceInfo`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-replicated-pvc
  annotations:
    csi.hpe.com/replicationPolicy: active
    csi.hpe.com/remoteCopyGroup: my-csi-rcg
    csi.hpe.com/replicationDevices: my-peer-persistence-target
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 374Gi
  storageClassName: hpe-standard
```

The same "edit" operation may be performed non-interactive with `kubectl`:

```text
kubectl annotate pvc/my-replicated-pvc \
    csi.hpe.com/replicationPolicy=active \
    csi.hpe.com/remoteCopyGroup=my-csi-rcg \
    csi.hpe.com/replicationDevices=my-peer-persistence-target
```

The underlying `PersistentVolume` is now being replicated.

!!! tip "Verification"
    The "VolumeEditSuccessful" event will be visible on the PVC if the mutation is successful, `kubectl events --for pvc/my-replicated-pvc`.

## VolumeSnapshotClass Parameters

These parameters are for `VolumeSnapshotClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../using.md#enabling_csi_snapshots) for more information. Volumes with snapshots are immutable.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../using.md#using_csi_snapshots).

| Parameter | String  | Description |
| --------- | ------  | ----------- |
| read_only | Boolean | Indicates if the snapshot is writable on the array. |

## VolumeGroupClass Parameters

In the HPE CSI Driver version 1.4.0+, a volume set with QoS settings can be created dynamically using the QoS parameters for the `VolumeGroupClass`. The following parameters are available for a `VolumeGroup` on the array. Learn more about `VolumeGroups` in the [provisioning concepts documentation](../../using.md#volume_groups).

| Parameter    | String   | Description |
| ------------ | -------- | ----------- |
| description  | Text     | An identifier to describe the `VolumeGroupClass`. Example: "My VolumeGroupClass" |
| domain       | Text     | The array Virtual Domain, with which the volume group and related objects are associated with. Example: "sample_domain" |
| bwMaxLimitKb | Text     | Bandwidth maximum limit in kilobytes per second for the target volume set. Example: "30000" |
| priority<sup>1</sup>    | Text   | The priority level for the target volume set. Example: "low", "normal", "high"|
| ioMinGoal<sup>1</sup>   | Text   | IOPS minimum goal for the target volume set. Example: "300" |
| ioMaxLimit              | Text   | IOPS maximum limit for the target volume set. Example: "10000" |
| bwMinGoalKb<sup>1</sup> | Text   | Bandwidth minimum goal in kilobytes per second for the target volume set. Example: "300" |
| latencyGoal<sup>1</sup> | Text   | Latency goal in milliseconds (ms) or microseconds(us) for the target volume set. Example: "300ms" or "500us" |

<small><sup>1</sup> = Parameter is deprecated and have no effect on HPE Alletra Storage MP B10000 10.5 and later.</small>

!!! caution "Important"
    All QoS parameters supported by the platform are mandatory when creating a `VolumeGroupClass`.

Example for HPE Primera:

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

!!! important
    In certain situations where `VolumeGroups` are used, the `StorageClass` needs to have `parameters.fsCreateOptions: "-K"` set to workaround a data path issue. A symptom of this issue is prolonged staging of newly provisioned `PersistentVolumes`.

## SnapshotGroupClass Parameters

These parameters are for `SnapshotGroupClass` objects when using CSI snapshots. The external snapshotter needs to be deployed on the Kubernetes cluster and is usually performed by the Kubernetes vendor. Check [enabling CSI snapshots](../../using.md#enabling_csi_snapshots) for more information. Volumes with snapshots are immutable.

How to use `VolumeSnapshotClass` and `VolumeSnapshot` objects is elaborated on in [using CSI snapshots](../../using.md#using_csi_snapshots).

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

## Active Peer Persistence Limitations

These are the current limitations of Active Peer Persistence when used with the HPE CSI Driver.

- Active Peer Persistence was introduced in version 3.0.0 of the CSI driver. Prior versions won't work.
- Three sites are required to provide a redundant Kubernetes control-plane and Quorum Witness for the RCGs.
- RCGs, volumes and VLUNs may not be managed outside of the CSI driver.
- Pods (containers or VMs) needs to be labeled accordingly to be monitored by the HPE CSI Driver [Pod Monitor](../../monitor.md) to prevent multi-attach errors.
- Only intra-cluster disaster recovery is supported.
- Only symmetric host proximity is supported and running Active Peer Persistence beyond a campus distance (around 1km) is not recommended.
- Only manual host and RCG creation is supported (this limitation will be removed in the future).
- Once an automatic failover has occurred, the recovered workloads needs to be manually restarted when the previous primary is restored. This will resume full redundancy with VLUNs created on both arrays for the workloads (a future platform update will address this workaround).
- It is not possible to clone volumes protected with Active Peer Persistence using the HPE CSI Driver.

<a name="remote_copy_limitations"></a>
## Classic Peer Persistence Limitations

These are the current limitations of the Remote Copy Classic Peer Persistence integration with the HPE CSI Driver.

- Classic Peer Persistence does not provide disaster recovery for workloads running on Kubernetes. Classic Peer Persistence provide disaster recovery for the storage system.
- Classic Peer Persistence only provide data path resilience. If the primary array is unreachable for the CSP or the role of the remote copy group has changed due to disaster recovery operations (manual or automatic switchover/failover), all CSI operations will cease to function until the primary array comes back up and the role of the remote copy groups returned to original state.
- When the primary array is unavailable for the Kubernetes cluster and remote copy group has failed over to the target array successfully, running workloads will continue to run if the host the workload was running on has redundant data paths to the target array (current primary array).
- It's possible to access volumes from the target array by [statically provisioning](#static_provisioning) `PersistentVolumes` without renaming the volume on the array. This is only safe if it has been determined that the primary array does not have active hosts accessing the volume against the primary array.

## Support

Please refer to the HPE Alletra Storage MP B10000, Alletra 9000 and Primera and 3PAR Storage CSP [support statement](../../../legal/support/index.md#hpe_alletra_storage_mp_b10000_alletra_9000_and_primera_and_3par_container_storage_provider_support).
