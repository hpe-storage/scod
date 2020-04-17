# Overview

The HPE 3PAR and Primera CSP is the reference implementation for the [HPE CSI Driver for Kubernetes](../../csi_driver/index.md). The CSP abstract the data management capabilities of the array for use by Kubernetes.

[TOC]

## Platform requirements

### HPE 3PAR and Primera Storage Platform Requirements

|  HPE 3PAR and Primera CSP for Kubernetes  |  Docker EE  |  Linux OS  |  OpenShift  |  Kubernetes  | 3PAR and Primera  |
|-------------------------------------------|------------ |------------|-------------|--------------|-------------------|
|HPE 3PAR and HPE Primera CSP v.1.0.0|Based on Kubernetes version guidelines<br><br><ul><li>For K8s 1.17: Docker Version 19.03.4 is recommended</li><li>For K8s 1.16: Docker Version 18.06.2 is recommended</li></ul>|<ul><li>CentOS: 7.7</li><li>RHEL: 7.6, 7.7 / RHCOS</li></ul>|<ul><li>OpenShift 4.2 with RHEL 7.6 or 7.7 or RHCOS as worker nodes</li></ul>| K8s 1.16, 1.17 |<ul><li>3PAR 3.3.1 MU5 (FC & iSCSI)</li><li>Primera OS: 4.0.0, 4.1.0 (FC only)</li><ul>|

!!! Important
    * Minimum 2 iSCSI IP ports should be in ready state
    * FC array should be in ready state and zoned with initiator hosts

    **Note:** 3PAR supports FC and iSCSI, Primera supports FC protocol

## Deployemnt
Refer [Deployment](../../csi_driver/deployment.md).

