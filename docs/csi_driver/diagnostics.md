# Introduction

It's recommended to familiarize yourself with inspecting workloads on Kubernetes. This particular [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods) is very useful to have readily available. 

## Sanity Checks

Once the CSI driver has been deployed either through object configuration files, Helm or an Operator. This view should be representative of what a healthy system should look like after install. If any of the workload deployments lists anything but `Running`, proceed to inspect the logs of the problematic workload.

```markdown fct_label="HPE Alletra 6000 and Nimble Storage"
kubectl get pods --all-namespaces -l 'app in (nimble-csp, hpe-csi-node, hpe-csi-controller)'
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
hpe-storage   hpe-csi-controller-7d9cd6b855-zzmd9   9/9     Running   0          15s
hpe-storage   hpe-csi-node-dk5t4                    2/2     Running   0          15s
hpe-storage   hpe-csi-node-pwq2d                    2/2     Running   0          15s
hpe-storage   nimble-csp-546c9c4dd4-5lsdt           1/1     Running   0          15s
```

```markdown fct_label="HPE Alletra 9000, Primera and 3PAR"
kubectl get pods --all-namespaces -l 'app in (primera3par-csp, hpe-csi-node, hpe-csi-controller)'
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
hpe-storage   hpe-csi-controller-7d9cd6b855-fqppd   9/9     Running   0          14s
hpe-storage   hpe-csi-node-86kh6                    2/2     Running   0          14s
hpe-storage   hpe-csi-node-k8p4p                    2/2     Running   0          14s
hpe-storage   hpe-csi-node-r2mg8                    2/2     Running   0          14s
hpe-storage   hpe-csi-node-vwb5r                    2/2     Running   0          14s
hpe-storage   primera3par-csp-546c9c4dd4-bcwc6      1/1     Running   0          14s
```

A Custom Resource Definition (CRD) named `hpenodeinfos.storage.hpe.com` holds important network and host initiator information. 

Retrieve list of nodes.

```markdown
kubectl get hpenodeinfos
$ kubectl get hpenodeinfos
NAME               AGE
tme-lnx-worker1   57m
tme-lnx-worker3   57m
tme-lnx-worker2   57m
tme-lnx-worker4   57m
```

Inspect a node.

```markdown
kubectl get hpenodeinfos/tme-lnx-worker1 -o yaml
apiVersion: storage.hpe.com/v1
kind: HPENodeInfo
metadata:
  creationTimestamp: "2020-08-24T23:50:09Z"
  generation: 1
  managedFields:
  - apiVersion: storage.hpe.com/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        .: {}
        f:chap_password: {}
        f:chap_user: {}
        f:iqns: {}
        f:networks: {}
        f:uuid: {}
    manager: csi-driver
    operation: Update
    time: "2020-08-24T23:50:09Z"
  name: tme-lnx-worker1
  resourceVersion: "30337986"
  selfLink: /apis/storage.hpe.com/v1/hpenodeinfos/tme-lnx-worker1
  uid: 3984752b-29ac-48de-8ca0-8381532cbf06
spec:
  chap_password: RGlkIHlvdSByZWFsbHkgZGVjb2RlIHRoaXM/
  chap_user: chap-user
  iqns:
  - iqn.1994-05.com.redhat:828e7a4eef40
  networks:
  - 10.2.2.2/16
  - 172.16.6.115/24
  - 172.16.8.115/24
  - 172.17.0.1/16
  - 10.1.1.0/12
  uuid: 0242f811-3995-746d-652d-6c6e78352d77
```

## NFS Server Provisioner Resources

