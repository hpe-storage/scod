# Introduction

The HPE CSI Driver for Kuberentes includes a Kubernetes worker node monitor. Specifically it looks for `Pods` with the label `monitored-by: hpe-csi`. If a node becomes unresponsive or partioned due to a network outage, the node monitor will delete the affected `Pod` and associated HPE CSI Driver `VolumeAttachment` to allow Kubernetes to reschedule the workload on a healthy node.

The node monitor is mandatory for the RWX server `Deployment` managed by the HPE CSI Driver. It may be used for any `Pods` on the Kubernetes cluster to perform more graceful automatic recovery than perform a manual intervention to resurrect stuck `Pods`.

!!! note "Tech Preview"
    The node monitor is currently in beta and should only be used for testing on non-production workloads.
