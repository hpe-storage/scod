# Introduction

[TOC]

## Platform Requirements

The HPE Alletra Storage MP B10000 needs to be running OS 10.5 or later to take advantage of the File Service. Further, there needs to be IP ports configured on the array that are reachable from the Kubernetes compute nodes.

## StorageClass parameters

The CSP only supports dynamic provisioning of `PersistentVolumes` and no data management such as snapshot, cloning or converting block to file volumes.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| accessProtocol                 | Text    | Mandatory, set to "nfs". Defaults to "iscsi" when unspecified. |
| destroyOnDelete<sup>1</sup>    | Boolean | Indicates the backing volume should be removed when the PVC is deleted. Defaults to "false" which means volumes needs to be pruned manually by a storage administrator. Defaults to "false". |
| shareSquashingOption           | Text    | Controls the NFS client UID to server UID mapping. Valid options: "no_root_squash", "root_squash", "all_squash". Defaults to "no_root_squash". |
| shareReadOnly                  | Boolean | Sets the server share read only. Defaults to "false". |
| shareNfsVersion                | Float   | For future use, Defaults to "4". |

<small><sup>1</sup> = This parameter has no effect if the `PersistentVolume` being removed is empty (no user data).</small>

Example default `StorageClass` ([download](examples/storageclass.yaml)):

```yaml
{% include "csi_driver/container_storage_provider/hpe_alletra_storage_mp_b10000_file_service/examples/storageclass.yaml" %}```

!!! hint
    If all mutable parameters have values provided during provisioning of the `PersistentVolumes`, the [Volume Mutator](../../using.md#using_volume_mutations) will later allow changes if needed.

## Using subPath, Inline NFS and Static NFS Persistent Volumes

Due to the constraints of the share count of the File Service on the array, there are a couple of workarounds that can be performed from the Kubernetes side to get better utilization.

!!! caution
    The Inline NFS and Static NFS Persistent Volumes examples are provided to customers that prefer end-to-end static provisioning of storage resources. It's not recommended to refer to HPE CSI Driver provisioned file shares either inline or with static PVs as paths and client access restrictions may change in future versions of the CSI driver.

### subPath

Assume there is a file `PVC` created. In normal circumstances a handful of workloads in a `Namespace` would access the `PVC` as one shared filesystem. If a different workload in the `Namespace` would require a separate filesystem a new `PVC` would be the appropriate step to create separation. By using "subPath", workloads can be separated on the `PVC` by using directories, one sub directory for each workload.

!!! tip "Good to know"
    The "subPath" functionality is not provided by the HPE CSI Driver, it's a generic construct provided by Kubernetes.

Example:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-pod-sub
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
          subPath: nginx
    - name: pod-datelog-2
      image: debian
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
          subPath: debian
    - name: pod-datelog-3
      image: ubuntu
      command: ["bin/sh"]
      args: ["-c", "while true; do sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
  volumes:
    - name: export1
      persistentVolumeClaim:
        claimName: my-first-pvc
```

Use `kubectl exec` to examine the directory structure in the third pod, without "subPath":

```text
$ kubectl exec my-pod-sub -c pod-datelog-3 -- find /data
/data
/data/nginx
/data/nginx/mydata.txt
/data/debian
/data/debian/mydata.txt
```

It's clear that the first and second `Pod` have different roots in the filesystem.

### Inline NFS

Inline NFS does NOT require the HPE CSI Driver, this procedure leverages the kubelet provide NFS client. It's expected that a share have been created on the array manually. Assume we have a filesystem volume named "volume" and the share name is "example" on the share IP address of 192.168.1.100.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-pod-inline
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
  volumes:
    - name: export1
      nfs:
        server: 192.168.1.100
        path: /file/volume/example
```

### Static NFS Persistent Volumes

Similar to inline NFS, static NFS `PVs` does not require the HPE CSI Driver. The file share needs to exist prior.

Create a static `PV` referencing an existing file share on the array:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-nfs-pv
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  nfs:
    path: /file/volume/example
    server: 192.168.1.100
```

Next, create a `PVC` referencing the `PV`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-first-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 8Gi
  volumeName: my-nfs-pv
  storageClassName: ""
```

Workloads can now reference the `PVC` as usual.

## Static Provisioning for Orphaned Persistent Volumes

In the event of a file share becoming an orphan on the array due to "destroyOnDelete" not being set properly, it's possible to statically define the `PersistentVolume` (`PV`) and claim the `PV` manually.

First, a `PV` needs to be created manually referencing the existing file share.

This table describes how to retrieve the attributes needed.

| Key | Value | Description |
| --- | ----- | ----------- |
| metadata.name | pvc-UUID | This is referenced on the array as the file share name. |
| spec.capacity.storage | Size in GiB | This must match the size of the existing file share filesystem. |
| spec.csi.volumeHandle | referenceUid | This value can be retrieved from URL in the web UI of the array while viewing the file share. |
| spec.csi.volumeAttributes.mountPath | Path | This is referenced on the array as path. |

Replace the zeroed fields below with the actual values from the file share on the array.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvc-00000000-0000-0000-0000-000000000000
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 22Gi
  csi:
    volumeHandle: 00000000000000000000000000000000
    driver: csi.hpe.com
    volumeAttributes:
      accessProtocol: nfs
      volumeAccessMode: mount
      mountPath: /file/pvc-00000000-0000-0000-0000-000000000000/pvc-00000000-0000-0000-0000-000000000000
    controllerPublishSecretRef:
      name: hpe-file-backend
      namespace: hpe-storage
    nodePublishSecretRef:
      name: hpe-file-backend
      namespace: hpe-storage
    controllerExpandSecretRef:
      name: hpe-file-backend
      namespace: hpe-storage
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
```

Next, create a `PVC` referencing the `PV`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-static-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 22Gi
  volumeName: pvc-00000000-0000-0000-0000-000000000000
  storageClassName: ""
```

The `PVC` may now be referenced and attached to a workload.

## Limitations

These are the current limitations of the HPE Alletra Storage MP B10000 File Service CSP.

- Snapshots and clones are not yet implemented.
- Maximum 16 File Service shares may exist at any given time per array controller.
- Maximum 20% of the array provisioned capacity may be used for File Service storage.
- Maximum 64TiB capacity per file share.
- Export permissions on the array are set to `*` (defined as "all" in the array interfaces) and will be reachable from any host that can reach the File interfaces on the array.

## Support

Please refer to the HPE Alletra Storage MP B10000, Alletra 9000 and Primera and 3PAR Storage CSP [support statement](../../../legal/support/index.md#hpe_alletra_storage_mp_b10000_alletra_9000_and_primera_and_3par_container_storage_provider_support).