### Deploying to Kubernetes
[Deploy using Helm](../../csi_driver/deployment.md#helm)

### Deploying to OpenShift
[Deploy using Operator](../../csi_driver/deployment.md#operator)

## Getting Started
Get started using the Container Storage Provider by setting up `Secret`, `StorageClass`, `PVC` API objects.

### Block storage Usage Table
| CSP for 3PAR and Primera v1.0.0| Features supported on K8S| Features Supported on OpenShift| Notes    |
|--------------------------------|--------------------------|--------------------------------|----------|
|<br>Dynamic provisioning:<br><ul><li>Create volume</li><li>Delete volume</li><li>List volume</li></ul>|YES|  YES ||
| Volume Exapnsion | YES   |   NO   |      |
| Multi User        | YES   |   YES  |      |
| Publish/Unpublish volumes |  YES   |   YES  |      |
| Snapshost and clones  |  YES  |   NO  | k8s version 1.17 onwards|
| Helm Charts  |  YES  |  NA   |     |
| RH Operators Deployment  |  NA  |  YES  |      |

### Step 1: Create a secret
Replace the `password` string (`YWRtaW4=`) below with a base64 encoded version of your password and replace the `backend` with your array IP address and save it as `hpe3parprimera-secret.yaml`.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: hpe3parprimera-secret 
  namespace: kube-system
stringData:
  serviceName: hpe3parprimera-csp-svc
  servicePort: "8080"
  backend: 192.168.1.1
  username: admin  
data:
  # echo -n "admin" | base64
  password: YWRtaW4=
```

Create the secret:

```yaml
kubectl create -f hpe-secret.yaml
secret "hpe3parprimera-secret" created
```

You should now see the HPE secret in the `kube-system` namespace.

### Step 2: Create a storage class

Save below file as `3par-sc.yaml`.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: 3par-sc
provisioner: csi.hpe.com
allowVolumeExpansion: true
parameters:
  csi.storage.k8s.io/fstype: ext4
  csi.storage.k8s.io/provisioner-secret-name: hpe3parprimera-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: hpe3parprimera-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: hpe3parprimera-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: hpe3parprimera-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  # Uncomment for k8s 1.14 for resize support
  #csi.storage.k8s.io/resizer-secret-name: hpe3parprimera-secret
  #csi.storage.k8s.io/resizer-secret-namespace: kube-system
  # Uncomment for k8s 1.15 for resize support
  csi.storage.k8s.io/controller-expand-secret-name: hpe3parprimera-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
  cpg: "FC_r6"
  # tpvv: "False"
  provisioning_type: "tpvv"
  accessProtocol: "iscsi"
```

create a storage class
```yaml
kubectl create -f 3par-sc.yaml
```

!!! Note
    name is user defined name
    <br>Below parameters are valid:
    <br>- <b>snap_cpg</b>: cpg used for creating a snapshot
    <br>- <b>provisioning_type</b>: "full"/"tpvv"/"tdvv"
    <br>- <b>compression</b>: "true"/"false"
    <br>- <b>accessProtocol</b>: "fc"/"iscsi"

### Step 3: Create a PVC

Save below file as `pvc-nginx40.yaml`.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-nginx40
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 19Gi
  storageClassName: 3par-sc
```

create a PVC
```
kubectl create -f pvc-nginx40.yaml
```

### Step 4: Create a POD

Save below file as `pod-hpe.yaml`.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-hpe
spec:
  containers:
  - name: minio
    image: minio/minio:latest
    args:
    - server
    - /export
    env:
    - name: MINIO_ACCESS_KEY
      value: minio
    - name: MINIO_SECRET_KEY
     value: doryspeakswhale
    ports:
    - containerPort: 9000
    volumeMounts:
    - name: export
      mountPath: /export
  volumes:
    - name: export
      persistentVolumeClaim:
        claimName: pvc-nginx40
```

create a POD
```
kubectl create -f pod-hpe.yaml
```

### Step 5: Create a snapshot

#### Step 5.1: Create a snapshotStorageClass

Save below file as `Snapshotclass.yaml`.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotClass
metadata:
  name: my-snapclass-1
driver: csi.hpe.com
deletionPolicy: Delete
parameters:
  description: "Snapshot create by the HPE CSI Driver"
  csi.storage.k8s.io/snapshotter-secret-name: hpe3parprimera-secret
  csi.storage.k8s.io/snapshotter-secret-namespace: kube-system
```

create a snapshotClass
```
kubectl create -f Snapshotclass.yaml
```

!!! Note
```
Valid parameters for snapshotClass are 
read_only : <"true"/"false"> default value is false.
```

#### Step 5.2: Create a snapshot

Save below file as `volumeSnapshot.yaml`.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: my-snapshot-1
spec:
  volumeSnapshotClassName: my-snapclass-1
  source:
    persistentVolumeClaimName: pvc-nginx40
```

create a snapshot
```
kubectl create -f volumeSnapshot.yaml
```

### Step 6: Create a clone

Save below file as `clonePVC.yaml`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-clone-1
spec:
  storageClassName: 3par-sc
  dataSource:  
    name: pvc-nginx40
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

create a clone
```
kubectl create -f clonePVC.yaml
```

!!! Note
    While creating a clone of a PVC, storage should be of same size that of PVC


## Diagnostics and Troubleshooting
### Pre-requisites check
#### Check if all the pods are running 

1. One node plugin on every worker node
2. One controller pod across the cluster, and 
3. One csp pod across the cluster.

Confirm it, by 
- Check for worker nodes.
```
[root@cssosbe01-196119 ~]# kubectl get nodes -o wide
NAME               STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION          CONTAINER-RUNTIME
cssosbe01-196119   Ready    master   39d   v1.16.6   15.212.196.119   <none>        CentOS Linux 7 (Core)   3.10.0-957.el7.x86_64   docker://18.9.7
cssosbe01-196120   Ready    master   39d   v1.16.6   15.212.196.120   <none>        CentOS Linux 7 (Core)   3.10.0-957.el7.x86_64   docker://18.9.7
cssosbe01-196121   Ready    master   39d   v1.16.6   15.212.196.121   <none>        CentOS Linux 7 (Core)   3.10.0-957.el7.x86_64   docker://18.9.7
cssosbe01-196150   Ready    <none>   39d   v1.16.6   15.212.196.150   <none>        CentOS Linux 7 (Core)   3.10.0-957.el7.x86_64   docker://18.9.7
cssosbe01-196151   Ready    <none>   39d   v1.16.6   15.212.196.151   <none>        CentOS Linux 7 (Core)   3.10.0-957.el7.x86_64   docker://18.9.7
```
- Check for Controller,CSP and Node Plugin pods
```
[root@cssosbe01-196119 ~]# kubectl get pods -o wide -n kube-system | grep hpe
hpe-csi-controller-84bdfc8df6-m2wxk        4/4     Running   0          42h   15.212.196.150   cssosbe01-196150   <none>           <none>
hpe-csi-node-cbvcl                         2/2     Running   0          42h   15.212.196.151   cssosbe01-196151   <none>           <none>
hpe-csi-node-w8hx5                         2/2     Running   0          42h   15.212.196.150   cssosbe01-196150   <none>           <none>
hpe3parprimera-csp-778cf87c8b-mffh2        1/1     Running   0          42h   15.212.196.150   cssosbe01-196150   <none>           <none>
```
- Check for REST endpoint of CSP service is reachable
CSP (Container Storage Provider) hosts a REST server on port 8080 for the CSI controller/node plugin to communicate
If the CSP REST endpoint on above port is not reachable, it will lead to failure in provisioning and pod mount operations.

To check whether this Service is up or not, the user can confirm it by
```
[root@k8smaster build]# kubectl get services -n kube-system
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
hpe3parprimera-csp-svc   ClusterIP   10.111.253.102   <none>        8080/TCP                 14h
kube-dns                 ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   141d
``` 
`hpe3parprimera-csp-svc` is the kubernetes service which exposes the REST server running in the CSP pod to the external services via kubernetes internal networking
Try making a health check call like
```
export no_proxy=$no_proxy,10.111.253.102
curl http://10.111.253.102:8080/
```
If the above call goes fine "Welcome!" will be listed

- Check for the multipath file in worker nodes
Eg. /etc/multipath.conf
```
defaults {
    polling_interval 10
    max_fds          8192
}
devices {
    device {
        no_path_retry        18
        rr_weight            uniform
        hardware_handler     "0"
        path_selector        "round-robin 0"
        path_grouping_policy multibus
        product              "VV"
        rr_min_io_rq         1
        features             "0"
        failback             immediate
        vendor               "3PARdata"
        #getuid_callout      "/lib/udev/scsi_id --whitelisted --device=/dev/%n"
        path_checker         tur
    }
    device {
        product              "Server"
        hardware_handler     "1 alua"
        vendor               "Nimble"
        path_grouping_policy group_by_prio
        path_selector        "service-time 0"
        no_path_retry        30
        path_checker         tur
        prio                 alua
        dev_loss_tmo         infinity
        fast_io_fail_tmo     5
        failback             immediate
    }
}

```
2. CSP log file is in the node which runs the csp service (cssosbe01-196150 in above example), Log location `/var/log/hpe-3par-primera-csp.log`
3. Controller plugin log is in node where the hpe-csi-controller* pod (eg 15.212.196.150) is running. Log location: `/var/log/hpe-csi-controller.log`
4. Node plugin log is in node where the  hpe-csi-node* pod is running (eg. ip ending with 150,151) , Log location: `/var/log/hpe-csi-node.log`

### Steps to debug if the pods are in "ContainerCreating" state
1. Identify the pod which is "ContainerCreating" state .. 
eg.
```
[root@cssosbe01-196113 ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                                             READY   STATUS              RESTARTS   AGE    IP               NODE                                     NOMINATED NODE   READINESS GATES
kube-system   coredns-6955765f44-pfwwj                                         1/1     Running             0          2d2h   10.244.0.2       cssosbe01-196113.in.rdlabs.hpecorp.net   <none>           <none>
kube-system   coredns-6955765f44-wlm5g                                         1/1     Running             0          2d2h   10.244.0.3       cssosbe01-196113.in.rdlabs.hpecorp.net   <none>           <none>
kube-system   etcd-cssosbe01-196113.in.rdlabs.hpecorp.net                      1/1     Running             0          2d2h   15.212.196.113   cssosbe01-196113.in.rdlabs.hpecorp.net   <none>           <none>
kube-system   hpe-csi-controller-5b69668588-mk74b                              4/4     Running             0          21h    15.212.196.115   cssosbe01-196115.in.rdlabs.hpecorp.net   <none>           <none>
kube-system   hpe-csi-node-96b5c                                               0/2     ContainerCreating   0          21h    15.212.196.115   cssosbe01-196115.in.rdlabs.hpecorp.net   <none>           <none>
kube-system   hpe-csi-node-p9655                                               0/2     ContainerCreating   0          21h    15.212.196.114   cssosbe01-196114.in.rdlabs.hpecorp.net   <none>           <none>

```

2. Issue `kubectl describe pod <pod-name> -n kube-system`
eg.
```
[root@cssosbe01-196113 ~]# kubectl describe pod hpe-csi-node-96b5c -n kube-system
Name:                 hpe-csi-node-96b5c
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 cssosbe01-196115.in.rdlabs.hpecorp.net/15.212.196.115
Start Time:           Wed, 05 Feb 2020 06:46:05 -0500
Labels:               app=hpe-csi-node
                      controller-revision-hash=764cb85467
                      pod-template-generation=1
                      role=hpe-csi
Annotations:          <none>
Status:               Pending
IP:                   15.212.196.115
IPs:
  IP:           15.212.196.115
Controlled By:  DaemonSet/hpe-csi-node
Containers:
  csi-node-driver-registrar:
    Container ID:
    Image:         quay.io/k8scsi/csi-node-driver-registrar:v1.1.0
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Args:
      --csi-address=$(ADDRESS)
      --kubelet-registration-path=$(DRIVER_REG_SOCK_PATH)
      --v=5
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:
      ADDRESS:               /csi/csi.sock
      DRIVER_REG_SOCK_PATH:  /var/lib/kubelet/plugins/csi.hpe.com/csi.sock
    Mounts:
      /csi/ from plugin-dir (rw)
      /registration from registration-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from hpe-csi-node-sa-token-kx95r (ro)
  hpe-csi-driver:
    Container ID:
    Image:         hpestorage/csi-driver:v1.1.1
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Args:
      --endpoint=$(CSI_ENDPOINT)
      --node-service
      --flavor=kubernetes
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:
      CSI_ENDPOINT:  unix:///csi/csi.sock
    Mounts:
      /csi from plugin-dir (rw)
      /dev from device-dir (rw)
      /etc/hpe-storage from etc-hpe-storage-dir (rw)
      /etc/iscsi from etc-iscsi (rw)
      /etc/kubernetes from etc-kubernetes (rw)
      /etc/multipath from etc-multipath (rw)
      /etc/multipath.conf from etc-multipath-conf (rw)
      /etc/os-release from etc-os-release (rw)
      /etc/redhat-release from etc-redhat-release (rw)
      /lib/modules from lib-modules (rw)
      /lib/systemd/system from libsystemd (rw)
      /opt/hpe-storage/nimbletune/config.json from linux-config-file (rw,path="config.json")
      /run/lock from runlock (rw)
      /run/systemd from runsystemd (rw)
      /sbin/iscsiadm from iscsiadm (rw)
      /sys from sys (rw)
      /usr_local from usrlocal (rw)
      /var/lib/iscsi from var-lib-iscsi (rw)
      /var/lib/kubelet from pods-mount-dir (rw)
      /var/log from log-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from hpe-csi-node-sa-token-kx95r (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  registration-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/kubelet/plugins_registry
    HostPathType:
  plugin-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/kubelet/plugins/csi.hpe.com
    HostPathType:
  pods-mount-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/kubelet
    HostPathType:
  device-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /dev
    HostPathType:
  log-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /var/log
    HostPathType:
  var-lib-iscsi:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/iscsi
    HostPathType:
  etc-iscsi:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/iscsi
    HostPathType:
  etc-multipath:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/multipath
    HostPathType:
  etc-multipath-conf:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/multipath.conf
    HostPathType:  FileOrCreate
  etc-redhat-release:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/redhat-release
    HostPathType:  FileOrCreate
  etc-os-release:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/os-release
    HostPathType:  FileOrCreate
  etc-hpe-storage-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/hpe-storage
    HostPathType:
  etc-kubernetes:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes
    HostPathType:
  iscsiadm:
    Type:          HostPath (bare host directory volume)
    Path:          /sbin/iscsiadm
    HostPathType:  FileOrCreate
  runlock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/lock
    HostPathType:
  lib-modules:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/modules
    HostPathType:
  runsystemd:
    Type:          HostPath (bare host directory volume)
    Path:          /run/systemd
    HostPathType:
  libsystemd:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/systemd/system
    HostPathType:
  usrlocal:
    Type:          HostPath (bare host directory volume)
    Path:          /usr
    HostPathType:
  sys:
    Type:          HostPath (bare host directory volume)
    Path:          /sys
    HostPathType:
  linux-config-file:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      hpe-linux-config
    Optional:  false
  hpe-csi-node-sa-token-kx95r:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  hpe-csi-node-sa-token-kx95r
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/disk-pressure:NoSchedule
                 node.kubernetes.io/memory-pressure:NoSchedule
                 node.kubernetes.io/network-unavailable:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute
                 node.kubernetes.io/pid-pressure:NoSchedule
                 node.kubernetes.io/unreachable:NoExecute
                 node.kubernetes.io/unschedulable:NoSchedule
Events:
  Type     Reason       Age                        From                                             Message
  ----     ------       ----                       ----                                             -------
  Warning  FailedMount  <invalid> (x667 over 20h)  kubelet, cssosbe01-196115.in.rdlabs.hpecorp.net  (combined from similar events): Unable to attach or mount volumes: unmounted volumes=[linux-config-file], unattached volumes=[etc-kubernetes log-dir etc-os-release usrlocal pods-mount-dir device-dir lib-modules etc-hpe-storage-dir libsystemd etc-redhat-release plugin-dir registration-dir etc-multipath var-lib-iscsi runsystemd hpe-csi-node-sa-token-kx95r sys iscsiadm linux-config-file runlock etc-iscsi etc-multipath-conf]: timed out waiting for the condition
  Warning  FailedMount  <invalid> (x537 over 21h)  kubelet, cssosbe01-196115.in.rdlabs.hpecorp.net  MountVolume.SetUp failed for volume "linux-config-file" : configmap "hpe-linux-config" not found

```
- In the above example you can see the configmap "hpe-linux-config" is missing due to which the pods is not getting initialized.
- Issue `wget https://github.com/hpe-storage/co-deployments/raw/master/yaml/csi-driver/v1.1.0/hpe-linux-config.yaml` to get the hpe-linux-config.yaml to create the above configmap
- Issue `kubectl create -f hpe-linux-config.yaml`


### Logging levels 
1. Log levels are controlled by the environment variable LOG_LEVEL 
  Source ref: https://github.hpe.com/Ecosystem-Integration/hpe_3par_primera_csp/blob/master/build/hpe-csi-k8s-1.16.yaml#L205
2. Log level `trace` is the most extensive logging
```yaml
 - name: hpe-csi-driver
          image: hpestorage/csi-driver:v1.1.1
          args :
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--flavor=kubernetes"
          env:
            - name: CSI_ENDPOINT
              value: unix:///var/lib/csi/sockets/pluginproxy/csi.sock
            - name: LOG_LEVEL
              value: trace
```

### PVC Creation in "Pending" state.
1. Logon to the worker node where the CSP pod is running
2. vi `/var/log/hpe-3par-primera-csp.log`

Eg. Here the volume creation fails for "Invalid CPG" as shown in the below log.
```
time="2020-02-06T05:08:06-05:00" level=info msg="<<<<<< Get Volume By Name" file="request_handler.go:187"
time="2020-02-06T05:08:06-05:00" level=info msg=">>>>> Create Volume Call " file="request_handler.go:85"
time="2020-02-06T05:08:06-05:00" level=info msg="Create volume request is : %v {\"data\":{\"name\":\"pvc-0647b318-1291-4c93-b77e-1dcbeeb3ccc2\",\"size\":20401094656,\"config\":{\"access_protocol\":\"iscsi\",\"cpg\":\"FC_rq\",\"multi_initiator\":false,\"provisioning_type\":\"tpvv\"}}}\n" file="create_volume_request.go:72"
time="2020-02-06T05:08:06-05:00" level=info msg="Create volume request (after unmarshal): \n &{{pvc-0647b318-1291-4c93-b77e-1dcbeeb3ccc2 20401094656  {FC_rq  tpvv false}}}" file="request_handler.go:94"
time="2020-02-06T05:08:06-05:00" level=info msg="Session key from request header:  0-f980989b2c51e09f427212939181b9e9-7de93b5e" file="request_handler.go:97"
time="2020-02-06T05:08:06-05:00" level=info msg="Array IP received:  15.212.195.246" file="request_handler.go:99"
time="2020-02-06T05:08:06-05:00" level=info msg="<<<<< Create Volume Cmd" file="create_volume_cmd.go:50"
time="2020-02-06T05:08:06-05:00" level=info msg="Create Volume Request: [&{pvc-0647b318-1291-4c93-b77e-1dc %!s(int64=19456)  FC_rq FC_rq %!s(bool=true) %!s(bool=false) %!s(bool=false)}]" file="volume_client.go:16"
time="2020-02-06T05:08:06-05:00" level=info msg="OK: Response status is :  404 Not Found" file="rest_client.go:27"
time="2020-02-06T05:08:06-05:00" level=info msg="Create volumes response: {\"code\":15,\"desc\":\"CPG does not exist\"}" file="client.go:73"
time="2020-02-06T05:08:06-05:00" level=info msg="Error creating volume:  404 Not Found" file="create_volume_cmd.go:58"
time="2020-02-06T05:08:06-05:00" level=info msg=">>>>> Create Volume Cmd" file="create_volume_cmd.go:79"
time="2020-02-06T05:08:06-05:00" level=info msg=" <<<<< Create Volume Call" file="request_handler.go:114"

```

### Known Limitations
1. If StorageClass (SC), Persistent Volume Claim (PVC), POD definitions are present in same YAML file, and if `kubectl create -f <file>.yaml`
is done, there is a possibility of StorageClass or PVC being deleted before the POD is deleted. And this will
lead to accidental deletion of SC/PVC. It's recommented to get the individual objects like `kubectl get <object>` and issue `kubectl delete <object>`

2. Array users (in both 3PAR and Primera) should have super/edit privileges to allow these users to do host creation.
3. Openshift 4.2 runs on Kubernetes 1.14.x due to which there is no support for volume expansion, clone / snapshot capability.
4. Due to RWO (constraint) in `accessMode` of PVC, the block volume associated with the PVC can only be used exclusively on a single worker node of kubernetes
5. Clone volume creation requires the user to provide the same size as that of the source volume to be created. For eg. if the source volume is 20GiB, then the clone volume size also should match the same size.
   This operation is performed on the target array as online copy process and there will be asynchronous task created for the same.
   `kubectl describe hpevolumeinfo <pv-name>` can be used for getting the status of the clone operation. 
6. Uninstall of the CSI driver does'nt remove Custom Resource Definitions like HPEVolumeInfo, HPENodeInfo etc.
   Also this operation does'nt delete the user created PVC/PV and SC objects.
7. TODO: Add CHAP related information.
### Responsibilities for Controller/Node/CSP plugin
1. Controller pod is responsible for create volume/delete volume calls. Also calls "Create host", "publish volume" calls via CSP
2. Node plugin is responsible for rescan of the scsi devices (as part of `NodeStageVolume`) and creating a temporary staging location for node
3. Node plugin as initiates `NodePublishVolume` after doing staging process in step 2, and does the bind mount of the staging location to the actual pod mount directory.
4. CSP plugin supplies the REST response for each of the controller/node plugin operation
