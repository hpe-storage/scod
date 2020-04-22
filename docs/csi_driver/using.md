# Overview
At this point the CSI driver and CSP should be configured. If you used either the Helm chart or Operator, you most likely have a base `StorageClass` to provision storage from.

[TOC]

## Enabling CSI snapshots

Support for `VolumeSnapshotClass` is available from Kubernetes 1.17+. The snapshot beta CRDs and the common snapshot controller needs to be installed manually. As per Kubernetes SIG Storage, these should not be installed as part of a CSI driver and should be deployed by the Kubernetes cluster vendor or user.

Install snapshot beta CRDs.

```markdown
kubectl create  -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl create  -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl create  -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
```

Install common snapshot controller.

```markdown
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

!!! tip
    The [provisioning](#provisioning_concepts) section contains examples on how to create `VolumeSnapshotClass` and `VolumeSnapshot` objects.

## Base StorageClass parameters
Each CSP has its own set of unique parameters to control the provisioning behavior. These examples serve as a base `StorageClass` example for each version of Kubernetes. See the respective [CSP](../container_storage_provider/index.md) for more elaborate examples.

```markdown fct_label="K8s 1.15+"
# Renamed csi.storage.k8s.io/resizer-secret-name to
# csi.storage.k8s.io/controller-expand-secret-name
#
# Alpha features: PVC cloning and Pod inline ephemeral volumes
#
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: hpe-storageclass
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/controller-expand-secret-name: hpe-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: hpe-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: hpe-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: hpe-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/provisioner-secret-name: hpe-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
reclaimPolicy: Delete
allowVolumeExpansion: true
```

```markdown fct_label="K8s 1.14"
# Alpha feature: volume expansion
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: hpe-storageclass
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/resizer-secret-name: hpe-secret
  csi.storage.k8s.io/resizer-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: hpe-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: hpe-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: hpe-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/provisioner-secret-name: hpe-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
reclaimPolicy: Delete
# Required to allow volume expansion
allowVolumeExpansion: true
```

```markdown fct_label="K8s 1.13"
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: hpe-storageclass
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/controller-publish-secret-name: hpe-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: hpe-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: hpe-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/provisioner-secret-name: hpe-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
reclaimPolicy: Delete
```

## Provisioning concepts

These instructions are provided as an example on how to use the HPE CSI Driver with a [CSP](../container_storage_provider/index.md) supported by HPE.

### Create a PersistentVolumeClaim from a StorageClass

The below YAML declarations are meant to be created with `kubectl create`. Either copy the content to a file on the host where `kubectl` is being executed, or copy & paste into the terminal, like this:

```markdown
kubectl create -f-
< paste the YAML >
^D (CTRL + D)
```

To get started, create a `StorageClass` API object referencing the CSI driver secret `hpe-secret`.

These examples are for Kubernetes 1.15+

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-scod
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/controller-expand-secret-name: hpe-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: hpe-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: hpe-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: hpe-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/provisioner-secret-name: hpe-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
reclaimPolicy: Delete
allowVolumeExpansion: true
```

Create a `PersistentVolumeClaim`. This object declaration ensures a `PersistentVolume` is created and provisioned on your behalf, make sure to reference the correct `.spec.storageClassName`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-first-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-scod
```

!!! note
    In most enviornments, there is a default `StorageClass` declared on the cluster. In such a scenario, the `.spec.storageClassName` can be omitted. The default `StorageClass` is controlled by an annotation: `.metadata.annotations.storageclass.kubernetes.io/is-default-class` set to either `"true"` or `"false"`.

After the PVC has been declared, check that a new `PersistentVolume` is created based on your claim:

```markdown
kubectl get pv
NAME              CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM                STORAGECLASS AGE
pvc-13336da3-7... 32Gi     RWO          Delete         Bound  default/my-first-pvc hpe-scod     3s
```

The above output means that the HPE CSI Driver successfully provisioned a new volume. The volume is not attached to any node yet. It will only be attached to a node if a workload is scheduled requesting the PVC. Now, let us create a `Pod` that refers to the above volume. When the `Pod` is created, the volume will be attached, formatted and mounted according to the specification.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-pod
spec:
  containers:
    - name: pod-datelog-1
      image: nginx
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
    - name: pod-datelog-2
      image: debian
      command: ["bin/sh"]
      args: ["-c", "while true; do date >> /data/mydata.txt; sleep 1; done"]
      volumeMounts:
        - name: export1
          mountPath: /data
  volumes:
    - name: export1
      persistentVolumeClaim:
        claimName: my-first-pvc
```

Check if the `Pod` is running successfully.

```markdown
kubectl get pod my-pod 
NAME        READY   STATUS    RESTARTS   AGE
my-pod      2/2     Running   0          2m29s
```

