!!! error "Expired content"
    The documentation described on this page may be obsolete and contain references to unsupported and deprecated software. Please reach out to your HPE representative if you think you need any of the components referenced within.

# Overview
The HPE Volume Driver for Kubernetes FlexVolume Plugin leverages HPE Nimble Storage or HPE Cloud Volumes to provide scalable and persistent storage for stateful applications.

!!! important
    Using HPE Nimble Storage with Kubernetes 1.13 and newer, please use the [HPE CSI Driver for Kubernetes](https://github.com/hpe-storage/csi-driver).

Source code and developer documentation is available in the [hpe-storage/flexvolume-driver](https://github.com/hpe-storage/flexvolume-driver) GitHub repo.

[TOC]

## Platform requirements
The FlexVolume driver supports multiple backends that are based on a "container provider" architecture. Currently, Nimble and Cloud Volumes are supported.

### HPE Nimble Storage Platform Requirements
| Driver      | HPE Nimble Storage Version | Release Notes    | Blog | 
|-------------|----------------------------|------------------|--------------|
| v3.0.0      | 5.0.8.x and 5.1.3.x onwards          | [v3.0.0](https://github.com/hpe-storage/flexvolume-driver/blob/master/release-notes/v3.0.0.md)| [HPE Storage Tech Insiders](https://community.hpe.com/t5/HPE-Storage-Tech-Insiders/Released-HPE-Volume-Driver-for-Kubernetes-FlexVolume-Plugin-3-0/ba-p/7063875) |
| v3.1.0      | 5.0.8.x and 5.1.3.x onwards          | [v3.1.0](https://github.com/hpe-storage/flexvolume-driver/blob/master/release-notes/v3.1.0.md)| 

* OpenShift Container Platform 3.9, 3.10 and 3.11.
* Kubernetes 1.10 and above.
* Redhat/CentOS 7.5+
* Ubuntu 16.04/18.04 LTS

**Note:** Synchronous replication (Peer Persistence) is not supported by the HPE Volume Driver for Kubernetes FlexVolume Plugin.

### HPE Cloud Volumes Platform Requirements

| Driver      | Release Notes    | Blog | 
|-------------|------------------|------|
| v3.1.0      | [v3.1.0](https://github.com/hpe-storage/flexvolume-driver/blob/master/release-notes/v3.1.0.md)| [Using HPE Cloud Volumes with Amazon EKS](https://developer.hpe.com/blog/using-hpe-cloud-volumes-with-amazon-eks) |

* Amazon EKS 1.12/1.13
* Microsoft Azure AKS 1.12/1.13
* US regions only

!!! important
    HPE Cloud Volumes was introduced in HPE CSI Driver for Kubernetes v1.5.0. Make sure to check if your cloud is supported by the [CSI driver](../../csi_driver) first.

## Deploying to Kubernetes
The recommended way to deploy and manage the HPE Volume Driver for Kubernetes FlexVolume Plugin is to use Helm. Please see the [co-deployments](https://github.com/hpe-storage/co-deployments) repository for further information.

Use the following steps for a manual installation.

### Step 1: Create a secret

#### HPE Nimble Storage
Replace the `password` string (`YWRtaW4=`) below with a base64 encoded version of your password and replace the `backend` with your array IP address and save it as `hpe-secret.yaml`.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: hpe-secret
  namespace: kube-system
stringData:
  backend: 192.168.1.1
  username: admin
  protocol: "iscsi"
data:
  # echo -n "admin" | base64
  password: YWRtaW4=
```

#### HPE Cloud Volumes
Replace the `username` and `password` strings (`YWRtaW4=`) with a base64 encoded version of your HPE Cloud Volumes "access_key" and "access_secret". Also, replace the `backend` with HPE Cloud Volumes portal fully qualified domain name (FQDN) and save it as `hpe-secret.yaml`.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: hpe-secret
  namespace: kube-system
stringData:
  backend: cloudvolumes.hpe.com
  protocol: "iscsi"
  serviceName: cv-cp-svc
  servicePort: "8080"
data:
  # echo -n "<my very confidential access key>" | base64
  username: YWRtaW4=
  # echo -n "<my very confidential secret key>" | base64
  password: YWRtaW4=
```

Create the secret:

```yaml
kubectl create -f hpe-secret.yaml
secret "hpe-secret" created
```

You should now see the HPE secret in the `kube-system` namespace.

```shell
kubectl get secret/hpe-secret -n kube-system
NAME                  TYPE                                  DATA      AGE
hpe-secret            Opaque                                5         3s
```

### Step 2. Create a ConfigMap
The `ConfigMap` is used to set and tweak defaults for both the FlexVolume driver and Dynamic Provisioner.

#### HPE Nimble Storage
Edit the below default parameters as required for FlexVolume driver and save it as `hpe-config.yaml`.

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: hpe-config
  namespace: kube-system
data:
  volume-driver.json: |-
    {
      "global":   {},
      "defaults": {
                 "limitIOPS":"-1",
                 "limitMBPS":"-1",
                 "perfPolicy": "Other"
                },
      "overrides":{}
    }
```

!!! tip
    Please see [Advanced](#advanced_configuration) for more `volume-driver.json` configuration options.

#### HPE Cloud Volumes
Edit the below parameters as required with your public cloud info and save it as `hpe-config.yaml`.

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: hpe-config
  namespace: kube-system
data:
  volume-driver.json: |-
    {
      "global": {
                "snapPrefix": "BaseFor",
                "initiators": ["eth0"],
                "automatedConnection": true,
                "existingCloudSubnet": "10.1.0.0/24",
                "region": "us-east-1",
                "privateCloud": "vpc-data",
                "cloudComputeProvider": "Amazon AWS"
      },
      "defaults": {
                "limitIOPS": 1000,
                "fsOwner": "0:0",
                "fsMode": "600",
                "description": "Volume provisioned by the HPE Volume Driver for Kubernetes FlexVolume Plugin",
                "perfPolicy": "Other",
                "protectionTemplate": "twicedaily:4",
                "encryption": true,
                "volumeType": "PF",
                "destroyOnRm": true
      },
      "overrides": {
      }
    }
```

Create the `ConfigMap`:

```shell
kubectl create -f hpe-config.yaml
configmap/hpe-config created
```

### Step 3. Deploy the FlexVolume driver and dynamic provisioner
Deploy the driver as a `DaemonSet` and the dynamic provisioner as a `Deployment`.

#### HPE Nimble Storage
Version 3.0.0:
```
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/flexvolume-driver/hpe-nimble-storage/hpe-flexvolume-driver-v3.0.0.yaml
```
Version 3.1.0:
```
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/flexvolume-driver/hpe-nimble-storage/hpe-flexvolume-driver-v3.1.0.yaml
```

#### HPE Cloud Volumes

Container-Provider Service:
```shell
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/flexvolume-driver/hpe-cloud-volumes/hpecv-cp-v3.1.0.yaml
```

The FlexVolume driver have different declarations depending on the Kubernetes distribution.

Amazon EKS:

```shell
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/flexvolume-driver/hpe-cloud-volumes/hpecv-aws-flexvolume-driver-v3.1.0.yaml
```

Microsoft Azure AKS:

```shell
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/flexvolume-driver/hpe-cloud-volumes/hpecv-azure-flexvolume-driver-v3.1.0.yaml
```
Generic:

```shell
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/flexvolume-driver/hpe-cloud-volumes/hpecv-flexvolume-driver-v3.1.0.yaml
```

!!! note
    The declarations for HPE Volume Driver for Kubernetes FlexVolume Plugin can be found in the [co-deployments](https://github.com/hpe-storage/co-deployments/tree/master/yaml/flexvolume-driver) repository.

Check to see all `hpe-flexvolume-driver` `Pods` (one per compute node) and the `hpe-dynamic-provisioner` Pod are running.

```shell
kubectl get pods -n kube-system
NAME                                            READY   STATUS    RESTARTS   AGE
hpe-flexvolume-driver-2rdt4                     1/1     Running   0          45s
hpe-flexvolume-driver-md562                     1/1     Running   0          44s
hpe-flexvolume-driver-x4k96                     1/1     Running   0          44s
hpe-dynamic-provisioner-59f9d495d4-hxh29        1/1     Running   0          24s
```

For HPE Cloud Volumes, check that `hpe-cv-cp` pod is running as well.
```shell
kubectl get pods -n kube-system -l=app=cv-cp
NAME                                READY   STATUS    RESTARTS   AGE
hpe-cv-cp-2rdt4                     1/1     Running   0          45s
```

## Using
Get started using the FlexVolume driver by setting up `StorageClass`, `PVC` API objects. See [Using](#using) for examples.

These instructions are provided as an example on how to use the HPE Volume Driver for Kubernetes FlexVolume Plugin with a HPE Nimble Storage Array.

The below YAML declarations are meant to be created with `kubectl create`. Either copy the content to a file on the host where `kubectl` is being executed, or copy & paste into the terminal, like this:

```shell
kubectl create -f-
< paste the YAML >
^D (CTRL + D)
```

!!! tip
    Some of the examples supported by the HPE Volume Driver for Kubernetes FlexVolume Plugin are available for [HPE Nimble Storage](https://github.com/hpe-storage/flexvolume-driver/tree/master/examples/kubernetes/hpe-nimble-storage) or [HPE Cloud Volumes](https://github.com/hpe-storage/flexvolume-driver/tree/master/examples/kubernetes/hpe-cloud-volumes) in the GitHub repo.

To get started, create a `StorageClass` API object referencing the `hpe-secret` and defining additional (optional) `StorageClass` parameters:

### Sample StorageClass

Sample storage classes can be found for [HPE Nimble Storage](https://github.com/hpe-storage/flexvolume-driver/tree/master/examples/kubernetes/hpe-nimble-storage/sc-nimble.yaml) and [HPE Cloud Volumes](https://github.com/hpe-storage/flexvolume-driver/tree/master/examples/kubernetes/hpe-cloud-volumes/sc-cv.yaml).

!!! hint
    See `StorageClass` parameters for [HPE Nimble Storage](#hpe_nimble_storage_storageclass_parameters) and [HPE Clound Volumes](#hpe_cloud_volumes_storageclass_parameters) for a comprehensive overview.

### Test and verify volume provisioning

Create a `StorageClass` with volume parameters as required.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sc-nimble
provisioner: hpe.com/nimble
parameters:
  description: "Volume from HPE FlexVolume driver"
  perfPolicy: "Other Workloads"
  limitIOPS: "76800"
```

Create a `PersistentVolumeClaim`. This makes sure a volume is created and provisioned on your behalf:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nimble
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: sc-nimble
```

Check that a new `PersistentVolume` is created based on your claim:

```shell
kubectl get pv
NAME                                            CAPACITY     ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
sc-nimble-13336da3-7ca3-11e9-826c-00505693581f  10Gi         RWO            Delete           Bound    default/pvc-nimble  sc-nimble               3s
```

The above output means that the FlexVolume driver successfully provisioned a new volume and bound to the requesting `PVC` to a new `PV`. The volume is not attached to any node yet. It will only be attached to a node if a workload is scheduled to a specific node. Now let us create a `Pod` that refers to the above volume. When the `Pod` is created, the volume will be attached, formatted and mounted to the specified container:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-nimble
spec:
  containers:
    - name: pod-nimble-con-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
    - name: pod-nimble-cont-2
      image: debian
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
  volumes:
    - name: export1
      persistentVolumeClaim:
        claimName: pvc-nimble
```

Check if the pod is running successfully:

```shell
kubectl get pod pod-nimble
NAME         READY   STATUS    RESTARTS   AGE
pod-nimble   2/2     Running   0          2m29s
```

### Use case specific examples

This `StorageClass` examples help guide combinations of options when provisioning volumes.

#### Data protection

This `StorageClass` creates thinly provisioned volumes with deduplication turned on. It will also apply the Performance Policy "SQL Server" along with a Protection Template. The Protection Template needs to be defined on the array.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
    name: oltp-prod
provisioner: hpe.com/nimble
parameters:
    thick: "false"
    dedupe: "true"
    perfPolicy: "SQL Server"
    protectionTemplate: "Retain-48Hourly-30Daily-52Weekly"
```

#### Clone and throttle for devs

This `StorageClass` will create clones of a "production" volume and throttle the performance of each clone to 1000 IOPS. When the PVC is deleted, it will be permanently deleted from the backend array.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: oltp-dev-clone-of-prod
provisioner: hpe.com/nimble
parameters:
  limitIOPS: "1000"
  cloneOf: "oltp-prod-1adee106-110b-11e8-ac84-00505696c45f"
  destroyOnRm: "true"
```

#### Clone a non-containerized volume

This `StorageClass` will clone a standard backend volume (without container metadata on it) from a particular pool on the backend.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: import-clone-legacy-prod
rovisioner: hpe.com/nimble
parameters:
  pool: "flash"
  importVolAsClone: "production-db-vol"
  destroyOnRm: "true"
```

#### Import (cutover) a volume
This `StorageClass` will import an existing Nimble volume to Kubernetes. The source volume needs to be offline for the import to succeed.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: import-clone-legacy-prod
provisioner: hpe.com/nimble
parameters:
  pool: "flash"
  importVol: "production-db-vol"
```

#### Using overrides
The HPE Dynamic Provisioner for Kubernetes understands a set of annotation keys a user can set on a `PVC`. If the corresponding keys exists in the list of the `allowOverrides` key in the `StorageClass`, the end-user can tweak certain aspects of the provisioning workflow. This opens up for very advanced data services.

StorageClass object:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-sc
provisioner: hpe.com/nimble
parameters:
  description: "Volume provisioned by StorageClass my-sc"
  dedupe: "false"
  destroyOnRm: "true"
  perfPolicy: "Windows File Server"
  folder: "myfolder"
  allowOverrides: snapshot,limitIOPS,perfPolicy
```

PersistentVolumeClaim object:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: my-pvc
 annotations:
    hpe.com/description: "This is my custom description"
    hpe.com/limitIOPS: "8000"
    hpe.com/perfPolicy: "SQL Server"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: my-sc
```

This will create a `PV` of 8000 IOPS with the Performance Policy of "SQL Server" and a custom volume description.

### Creating clones of PVCs
Using a `StorageClass` to clone a `PV` is practical when there's needs to clone across namespaces (for example from prod to test or stage). If a user wants to clone any arbitrary volume, it becomes a bit tedious to create a `StorageClass` for each clone. The annotation `hpe.com/CloneOfPVC` allows a user to clone any `PVC` within a namespace.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: my-pvc-clone
 annotations:
    hpe.com/cloneOfPVC: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: my-sc
```

## StorageClass parameters
This section highlights all the available `StorageClass` parameters that are supported.

### HPE Nimble Storage StorageClass parameters

A `StorageClass` is used to provision or clone an HPE Nimble Storage-backed persistent volume. It can also be used to import an existing HPE Nimble Storage volume or clone of a snapshot into the Kubernetes cluster. The parameters are grouped below by those same workflows.

A sample [StorageClass](https://raw.githubusercontent.com/hpe-storage/flexvolume-driver/master/examples/kubernetes/hpe-nimble-storage/sc-nimble.yaml) is provided.

!!! note
    These are optional parameters.

#### Common parameters for Provisioning and Cloning

These parameters are mutable betweeen a parent volume and creating a clone from a snapshot.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| nameSuffix| Text   | Suffix to append to Nimble volumes. Defaults to .docker |
| destroyOnRm | Boolean | Indicates the backing Nimble volume (including snapshots) should be destroyed when the PVC is deleted. |
| limitIOPS | Integer | The IOPS limit of the volume. The IOPS limit should be in the range 256 to 4294967294, or -1 for unlimited (default). |
| limitMBPS | Integer | The MB/s throughput limit for the volume. |
| description | Text | Text to be added to the volume's description on the Nimble array. |
| perfPolicy | Text | The name of the performance policy to assign to the volume. Default example performance policies include "Backup Repository", "Exchange 2003 data store", "Exchange 2007 data store", "Exchange 2010 data store", "Exchange log", "Oracle OLTP", "Other Workloads", "SharePoint", "SQL Server", "SQL Server 2012", "SQL Server Logs". |
| protectionTemplate | Text | The name of the protection template to assign to the volume. Default examples of protection templates include "Retain-30Daily", "Retain-48Hourly-30aily-52Weekly", and "Retain-90Daily". |
| folder | Text | The name of the Nimble folder in which to place the volume. |
| thick | Boolean | Indicates that the volume should be thick provisioned. |
| dedupeEnabled | Boolean | Indicates that the volume should enable deduplication. |
| syncOnUnmount | Boolean | Indicates that a snapshot of the volume should be synced to the replication partner each time it is detached from a node. |

!!! note
    Performance Policies, Folders and Protection Templates are Nimble specific constructs that can be created on the Nimble array itself to address particular requirements or workloads. Please consult with the storage admin or read the admin guide found on [HPE InfoSight](https://infosight.hpe.com).

#### Provisioning parameters

These parameters are immutable for clones once a volume has been created.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| fsOwner | userId:groupId | The user id and group id that should own the root directory of the filesystem. |
| fsMode | Octal digits | 1 to 4 octal digits that represent the file mode to be applied to the root directory of the filesystem. |
| encryption | Boolean | Indicates that the volume should be encrypted. |
| pool | Text | The name of the pool in which to place the volume. |

#### Cloning parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a PVC in the current namespace or use `importVolAsClone` and reference a Nimble volume name to clone and import to Kubernetes.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| cloneOf | Text | The name of the PV to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text | The name of the Nimble volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| snapshot | Text | The name of the snapshot to base the clone on. This is optional. If not specified, a new snapshot is created. |
| createSnapshot | Boolean | Indicates that a new snapshot of the volume should be taken matching the name provided in the `snapshot` parameter. If the `snapshot` parameter is not specified, a default name will be created. |
| snapshotPrefix | Text | A prefix to add to the beginning of the snapshot name. |

#### Import parameters

Importing volumes to Kubernetes requires the source Nimble volume to be offline. All previous Access Control Records and Initiator Groups will be stripped from the volume when put under control of the HPE Volume Driver for Kubernetes FlexVolume Plugin.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| importVol | Text | The name of the Nimble volume to import. |
| snapshot | Text | The name of the Nimble snapshot to restore the imported volume to after takeover. If not specified, the volume will not be restored. |
| restore  | Boolean | Restores the volume to the last snapshot taken on the volume. |
| takeover | Boolean | Indicates the current group will takeover ownership of the Nimble volume and volume collection. This should be performed against a downstream replica. |
| reverseRepl | Boolean | Reverses the replication direction so that writes to the Nimble volume are replicated back to the group where it was replicated from. |
| forceImport | Boolean | Forces the import of a volume that is not owned by the group and is not part of a volume collection. If the volume is part of a volume collection, use takeover instead.

!!! note
    HPE Nimble Docker Volume workflows works with a 1-1 mapping between volume and volume collection.

### HPE Cloud Volumes StorageClass parameters

A `StorageClass` is used to provision or clone an HPE Cloud Volumes-backed persistent volume. It can also be used to import an existing HPE Cloud Volumes volume or clone of a snapshot into the Kubernetes cluster. The parameters are grouped below by those same workflows.

A sample [StorageClass](https://raw.githubusercontent.com/hpe-storage/flexvolume-driver/master/examples/kubernetes/hpe-nimble-storage/sc-cv.yaml) is provided.

!!! note
    These are optional parameters.

#### Common parameters for Provisioning and Cloning

These parameters are mutable betweeen a parent volume and creating a clone from a snapshot.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| nameSuffix| Text   | Suffix to append to Cloud Volumes. |
| destroyOnRm | Boolean | Indicates the backing Cloud volume (including snapshots) should be destroyed when the PVC is deleted. |
| limitIOPS | Integer | The IOPS limit of the volume. The IOPS limit should be in the range 300 to 50000. |
| perfPolicy | Text | The name of the performance policy to assign to the volume. Default example performance policies include "Other, Exchange, Oracle, SharePoint, SQL, Windows File Server". |
| protectionTemplate | Text | The name of the protection template to assign to the volume. Default examples of protection templates include "daily:3, daily:7, daily:14, hourly:6, hourly:12, hourly:24, twicedaily:4, twicedaily:8, twicedaily:14, weekly:2, weekly:4, weekly:8, monthly:3, monthly:6, monthly:12 or none". |
| volumeType | Text | Cloud Volume type. Supported types are PF and GPF. |

#### Provisioning parameters

These parameters are immutable for clones once a volume has been created.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| fsOwner | userId:groupId | The user id and group id that should own the root directory of the filesystem. |
| fsMode | Octal digits | 1 to 4 octal digits that represent the file mode to be applied to the root directory of the filesystem. |
| encryption | Boolean | Indicates that the volume should be encrypted. |

#### Cloning parameters

Cloning supports two modes of cloning. Either use `cloneOf` and reference a PVC in the current namespace or use `importVolAsClone` and reference a Cloud volume name to clone and import to Kubernetes.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| cloneOf | Text | The name of the PV to be cloned. `cloneOf` and `importVolAsClone` are mutually exclusive. |
| importVolAsClone | Text | The name of the Cloud Volume volume to clone and import. `importVolAsClone` and `cloneOf` are mutually exclusive. |
| snapshot | Text | The name of the snapshot to base the clone on. This is optional. If not specified, a new snapshot is created. |
| createSnapshot | Boolean | Indicates that a new snapshot of the volume should be taken matching the name provided in the `snapshot` parameter. If the `snapshot` parameter is not specified, a default name will be created. |
| snapshotPrefix | Text | A prefix to add to the beginning of the snapshot name. |
| replStore | Text | Replication store name. Should be used with importVolAsClone parameter to clone a replica volume |

#### Import parameters

Importing volumes to Kubernetes requires the source Cloud volume to be not attached to any nodes. All previous Access Control Records will be stripped from the volume when put under control of the HPE Volume Driver for Kubernetes FlexVolume Plugin.

| Parameter | String | Description |
| --------- | ------ | ----------- |
| importVol | Text | The name of the Cloud volume to import. |
| forceImport | Boolean | Forces the import of a volume that is provisioned by another K8s cluster but not attached to any nodes. |

## Diagnostics

This section outlines a few troubleshooting steps for the HPE Volume Driver for Kubernetes Plugin. This product is supported by HPE, please consult with your support organization (Nimble, Cloud Volumes etc) prior attempting any configuration changes.

### Troubleshooting FlexVolume driver

The FlexVolume driver is a binary executed by the kubelet to perform mount/unmount/attach/detach operations as workloads request storage resources. The binary relies on communicating with a socket on the host where the volume plugin responsible for the MUAD operations perform control-plane or data-plane operations against the backend system hosting the actual volumes.

#### Locations

The driver has a configuration file where certain defaults can be tweaked to accommodate a certain behavior. Under normal circumstances, this file does not need any tweaking.

The name and the location of the binary varies based on Kubernetes distribution (the default 'exec' path) and what backend driver is being used. In a typical scenario, using Nimble, this is expected:

* Binary: `/usr/libexec/kubernetes/kubelet-plugins/volume/exec/hpe.com~nimble/nimble`
* Config file: `/usr/libexec/kubernetes/kubelet-plugins/volume/exec/hpe.com~nimble/nimble.json`

#### Override defaults

By default, it contains only the path to the socket file for the volume plugin:

```json
{
    "dockerVolumePluginSocketPath": "/etc/hpe-storage/nimble.sock"
}
```

Valid options for the FlexVolume driver can be inspected by executing the binary on the host with the `config` argument:

```markdown
/usr/libexec/kubernetes/kubelet-plugins/volume/exec/hpe.com~nimble/nimble config
Error processing option 'logFilePath' - key:logFilePath not found
Error processing option 'logDebug' - key:logDebug not found
Error processing option 'supportsCapabilities' - key:supportsCapabilities not found
Error processing option 'stripK8sFromOptions' - key:stripK8sFromOptions not found
Error processing option 'createVolumes' - key:createVolumes not found
Error processing option 'listOfStorageResourceOptions' - key:listOfStorageResourceOptions not found
Error processing option 'factorForConversion' - key:factorForConversion not found
Error processing option 'enable1.6' - key:enable1.6 not found

Driver=nimble Version=v2.5.1-50fbff2aa14a693a9a18adafb834da33b9e7cc89
Current Config:
  dockerVolumePluginSocketPath = /etc/hpe-storage/nimble.sock
           stripK8sFromOptions = true
                   logFilePath = /var/log/dory.log
                      logDebug = false
                 createVolumes = false
                     enable1.6 = false
           factorForConversion = 1073741824
  listOfStorageResourceOptions = [size sizeInGiB]
          supportsCapabilities = true
```

An example tweak could be to enable debug logging and enable support for Kubernetes 1.6 (which we don't officially support). The config file would then end up like this:

```json
{
    "dockerVolumePluginSocketPath": "/etc/hpe-storage/nimble.sock",
    "logDebug": true,
    "enable1.6": true
}
```

Execute the binary again (`nimble config`) to ensure the parameters and config file gets parsed correctly. Since the config file is read on each FlexVolume operation, no restart of anything is needed.

See [Advanced](#advanced_configuration) for more parameters for the `driver.json` file.

#### Connectivity
To verify the FlexVolume binary can actually communicate with the backend volume plugin, issue a faux mount request:

```
/usr/libexec/kubernetes/kubelet-plugins/volume/exec/hpe.com~nimble/nimble mount no/op '{"name":"myvol1"}'
```

If the FlexVolume driver can successfully communicate with the volume plugin socket:

```json
{"status":"Failure","message":"configured to NOT create volumes"}
```

In the case of any other output, check if the backend volume plugin is alive with `curl`:

```shell
curl --unix-socket /etc/hpe-storage/nimble.sock -d '{}' http://localhost/VolumeDriver.Capabilities
```

It should output:

```json
{"capabilities":{"scope":"global"},"Err":""}
```

### FlexVolume and dynamic provisioner driver logs

Log files associated with the HPE Volume Driver for Kubernetes FlexVolume Plugin logs data to the standard output stream. If the logs need to be retained for long term, use a standard logging solution. Some of the logs on the host are persisted which follow standard logrotate policies.

FlexVolume driver logs:
```shell
kubectl logs -f daemonset.apps/hpe-flexvolume-driver -n kube-system
```

The logs are persisted at `/var/log/hpe-docker-plugin.log` and `/var/log/dory.log`

Dynamic Provisioner logs:

```shell
kubectl logs -f  deployment.apps/hpe-dynamic-provisioner -n kube-system
```

The logs are persisted at `/var/log/hpe-dynamic-provisioner.log`

### Log Collector

Log collector script `hpe-logcollector.sh` can be used to collect diagnostic logs using kubectl

Download the script as follows:

```shell
curl -O https://raw.githubusercontent.com/hpe-storage/flexvolume-driver/master/hpe-logcollector.sh
chmod 555 hpe-logcollector.sh
```

Usage:

```shell
./hpe-logcollector.sh -h
Diagnostic Script to collect HPE Storage logs using kubectl

Usage:
     hpe-logcollector.sh [-h|--help][--node-name NODE_NAME][-n|--namespace NAMESPACE][-a|--all]
Where
-h|--help                  Print the Usage text
--node-name NODE_NAME      where NODE_NAME is kubernetes Node Name needed to collect the
                           hpe diagnostic logs of the Node
-n|--namespace NAMESPACE   where NAMESPACE is namespace of the pod deployment. default is kube-system
-a|--all                   collect diagnostic logs of all the nodes.If
                           nothing is specified logs would be collected
                           from all the nodes
```

## Advanced Configuration
This section describes some of the advanced configuration steps available to tweak behavior of the HPE Volume Driver for Kubernetes FlexVolume Plugin. 

### Set defaults at the compute node level
During normal operations, defaults are set in either the `ConfigMap` or in a `StorageClass` itself. The picking order is:

- StorageClass
- ConfigMap
- driver.json

Please see [Diagnostics](#diagnostics) to locate the driver for your particular environment. Add this object to the configuration file, `nimble.json`, for example:

```json
{
    "defaultOptions": [{"option1": "value1"}, {"option2": "value2"}]
}
```
Where `option1` and `option2` are valid backend volume plugin create options.

!!! note
     It's highly recommended to control defaults with `StorageClass` API objects or the `ConfigMap`.

### Global options
Each driver supports setting certain "global" options in the `ConfigMap`. Some options are common, some are driver specific.

#### Common

| Parameter | String | Description |
| --------- | ------ | ----------- |
| volumeDir | Text   | Root directory on the host to mount the volumes. This parameter needs correlation with the `podsmountdir` path in the `volumeMounts` stanzas of the deployment. | 
| logDebug  | Boolean | Turn on debug logging, set to false by default. |

