# Introduction

The HPE CSI Driver for Kubernetes includes a Kubernetes Pod Monitor. Specifically it looks for `Pods` with the label `monitored-by: hpe-csi` and has `NodeLost` status set on them. This usually occurs if a node becomes unresponsive or partioned due to a network outage. The Pod Monitor will delete the affected `Pod` and associated HPE CSI Driver `VolumeAttachment` to allow Kubernetes to reschedule the workload on a healthy node.

[TOC]

The Pod Monitor is mandatory and automatically applied for the RWX server `Deployment` managed by the HPE CSI Driver. It may be used for any `Pods` on the Kubernetes cluster to perform a more graceful automatic recovery rather than performing a manual intervention to resurrect stuck `Pods`.

## CSI Driver Parameters

The Pod Monitor is part of the "hpe-csi-controller" `Deployment` served by the "hpe-csi-driver" container. It's by default enabled and the Pod Monitor interval is set to 30 seconds.

Edit the CSI driver deployment to change the interval or disable the Pod Monitor. 

```markdown
kubectl edit -n hpe-storage deploy/hpe-csi-controller
```

The parameters that control the "hpe-csi-driver" are the following:

```markdown
        - --pod-monitor
        - --pod-monitor-interval=30
```

## Pod Inclusion

Enable the Pod Monitor for a single replica `Deployment` by labeling the `Pod` (assumes an existing PVC name "my-pvc" exists).

```markdown
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        monitored-by: hpe-csi
        app: my-app
    spec:
      containers:
      - image: busybox
        name: busybox
        command:
          - "sleep"
          - "4800"
        volumeMounts:
        - mountPath: /data
          name: my-vol
      volumes:
      - name: my-vol
        persistentVolumeClaim:
          claimName: my-pvc
```

## Limitations

* Kubernetes provide automatic recovery for your applications, not high availability. Expect applications to take minutes (up to 8 minutes) to fully recover during a node failure or network partition using the Pod Monitor for `Pods` with `PersistentVolumeClaims`.