!!! tip
    A simple `Pod` does not provide any automatic recovery if the node the `Pod` is scheduled on crashes or become unresponsive. Please see [the official Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/) for different workload types that provide automatic recovery. A shortlist of recommended workload types that are suitable for persistent storage is available in [this blog post](https://datamattsson.tumblr.com/post/182297931146/highly-available-stateful-workloads-on-kubernetes) and best practices are outlined in [this blog post](https://datamattsson.tumblr.com/post/185031432701/best-practices-for-stateful-workloads-on).

The default `volumeMode` for the `PersistentVolumeClaim` is set to `Filesystem`. If a Raw Block Device is desired, `volumeMode` needs to be set to `Block`. Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-block
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-scod
  volumeMode: Block
```

Creating the PVC is identical to `volumeMode: Filesystem`, mapping the device in a `Pod` specification is slightly different as a `volumeDevices` section is added instead of a `volumeMounts` stanza:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-block
spec:
  containers:
    - name: my-null-pod
      image: fedora:31
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: my-pvc-block
```

### Using CSI snapshots

CSI introduces snapshots as native objects in Kubernetes to allow end-users to provision `VolumeSnapshot` objects from an existing PVC. New PVCs may then be created using the snapshot as a source.

Start by creating a `VolumeSnapshotClass` referencing the `hpe-secret` and defining additional snapshot parameters.

Kubernetes 1.17+ (CSI snapshots in beta)

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotClass
metadata:
  name: hpe-snapshot
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: csi.hpe.com
deletionPolicy: Delete
parameters:
  description: "Snapshot created by the HPE CSI Driver"
  csi.storage.k8s.io/snapshotter-secret-name: hpe-secret
  csi.storage.k8s.io/snapshotter-secret-namespace: kube-system
```

Create a `VolumeSnapshot`. This will create a new snapshot of the volume.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  source:
    persistentVolumeClaimName: my-pvc
```

!!! tip
    If a specific `VolumeSnapshotClass` is desired, use `.spec.snapshotClassName` to call it out.

Check that a new `VolumeSnapshot` is created based on your claim:

```markdown
kubectl describe volumesnapshot my-snapshot
Name:         my-snapshot
Namespace:    default
...
Status:
  Creation Time:  2019-05-22T15:51:28Z
  Ready:          true
  Restore Size:   32Gi
```

!!! seealso "Learn more"
    For a more comprehensive tutorial on how to use snapshots and clones with CSI on Kubernetes 1.17, see [HPE CSI Driver for Kubernetes: Snapshots, Clones and Volume Expansion](https://developer.hpe.com/blog/PklOy39w8NtX6M2RvAxW/hpe-csi-driver-for-kubernetes-snapshots-clones-and-volume-expansion) on HPE DEV.

### Expanding PVCs

To perform expansion operations on Kubernetes 1.14+, you must enhance your `StorageClass` with some additional attributes. Please see [base `StorageClass` parameters](#base_storageclass_parameters).

Then, a volume provisioned by a `StorageClass` with expansion attributes may have its PVCs expanded by altering the `.spec.resources.requests.storage` key of the `PersistentVolumeClaim`.

This may be done by the `kubectl patch` command.

```markdown
kubectl patch pvc/my-pvc --patch '{"spec": {"resources": {"requests": {"storage": "64Gi"}}}}'
persistentvolumeclaim/my-pvc patched
```

The new PVC size may be observed with `kubectl get pvc/my/pvc` after a few moments.

### Using PVC Overrides

The HPE CSI Driver allows the `PersistentVolumeClaim` to override the `StorageClass` parameters by annotating the `PersistentVolumeClaim`. Define the parameters allowed to be overridden in the `StorageClass` by setting the `allowOverrides` parameter:

```yaml fct_label="HPE Nimble Storage"
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-scod-override
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/provisioner-secret-name: nimble-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: nimble-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: nimble-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: nimble-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  description: "Volume provisioned by the HPE CSI Driver"
  accessProtocol: "iscsi"
  allowOverrides: description,accessProtocol
```

```yaml fct_label="HPE 3PAR and Primera Storage"
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-scod-override
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: ext4
  csi.storage.k8s.io/provisioner-secret-name: hpe-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: hpe-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: hpe-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: hpe-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  cpg: "FC_r6"
  provisioning_type: "tpvv"
  accessProtocol: "iscsi"
  allowOverrides: cpg,provisioning_type
```

The end-user may now control those parameters (the `StorageClass` provides the default values).

```yaml fct_label="HPE Nimble Storage"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-override
  annotations:
    csi.hpe.com/description: "This is my custom description"
    csi.hpe.com/accessProtocol: "fc"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: hpe-scod-override
```

```yaml fct_label="HPE 3PAR and Primera Storage"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-override
  annotations:
    csi.hpe.com/provisioning_type: "full"
    csi.hpe.com/cpg: "SSD_r6"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: hpe-scod-override
```

## Further reading

The [official Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/) contains comprehensive documentation on how to markup `PersistentVolumeClaim` and `StorageClass` API objects to tweak certain behaviors.

Each CSP has a set of unique `StorageClass` parameters that may be tweaked to accommodate a wide variety of use cases. Please see the documentation of [the respective CSP for more details](../container_storage_provider/index.md).
