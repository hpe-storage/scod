# Introduction

The HPE CSI Driver for Kubernetes includes a Kubernetes Pod Monitor. Specifically it looks for `Pods` with the label `monitored-by: hpe-csi` and has `NodeLost` status set on them. This usually occurs if a node becomes unresponsive or partioned due to a network outage. The Pod Monitor will delete the affected `Pod` and associated HPE CSI Driver `VolumeAttachment` to allow Kubernetes to reschedule the workload on a healthy node.

The Pod Monitor is mandatory and automatically applied for the RWX server `Deployment` managed by the HPE CSI Driver. It may be used for any `Pods` on the Kubernetes cluster to perform more graceful automatic recovery than perform a manual intervention to resurrect stuck `Pods`.

## CSI driver parameters

The Pod Monitor is part of the "hpe-csi-controller" `Deployment` served by the "hpe-csi-driver" container. It's by default enabled and the Pod Monitor interval is set to 30 seconds.

Edit the CSI driver deployment to change the interval or disable the Pod Monitor. 

```markdown
kubectl edit -n kube-system deploy/hpe-csi-controller
```

The parameters that control the "hpe-csi-driver" are the following:

```markdown
        - --pod-monitor
        - --pod-monitor-interval=30
```

## Pod inclusion

Enable the Pod Monitor for a workload by labeling the `Pod`. 

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    monitored-by: hpe-csi 
spec:
  containers:
  - image: busybox
    name: busybox
    command:
      - "sleep"
      - "300"
    volumeMounts:
    - mountPath: /data
      name: my-vol
  volumes:
  - name: my-vol
    persistentVolumeClaim:
      claimName: my-pvc
```

!!! note "Tech Preview"
    The Pod Monitor is currently in beta and should only be used for testing on non-production workloads.
