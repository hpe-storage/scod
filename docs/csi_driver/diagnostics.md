# Introduction

It's recommended to familiarize yourself with inspecting workloads on Kubernetes. This particular [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods) is very useful to have readily available. 

## Sanity Checks

Once the CSI driver has been deployed either through object configuration files, Helm or an Operator. This view should be representative of what a healthy system should look like after install. If any of the workload deployments lists anything but `Running`, proceed to inspect the logs of the problematic workload.

```text fct_label="HPE Alletra Storage MP B10000, Alletra 9000, Primera and 3PAR"
kubectl get pods --all-namespaces -l 'app in (primera3par-csp, hpe-csi-node, hpe-csi-controller, alletrastoragemp-b10000-nfs-csp)'
NAMESPACE     NAME                                               READY   STATUS    RESTARTS   AGE
hpe-storage   alletrastoragemp-b10000-nfs-csp-6fccc4cbdf-dmdst   1/1     Running   0          34m
hpe-storage   hpe-csi-controller-5f7d7cf87-l9kc5                 9/9     Running   0          34m
hpe-storage   hpe-csi-node-kv6ff                                 2/2     Running   0          34m
hpe-storage   hpe-csi-node-q7nqn                                 2/2     Running   0          34m
hpe-storage   primera3par-csp-d95885d54-kfcwt                    1/1     Running   0          34m
```

```text fct_label="HPE Alletra 5000/6000 and Nimble Storage"
kubectl get pods --all-namespaces -l 'app in (nimble-csp, hpe-csi-node, hpe-csi-controller)'
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
hpe-storage   hpe-csi-controller-5f7d7cf87-l9kc5   9/9     Running   0          35m
hpe-storage   hpe-csi-node-kv6ff                   2/2     Running   0          35m
hpe-storage   hpe-csi-node-q7nqn                   2/2     Running   0          35m
hpe-storage   nimble-csp-6d9bf4cb5b-lh5tf          1/1     Running   0          35m
```

A Custom Resource Definition (CRD) named `hpenodeinfos.storage.hpe.com` holds important network and host initiator information. 

Retrieve list of nodes.

```text
kubectl get hpenodeinfos
$ kubectl get hpenodeinfos
NAME               AGE
tme-lnx-worker1   57m
tme-lnx-worker3   57m
tme-lnx-worker2   57m
tme-lnx-worker4   57m
```

Inspect a node.

```yaml
kubectl get hpenodeinfos/tme-lnx-worker1 -o yaml
apiVersion: storage.hpe.com/v1
kind: HPENodeInfo
metadata:
  creationTimestamp: "2026-02-24T23:50:09Z"
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
        f:nqns: {}
    manager: csi-driver
    operation: Update
    time: "2026-02-24T23:50:09Z"
  name: tme-lnx-worker1
  resourceVersion: "30337986"
  selfLink: /apis/storage.hpe.com/v1/hpenodeinfos/tme-lnx-worker1
  uid: 3984752b-29ac-48de-8ca0-8381532cbf06
spec:
  chap_password: RGlkIHlvdSByZWFsbHkgZGVjb2RlIHRoaXM/
  chap_user: chap-user
  iqns:
  - iqn.1994-05.com.redhat:1f21f7d1a19
  networks:
  - 10.132.45.226/22
  - 2001:db8:a000::3b/64
  - fe80::f153:a365:7263:2758/64
  - 2001:db8:b000::3b/64
  - fe80::4121:8da4:7313:67d5/64
  - fd01:0:0:5::2/64
  - fe80::858:52ff:fe44:cfc5/64
  - fe80::874:e4ff:fe12:9f0f/64
  - fd69::2/112
  - fd10:abcd:1234:10::47/64
  - fe80::250:56ff:fe8e:c9de/64
  - fe80::82a:2eff:fe99:96ec/64
  nqns:
  - nqn.2014-08.org.nvmexpress:uuid:6a660060-d81a-49d8-8151-d313e6f62d71
  uuid: 1e6ba563-1623-6411-3039-ac80aeb010ca
  wwpns:
  - 100020677c5d3c41
  - 100020677c5d3c49
  - 1000d06726e092e2
  - 1000d06726e092e3
```

## NFS Server Provisioner Resources