The NFS Server Provisioner consists of a number of Kubernetes resources per PVC. The default `Namespace` where the resources are deployed is "hpe-nfs" but is configurable in the `StorageClass`. See [base `StorageClass` parameters](using.md#base_storageclass_parameters) for more details.

| Object                | Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Purpose       |
| --------------------- | -------------------- | ------------- |
| ConfigMap             | hpe-nfs-config       | This `ConfigMap` holds the configuration file for the NFS server. Local tweaks may be wanted. Please see the [config file reference](https://github.com/nfs-ganesha/nfs-ganesha/tree/master/src/config_samples) for more details. |
| Deployment            | hpe-nfs-&lt;UUID&gt; | The `Deployment` that is running the NFS `Pod`.  |
| Service               | hpe-nfs-&lt;UUID&gt; | `Pod` `Service` the NFS clients perform mounts against. |
| PVC                   | hpe-nfs-&lt;UUID&gt; | The RWO claim serving the NFS workload. |

!!! tip
    The `<UUID>` stems from the user request RWX claim UUID for easy tracking.

## Volume and Snapshot Groups

If there's issues with `VolumeSnapshots` not being created when performing `SnapshotGroup` snapshots, checking the logs of the "csi-volume-group-provisioner" and "csi-volume-group-snapshotter" in the "hpe-csi-controller" `Deployment`.

```markdown
kubectl logs -n hpe-storage deploy/hpe-csi-controller csi-volume-group-provisioner
kubectl logs -n hpe-storage deploy/hpe-csi-controller csi-volume-group-snapshotter
```

## Logging

Log files associated with the HPE CSI Driver logs data to the standard output stream. If the logs need to be retained for long term, use a standard logging solution for Kubernetes such as Fluentd. Some of the logs on the host are persisted which follow standard logrotate policies.

### CSI Driver Logs

Node driver
```
kubectl logs -f  daemonset.apps/hpe-csi-node  hpe-csi-driver -n hpe-storage
```
Controller driver
```
kubectl logs -f deployment.apps/hpe-csi-controller hpe-csi-driver -n hpe-storage
```

!!! tip
    The logs for both node and controller drivers are persisted at `/var/log/hpe-csi.log`

### Log Level

Log levels for both CSI Controller and Node driver can be controlled using `LOG_LEVEL` environment variable. Possible values are `info`, `warn`, `error`, `debug`, and `trace`. Apply the changes using `kubectl apply -f <yaml>` command after adding this to CSI controller and node container spec as below. For Helm charts this is controlled through `logLevel` variable in `values.yaml`.

```markdown
          env:
            - name: LOG_LEVEL
              value: trace
```

### CSP Logs

CSP logs can be accessed from their respective services.

```markdown fct_label="HPE Alletra 6000 and Nimble Storage"
kubectl logs -f deploy/nimble-csp -n hpe-storage
```

```markdown fct_label="HPE Alletra 9000, Primera and 3PAR"
kubectl logs -f deploy/primera3par-csp -n hpe-storage
```

### Log Collector

Log collector script `hpe-logcollector.sh` can be used to collect the logs from any node which has `kubectl` access to the cluster.

```markdown
curl -O https://raw.githubusercontent.com/hpe-storage/csi-driver/master/hpe-logcollector.sh
chmod 555 hpe-logcollector.sh
```

Usage:

```markdown
./hpe-logcollector.sh -h
Collect HPE storage diagnostic logs using kubectl.

Usage:
     hpe-logcollector.sh [-h|--help] [--node-name NODE_NAME] \
                         [-n|--namespace NAMESPACE] [-a|--all]
Options:
-h|--help                  Print this usage text
--node-name NODE_NAME      Collect logs only for Kubernetes node
                           NODE_NAME
-n|--namespace NAMESPACE   Collect logs from HPE CSI deployment in namespace
                           NAMESPACE (default: kube-system)
-a|--all                   Collect logs from all nodes (the default)
```

## Tuning

HPE provides a set of well tested defaults for the CSI driver and all the supported CSPs. In certain case it may be necessary to fine tune the CSI driver to accommodate a certain workload or behavior. 

### Data Path Configuration

The HPE CSI Driver for Kubernetes automatically configures Linux iSCSI/multipath settings based on [config.json](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/helm/charts/hpe-csi-driver/files/config.json). In order to tune these values, edit the config map with `kubectl edit configmap hpe-linux-config -n hpe-storage` and restart node plugin using `kubectl delete pod -l app=hpe-csi-node` to apply.

!!! important
    HPE provide a set of general purpose default values for the IO paths, tuning is only required if prescribed by HPE.
