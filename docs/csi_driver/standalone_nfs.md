# Standalone NFS Server

In certain situations is desirable to run the NFS Server Provisioner image without the dual `PersistentVolumeClaim` (PVC) semantic in a more static fashion on top of a `PVC` provisioned by a non-HPE CSI Driver `StorageClass`.

!!! caution "Notice"
    Since HPE CSI Driver for Kubernetes v2.4.1, this functionality is built into the CSI driver. See [Using a Foreign StorageClass](using.md#using_a_foreign_storageclass) how to use it.

[TOC]

## Limitations

- The standalone NFS server is not part of the HPE CSI Driver and should be considered a standalone Kubernetes application altogether. The HPE CSI Driver NFS Server Provisioner NFS servers may co-exist on the same cluster and `Namespace` without risk of conflict but not recommended.
- The [Pod Monitor](monitor.md) which normally monitors `Pods` status for the "NodeLost" condition is not included with the standalone NFS server and recovery is at the mercy of the underlying storage platform and driver.
- Support is limited on the standalone NFS server and only available to select users.

## Prerequisites

It's assumed during the creation steps that a Kubernetes cluster is available with enough permissions to deploy privileged `Pods` with `SYS_ADMIN` and `DAC_READ_SEARCH` capabilities. All steps are run in a terminal with `kubectl` and `git` in in the path.

- A default `StorageClass` declared on the cluster
- Worker nodes that will serve the NFS exports *must* be labeled with `csi.hpe.com/hpe-nfs: "true"`
- `kubectl` and Kubernetes v1.21 or newer

## Create a Workspace

NFS server configurations are managed with the kustomize templating system. Clone this repository to get started and change working directory.

```text
git clone https://github.com/hpe-storage/scod
cd scod/docs/csi_driver/examples/standalone_nfs
```

In the current directory, various manifests and configuration directives exist to deploy and manage NFS servers.

Run `tree .` in the current directory: 

```text
.
├── base
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── environment.properties
│   ├── kustomization.yaml
│   ├── pvc.yaml
│   ├── service.yaml
│   └── values.yaml
└── overlays
    └── example
        ├── deployment.yaml
        ├── environment.properties
        └── kustomization.yaml

4 directories, 10 files
```

!!! important 
    The current directory is now the "home" for the remainder of this guide.

## Create an NFS Server

Copy the "example" overlay into a new directory. In the examples "my-server" is used.

```text
cp -a overlays/example overlays/my-server
```

Edit both "environment.properties" and "kustomization.yaml" in the newly created overlay. Also pay attention to if the remote `Pods` mounting the NFS export are running as a non-root user, if that's the case, the group ID is needed of those `Pods` (customizable per NFS server).

### environment.properties

```text
# This is the domain associated with worker node (not inter-cluster DNS)
CLUSTER_NODE_DOMAIN_NAME=my-domain.example.com

# The size of the backend RWO claim
PERSISTENCE_SIZE=16Gi

# Default resource limits for the NFS server
NFS_SERVER_CPU_LIMIT=1
NFS_SERVER_MEMORY_LIMIT=2Gi
```

The "CLUSTER_NODE_DOMAIN_NAME" variable refers to the DNS domain name that the worker node is resolvable in, not the Kubernetes cluster DNS.

The "PERSISTENCE_SIZE" is the backend `PVC` size expressed in the same format accepted by a `PVC`.

Configuring resource limits are optional but recommended for high performance workloads.

### kustomization.yaml

Change the resource prefix in "kustomization.yaml" either with an editor or `sed`:

```text
sed -i"" 's/example-/my-server-/g' overlays/my-server/kustomization.yaml
```

!!! Seealso
    If the NFS server needs to be deployed in a different `Namespace` than the current, edit and uncomment the "namespace" parameter in `overlays/my-server/kustomization.yaml`.

### Change the default fsGroup

The default "fsGroup" is mapped to "nobody" (gid=65534) which allows remote `Pods` run as the root user to write in the NFS export. This may not be desirable as best practices dictate that `Pods` should run with a user id larger than 99.

To allow user `Pods` to write in the export, edit `overlays/my-server/deployment.yaml` and change the "fsGroup" to the corresponding gid running in the remote `Pod`.

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpe-nfs
spec:
  template:
    spec:
      securityContext:
        fsGroup: 65534
        fsGroupChangePolicy: OnRootMismatch
```

Deploy the NFS server by issuing `kubectl apply -k overlays/my-server`:

```text
configmap/my-server-hpe-nfs-conf created
configmap/my-server-local-conf-97898bftbh created
service/my-server-hpe-nfs created
persistentvolumeclaim/my-server-hpe-nfs created
deployment.apps/my-server-hpe-nfs created
```

Inspect the resources with `kubectl get -k overlays/my-server`:

```text
NAME                                        DATA   AGE
configmap/my-server-hpe-nfs-conf            1      59s
configmap/my-server-local-conf-97898bftbh   2      59s

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                               AGE
service/my-server-hpe-nfs   ClusterIP   10.100.200.11   <none>        49000/TCP,2049/TCP,2049/UDP,32803/TCP,32803/UDP,20048/TCP,20048/UDP,111/TCP,111/UDP,662/TCP,662/UDP,875/TCP,875/UDP   59s

NAME                                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/my-server-hpe-nfs   Bound    pvc-ae943116-d0af-4696-8b1b-1dcf4316bdc2   18Gi       RWO            vsphere-sc     58s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-server-hpe-nfs   1/1     1            1           59s
```

Make a note of the IP address assigned to "service/my-server-hpe-nfs", that is the IP address needed to mount the NFS export.

!!! tip
    If the Kubernetes cluster DNS service is resolvable from the worker node host OS, it possible to use the cluster DNS address to mount the `Service`, in this example that would be "my-server-hpe-nfs.default.svc.cluster.local".

## Mounting the NFS Server

There are two ways to mount the NFS server.

1. Inline declaration of where to find the NFS server and NFS Export
1. Statically creating a `PersistentVolume` with the NFS server details and mount options and manually claiming the `PV` with a `PVC` using the `.spec.volumeName` parameter

### Inline Declaration

This is the most elegant solution as it does not require any intermediary `PVC` or `PV` and directly refers to the NFS server within a workload stanza.

This is an example from a `StatefulSet` workload controller having multiple replicas.

```text
...
spec:
  replicas: 3
  template:
    ...
    spec:
      containers:
        volumeMounts:
        - name: vol
          mountPath: /vol
      ...
      volumes:
      - name: vol
        nfs:
          server: 10.100.200.11
          path: /export
```

!!! important
    Replace `.spec.template.spec.volumes[].nfs.server` with IP address from the actual `Service` IP address and not the examples.

### Static Provisioning

Refer to the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) for the built-in NFS client on how to perform static provisioning of NFS `PVs` and `PVCs`.

## Expand PVC

If the `StorageClass` and underlying CSI driver supports volume expansion, simply edit `overlays/my-server/environment.properties` with the new (larger) size and issue `kubectl apply -k overlays/my-server` to expand the volume.

## Deleting the NFS Server

Ensure no workloads have active mounts against the NFS server `Service`. If there are, those `Pods` will be stuck indefinitely. 

Run `kubectl delete -k overlays/my-server`:

```text
configmap "my-server-hpe-nfs-conf" deleted
configmap "my-server-local-conf-97898bftbh" deleted
service "my-server-hpe-nfs" deleted
persistentvolumeclaim "my-server-hpe-nfs" deleted
deployment.apps "my-server-hpe-nfs" deleted
```

!!! caution
    Unless the `StorageClass` "reclaimPolicy" is set to "Retain". The underlying `PV` will be deleted from the cluster and data needs to be restored from backups if needed.