The NFS Server Provisioner consists of a number of Kubernetes resources per PVC. The default `Namespace` where the resources are deployed is "hpe-nfs" but is configurable in the `StorageClass`. See [base `StorageClass` parameters](using.md#base_storageclass_parameters) for more details.

| Object                | Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Purpose       |
| --------------------- | -------------------- | ------------- |
| ConfigMap             | hpe-nfs-config       | This `ConfigMap` holds the configuration file for the NFS server. Local tweaks may be wanted. Please see the [config file reference](https://github.com/nfs-ganesha/nfs-ganesha/tree/master/src/config_samples) for more details. |
| Deployment            | hpe-nfs-UID          | The `Deployment` that is running the NFS `Pod`.  |
| Service               | hpe-nfs-UID          | The `Service` the NFS clients perform mounts against. |
| PVC                   | hpe-nfs-UID          | The RWO claim serving the NFS workload. |

!!! tip
    The UID stems from the user request RWX `PVC` for easy tracking. Use `kubectl get pvc/my-pvc -o jsonpath='{.metadata.uid}{"\n"}'` to retrieve it.

### Tracing NFS resources

When troubleshooting NFS deployments it's common that only the source RWX `PVC` and `Namespace` is known. The next few steps explains how resources can be easily traced.

Retrieve the "hpe-nfs-UID" from the NFS `Pod` by specifying `PVC` and `Namespace` of the RWX `PVC`:
```text
kubectl get pods -l provisioned-by=my-pvc,provisioned-from=my-namespace -A -o jsonpath='{.items[].metadata.labels.app}{"\n"}'
```

Next, enumerate the resources from the "hpe-nfs-UID":
```text
kubectl get pvc,svc,deploy -A -o name --field-selector metadata.name=hpe-nfs-UID
```

Example output:

```text
persistentvolumeclaim/hpe-nfs-UID
service/hpe-nfs-UID
deployment.apps/hpe-nfs-UID
```

If only the `PV` name is known, looking from the backend storage perspective, the `PV` name (and `.spec.claimRef.uid`) contains the UID, for example: "pvc-UID".

!!! seealso "Clarification"
    The `hpe-nfs-UID` is abbreviated, it will contain a real UID added on, for example "hpe-nfs-98ce7c80-13f9-45d0-9609-089227bf97f1".

## Volume and Snapshot Groups

If there's issues with `VolumeSnapshots` not being created when performing `SnapshotGroup` snapshots, checking the logs of the "csi-volume-group-provisioner" and "csi-volume-group-snapshotter" in the "hpe-csi-controller" `Deployment`.

```text
kubectl logs -n hpe-storage deploy/hpe-csi-controller csi-volume-group-provisioner
kubectl logs -n hpe-storage deploy/hpe-csi-controller csi-volume-group-snapshotter
```

## Logging

Log files associated with the HPE CSI Driver logs data to the standard output stream. If the logs need to be retained for long term, use a standard logging solution for Kubernetes such as Fluentd. Some of the logs on the host are persisted which follow standard logrotate policies.

### CSI Driver Logs

Node driver:

```text
kubectl logs -f  daemonset.apps/hpe-csi-node  hpe-csi-driver -n hpe-storage
```

Controller driver:

```text
kubectl logs -f deployment.apps/hpe-csi-controller hpe-csi-driver -n hpe-storage
```

!!! tip
    The logs for both node and controller drivers are persisted at `/var/log/hpe-csi-node.log` and `/var/log/hpe-csi-controller.log` respectively.

### Log Level

Log levels for both CSI Controller and Node driver can be controlled using `LOG_LEVEL` environment variable. Possible values are `info`, `warn`, `error`, `debug`, and `trace`. Apply the changes using `kubectl apply -f <yaml>` command after adding this to CSI controller and node container spec as below. For Helm charts this is controlled through `logLevel` variable in `values.yaml`.

```text
          env:
            - name: LOG_LEVEL
              value: trace
```

### CSP Logs

CSP logs can be accessed from their respective services.

```text fct_label="HPE Alletra Storage MP B10000, Alletra 9000, Primera and 3PAR"
kubectl logs -f deploy/primera3par-csp -n hpe-storage
```

```text fct_label="B10000 File Service"
kubectl logs -f deploy/alletrastoragemp-b10000-nfs-csp -n hpe-storage
```

```text fct_label="HPE Alletra 5000/6000 and Nimble Storage"
kubectl logs -f deploy/nimble-csp -n hpe-storage
```

### Log Collector

Log collector script `hpe-logcollector.sh` can be used to collect the logs from any node which has `kubectl` access to the cluster.

```text
curl -O https://raw.githubusercontent.com/hpe-storage/csi-driver/master/hpe-logcollector.sh
chmod 555 hpe-logcollector.sh
```

Usage:

```text
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
