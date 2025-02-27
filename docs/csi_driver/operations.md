# Overview

The documentation in this section illustrates officially HPE supported procedures to perform maintenance tasks on the CSI driver outside the scope of deploying and uninstalling the driver.

[TOC]    

## Migrate Encrypted Volumes

Persistent volumes created with v2.1.1 or below using [volume encryption](using.md#using_volume_encryption), the CSI driver use LUKS2 (WikiPedia: [Linux Unified Key Setup](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup)) and can't expand the `PersistentVolumeClaim`. With v2.2.0 and above, LUKS1 is used and the CSI driver is capable of expanding the `PVC`. 

This procedure migrate (copy) data from LUKS2 to LUKS1 PVCs to allow expansion of the volume. 

!!! note
    It's not a limitation of LUKS2 to not allow expansion but rather how the CSI driver interact with the host.

### Assumptions

These are the assumptions made throughout this procedure.

- Data to be migrated has a good backup to restore to, not just a snapshot.
- HPE CSI Driver for Kubernetes v2.3.0 or later installed.
- Worker nodes with access to the Quay registry and SCOD.
- Access to the commands `kubectl`, `curl`, `jq` and `yq`.
- Cluster privileges to manipulate `PersistentVolumes`.
- None of the commands executed should return errors or have non-zero exit codes.
- Only `ReadWriteOnce` `PVCs` are covered.
- No custom `PVC` annotations.

!!! tip
    There are many different ways to copy `PVCs`. These steps outlines and uses one particular method developed and tested by HPE and similar workflows may be applied with other tools and procedures.

### Prepare the Workload and Persistent Volume Claims

First, identify the `PersistentVolume` to migrate from and set shell variables.

```text
export OLD_SRC_PVC=<insert your existing PVC name here>
export OLD_SRC_PV=$(kubectl get pvc -o json | \
       jq -r ".items[] | \
        select(.metadata.name | \
        test(\"${OLD_SRC_PVC}\"))".spec.volumeName)
```
!!! important
    Ensure these shell variables are set at all times.

In order to copy data out of a `PVC`, the running workload needs to be disassociated with the `PVC`. It's not possible to scale the replicas to zero, the exception being `ReadWriteMany` `PVCs` which could lead to data inconsistency problems. These procedures assumes application consistency by having the workload shut down.

It's out of scope for this procedure to demonstrate how to shut down a particular workload. Ensure there are no `volumeattachments` associated with the `PersistentVolume`.

```text
kubectl get volumeattachment -o json | \
 jq -r ".items[] | \
  select(.spec.source.persistentVolumeName | \
  test(\"${OLD_SRC_PV}\"))".spec.source
```

!!! tip
    For large `volumeMode: Filesystem` `PVCs` where copying data may take days, it's recommended to use the [Optional Workflow with Filesystem Persistent Volume Claims](#optional_workflow_with_filesystem_persistent_volume_claims) that utilizes the `PVC` `dataSource` capability.

#### Create a new Persistent Volume Claim and Update Retain Policies

Create a new `PVC` named "new-pvc" with enough space to host the data from the old source `PVC`.

```text 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: new-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  volumeMode: Filesystem
```

!!! important
    If the source `PVC` is a raw block volume, ensure `volumeMode: Block` is set on the new `PVC`.

Edit and set the shell variables for the newly created `PVC`.

```text
export NEW_DST_PVC_SIZE=32Gi
export NEW_DST_PVC_VOLMODE=Filesystem
export NEW_DST_PVC=new-pvc
export NEW_DST_PV=$(kubectl get pvc -o json | \
       jq -r ".items[] | \
       select(.metadata.name | \
       test(\"${NEW_DST_PVC}\"))".spec.volumeName)
```

!!! hint
    The `PVC` name "new-pvc" is a placeholder name. When the procedure is done, the `PVC` will have its original name restored.

##### Important Validation Steps 

At this point, there should be six shell variables declared. Example:

```text
$ env | grep _PV
NEW_DST_PVC_SIZE=32Gi
NEW_DST_PVC=new-pvc
OLD_SRC_PVC=old-pvc <-- This should be the original name of the PVC
NEW_DST_PVC_VOLMODE=Filesystem
NEW_DST_PV=pvc-ad7a05a9-c410-4c63-b997-51fb9fc473bf
OLD_SRC_PV=pvc-ca7c2f64-641d-4265-90f8-4aed888bd2c5
```

Regardless of the `retainPolicy` set in the `StorageClass`, ensure the `persistentVolumeReclaimPolicy` is set to "Retain" for both `PVs`.

```text
kubectl patch pv/${OLD_SRC_PV} pv/${NEW_DST_PV} \
 -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

!!! caution "Data Loss Warning"
    It's **EXTREMELY** important no errors are returned from the above command. It **WILL** lead to data loss.

Validate the "persistentVolumeReclaimPolicy".

```text
kubectl get pv/${OLD_SRC_PV} pv/${NEW_DST_PV} -o json | \
 jq -r ".items[] | \
  select(.metadata.name)".spec.persistentVolumeReclaimPolicy
```

!!! important
    The above command should output nothing but two lines with the word "Retain" on it.

### Copy Persistent Volume Claim and Reset

In this phase, the data will be copied from the original `PVC` to the new `PVC` with a `Job` submitted to the cluster. Different tools are being used to perform the copy operation, ensure to pick the correct `volumeMode`.

#### PVCs with volumeMode: Filesystem

```text
curl -s {{ config.site_url}}csi_driver/examples/operations/pvc-copy-file.yaml | \
  yq "( select(.spec.template.spec.volumes[] | \
    select(.name == \"src-pv\") | \
    .persistentVolumeClaim.claimName = \"${OLD_SRC_PVC}\")
    " | kubectl apply -f-
```

Wait for the `Job` to complete.

```text 
kubectl get job.batch/pvc-copy-file -w
```

Once the `Job` has completed, validate exit status and log files.

```text
kubectl get job.batch/pvc-copy-file -o jsonpath='{.status.succeeded}'
kubectl logs job.batch/pvc-copy-file
```

Delete the `Job` from the cluster.

```text
kubectl delete job.batch/pvc-copy-file
```

Proceed to [restart the workload](#restart_the_workload).

#### PVCs with volumeMode: Block

```text
curl -s {{ config.site_url}}csi_driver/examples/operations/pvc-copy-block.yaml | \
  yq "( select(.spec.template.spec.volumes[] | \
    select(.name == \"src-pv\") | \
    .persistentVolumeClaim.claimName = \"${OLD_SRC_PVC}\")
    " | kubectl apply -f-
```

Wait for the `Job` to complete.

```text 
kubectl get job.batch/pvc-copy-block -w
```

!!! hint
    Data is copied block for block, verbatim, regardless of how much application data is stored in the block devices.

Once the `Job` has completed, validate exit status and log files.

```text
kubectl get job.batch/pvc-copy-block -o jsonpath='{.status.succeeded}'
kubectl logs job.batch/pvc-copy-block
```

Delete the `Job` from the cluster.

```text
kubectl delete job.batch/pvc-copy-block
```

Proceed to [restart the workload](#restart_the_workload).

### Restart the Workload

This step requires both the old source `PVC` and the new destination `PVC` to be deleted. Once again, ensure the correct `persistentVolumeReclaimPolicy` is set on the `PVs`.

```text
kubectl get pv/${OLD_SRC_PV} pv/${NEW_DST_PV} -o json | \
 jq -r ".items[] | \
  select(.metadata.name)".spec.persistentVolumeReclaimPolicy
```

!!! important
    The above command should output nothing but two lines with the word "Retain" on it, if not revisit [Important Validation Steps](#important_validation_steps) to apply the policy and ensure environment variables are set correctly.

Delete the `PVCs`.

```text
kubectl delete pvc/${OLD_SRC_PVC} pvc/${NEW_DST_PVC}
```

Next, allow the new `PV` to be reclaimed.

```text
kubectl patch pv ${NEW_DST_PV} -p '{"spec":{"claimRef": null }}'
```

Next, create a `PVC` with the old source name and ensure it matches the size of the new destination `PVC`. 

```text
curl -s {{ config.site_url}}csi_driver/examples/operations/pvc-copy.yaml | \
  yq ".spec.volumeName = \"${NEW_DST_PV}\" | \
    .metadata.name = \"${OLD_SRC_PVC}\" | \
    .spec.volumeMode = \"${NEW_DST_PVC_VOLMODE}\" | \
    .spec.resources.requests.storage = \"${NEW_DST_PVC_SIZE}\" \
    " | kubectl apply -f-
```

Verify the new `PVC` is "Bound" to the correct `PV`.

```text
kubectl get pvc/${OLD_SRC_PVC} -o json | \
  jq -r ". | \
    select(.spec.volumeName == \"${NEW_DST_PV}\").metadata.name"
```

If the command is successful, it should output your original `PVC` name.

At this point the original workload should be deployed, verified and resumed. 

Optionally, the old source `PV` may be removed.

```text
kubectl delete pv/${OLD_SRC_PV}
```

### Optional Workflow with Filesystem Persistent Volume Claims

If there's a lot of content (millions of files, terabytes of data) that needs to be transferred in a `volumeMode: Filesystem` `PVC` it's recommended to transfer content incrementally. This is achieved by substituting the "old-pvc" with a `dataSource` clone of the running workload and perform the copy from the clone onto the "new-pvc".

After the first transfer completes, the copy job may be recreated as many times as needed with a fresh clone of "old-pvc" until the downtime window has shrunk to an acceptable duration. For the final transfer, the actual source `PVC` will be used instead of the clone.

This is an example `PVC`.

```text
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: clone-of-pvc
spec:
  dataSource:
    name: this-is-the-current-prod-pvc
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
```

!!! tip
    The capacity of the `dataSource` clone must match the original `PVC`.

Enabling and setting up the CSI snapshotter and related `CRDs` is not necessary but it's recommended to be familiar with using [CSI snapshots](using.md#using_csi_snapshots).

<a name="upgrade_to_v240"></a><a name="upgrade_to_v230"></a><a name="upgrade_to_v220"></a>
## Upgrade NFS Servers 

In the event the CSI driver contains updates to the NFS Server Provisioner, any running NFS server needs to be updated manually. 

### Upgrade to v2.5.2

Any prior deployed NFS servers may be upgraded to v2.5.2.

### Upgrade to v2.5.0

Any prior deployed NFS servers may be upgraded to v2.5.0.

### Upgrade to v2.4.2

No changes to NFS Server Provisioner image between v2.4.1 and v2.4.2.

### Upgrade to v2.4.1

Any prior deployed NFS servers may be upgraded to v2.4.1.

!!! important
    With v2.4.0 and onwards the NFS servers are deployed with default resource limits and in v2.5.0 resource requests were added. Those won't be applied on running NFS servers, only new ones.

#### Assumptions

- HPE CSI Driver or Operator v2.4.1 installed.
- All running NFS servers are running in the "hpe-nfs" `Namespace`.
- Worker nodes with access to the Quay registry and SCOD.
- Access to the commands `kubectl`, `yq` and `curl`.
- Cluster privileges to manipulate resources in the "hpe-nfs" `Namespace`.
- None of the commands executed should return errors or have non-zero exit codes.

!!! seealso
    If NFS `Deployments` are scattered across `Namespaces`, use the [Validation](#validation) steps to find where they reside.

#### Patch Running NFS Servers

When patching the NFS `Deployments`, the `Pods` will restart and cause a pause in I/O for the NFS clients with active mounts. The clients will recover gracefully once the NFS `Pod` is running again.

Patch all NFS `Deployments` with the following.

```text
curl -s {{ config.site_url}}csi_driver/examples/operations/patch-nfs-server-2.5.2.yaml | \
  kubectl patch -n hpe-nfs \
  $(kubectl get deploy -n hpe-nfs -o name) \
  --patch-file=/dev/stdin
```

!!! tip
    If it's desired to patch one NFS `Deployment` at a time, replace the shell substituion with a `Deployment` name.

### Validation

This command will list all "hpe-nfs" `Deployments` across the entire cluster. Each `Deployment` should be using v3.0.6 of the "nfs-provisioner" image after the uprade is complete.

```text
kubectl get deploy -A -o yaml | \
  yq -r '.items[] | [] + { "Namespace": select(.spec.template.spec.containers[].name == "hpe-nfs").metadata.namespace, "Deployment": select(.spec.template.spec.containers[].name == "hpe-nfs").metadata.name, "Image": select(.spec.template.spec.containers[].name == "hpe-nfs").spec.template.spec.containers[].image }'
```

!!! note
    The above line is very long.

## Manual Node Configuration

With the release of HPE CSI Driver v2.4.0 it's possible to completely disable the node conformance and node configuration performed by the CSI node driver at startup. This transfers the responsibilty from the HPE CSI Driver to the Kubernetes cluster administrator to ensure worker nodes boot with a supported configuration.

!!! important
    This feature is mainly for users who require 100% control of the worker nodes.

### Stages of Initialization

There are two stages of initialization the administrator can control through parameters in the Helm chart.

#### disableNodeConformance

The node conformance runs with the entrypoint of the node driver container. The conformance inserts and runs a systemd service on the node that installs all required packages on the node to allow nodes to attach block storage devices and mount NFS exports. It starts all the required services and configure an important udev rule on the worker node.

This flag was intended to allow administrators to run the CSI driver on nodes with an unsupported or unconfigured package manager.

If node conformance needs to be disabled for any reason, these packages and services needs to be installed and running prior to installing the HPE CSI Driver:

- iSCSI (not necessary when using FC)
- Multipath
- XFS programs/utilities
- NFSv4 client

Package names and services vary greatly between different Linux distributions and it's the system administrator's duty to ensure these are available to the HPE CSI Driver.

#### disableNodeConfiguration

When disabling node configuration the CSI node driver will not touch the node at all. Besides indirectly disabling node conformance, all attempts to write configuration files or manipulate services during runtime are disabled. 

### Mandatory Configuration

These steps are **REQUIRED** for disabling either node configuration or conformance.

On each current and future worker node in the cluster:

```text
# Don't let udev automatically scan targets(all luns) on Unit Attention.
# This will prevent udev scanning devices which we are attempting to remove.

if [ -f /lib/udev/rules.d/90-scsi-ua.rules ]; then
    sed -i 's/^[^#]*scan-scsi-target/#&/' /lib/udev/rules.d/90-scsi-ua.rules
    udevadm control --reload-rules
fi
```

### iSCSI Configuration

Skip this step if only Fibre Channel is being used. This step is only required when node configuration is disabled.

#### iscsid.conf

This example is taken from a Rocky Linux 9.2 node with the HPE parameters applied. Certain parameters may differ for other distributions of either iSCSI or the host OS.

!!! note
    The location of this file varies between Linux and iSCSI distributions.

Ensure `iscsid` is stopped.

```text
systemctl stop iscsid
```

[Download]({{ config.site_url }}csi_driver/examples/operations/iscsid.conf): /etc/iscsi/iscsid.conf 

```text
{% include "csi_driver/examples/operations/iscsid.conf" %}```

!!! tip "Pro tip!"
    When nodes are provisioned from some sort of templating system with iSCSI pre-installed, it's notoriously common that nodes are provisioned with identical IQNs. This will lead to device attachment problems that aren't obvious to the user. Make sure each node has a unique IQN.

Ensure `iscsid` is running and enabled:

```text
systemctl enable --now iscsid
```

!!! Seealso
    Some Linux distributions requires the `iscsi_tcp` kernel module to be loaded. Where kernel modules are loaded varies between Linux distributions.

### Multipath Configuration

This step is only required when node configuration is disabled.

#### multipath.conf

The defaults section of the configuration file is merely a preference, make sure to leave the device and blacklist stanzas intact when potentially adding more entries from foreign devices.

!!! note
    The location of this file varies between Linux and iSCSI distributions.

Ensure `multipathd` is stopped.

```text
systemctl stop multipathd
```

[Download]({{ config.site_url }}csi_driver/examples/operations/multipath.conf): /etc/multipath.conf 

```text
{% include "csi_driver/examples/operations/multipath.conf" %}```

Ensure `multipathd` is running and enabled:

```text
systemctl enable --now multipathd
```

### Important Considerations

While both disabling conformance and configuration parameters lends itself to a more predictable behaviour when deploying nodes from templates with less runtime configuration, it's still not a complete solution for having immutable nodes. The CSI node driver creates a unique identity for the node and stores it in `/etc/hpe-storage/node.gob`. This file must persist across reboots and redeployments of the node OS image. Immutable Linux distributions such as CoreOS persist the `/etc` directory, some don't.

<!--
yum install -y iscsi-initiator-utils device-mapper-multipath iscsi-initiator-utils-iscsiuio nfs-utils
-->

## Expose NFS Services Outside of the Kubernetes Cluster

In certain situations it's practical to expose the NFS exports outside the Kubernetes cluster to allow external applications to access data as part of an ETL (Extract, Transform, Load) pipeline or similar.

Since this is an untested feature with questionable security standards, HPE does not recommend using this facility in production at this time. Reach out to your HPE account representative if this is a critical feature for your workloads.

!!! danger
    The exports on the NFS servers does not have any network Access Control Lists (ACL) without root squash. Anyone with an NFS client that can reach the load balancer IP address have full access to the filesystem.

### From ClusterIP to LoadBalancer

The NFS server `Service` must be transformed into a "LoadBalancer".

In this example we'll assume a "RWX" `PersistentVolumeClaim` named "my-pvc-1" and NFS resources deployed in the default `Namespace`, "hpe-nfs".

Retrieve NFS UUID

```text
export UUID=$(kubectl get pvc my-pvc-1 -o jsonpath='{.spec.volumeName}{"\n"}' | awk -Fpvc- '{print $2}')
```

Patch the NFS `Service`:
```text
kubectl patch -n hpe-nfs svc/hpe-nfs-${UUID} -p '{"spec":{"type": "LoadBalancer"}}'
```

The `Service` will be assigned an external IP address by the load balancer deployed in the cluster. If there is no load balancer deployed, a MetalLB example is provided below.

### MetalLB Example

Deploying MetalLB is outside the scope of this document. In this example, MetalLB was deployed on OpenShift 4.16 (Kubernetes v1.29) using the Operator provided by Red Hat in the "metallb-system" `Namespace`.

Determine the IP address range that will be assigned to the load balancers. In this example, 192.168.1.40 to 192.168.1.60 is being used. Note that the worker nodes in this cluster already have reachable IP addresses in the 192.168.1.0/24 network, which is a requirement.

Create the MetalLB instances, IP address pool and Layer 2 advertisement.

```text
---
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: metallb-system

---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: hpe-nfs-servers
spec:
  protocol: layer2
  addresses:
  - 192.168.1.40-192.168.1.60

---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
   - hpe-nfs-servers
```

Shortly, the external IP address of the NFS `Service` patched in the previous steps should have an IP address assigned.

```text
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S) 
hpe-nfs-UUID   LoadBalancer   172.30.217.203   192.168.1.40    <long list of ports>
```

### Mount the NFS Server from an NFS Client

Mounting the NFS export externally is now possible.

As root:

```text
mount -t nfs4 192.168.1.40:/export /mnt
```

!!! note
    If the NFS server is rescheduled in the Kubernetes cluster, the load balancer IP address follows, and the client will recover and resume IO after a few minutes.

## Apply Custom Images to the Helm Chart and Operator

Container images that comprise the CSI driver can be individually replaced supply a fix, workaround or address a particular Common Vulnerability and Exposure (CVE).

It's preferred to perform these actions while using the Helm chart or Operator. Images may be changed directly in running `Deployments` and `DaemonSets` while the CSI driver is deployed with either YAML manifests or the Helm chart. The Operator will not tolerate runtime changes and the `HPECSIDriver` resource needs to be updated for the change to take.

!!! important
    The examples below demonstrates how to replace the CSI node and controller driver only. HPE may ask to replace any number of images comprising the HPE CSI Driver, such as a CSP or upstream sidecar.

### Helm

Parameters supplied to a Helm can be inserted either on the command-line or using a "values" YAML file. For an overview of parameters and in this case container images that needs to be manipulated, dump the values file for the chart.

```text
helm show values hpe-storage/hpe-csi-driver
```

!!! tip "Clarification"
    The above command will dump the values for the latest chart in the repository. It will not contain any locally installed values. To pull the values of an installed CSI driver chart, use `helm get values -n hpe-storage my-hpe-csi-driver`.

The section of the values file that concerns container image manipulation is `.images`.

#### Via Command-Line

Imagine there's a patch release from engineering to address a particular issue, say "CON-1234" in the CSI driver images.

```text
helm install --create-namespace -n hpe-storage my-hpe-csi-driver \
--set images.csiNodeDriver=quay.io/hpestorage/csi-driver:v0.0.0-CON-1234 \
--set images.csiControllerDriver=quay.io/hpestorage/csi-driver:v0.0.0-CON-1234 \
hpe-storage/hpe-csi-driver
```

#### Via values.yaml

Since the built-in values provide sane defaults, it's only necessary to manipulate the keys and values that are relevant to the change. If there are other changes that are necessary for this particular install, supply those parameters as well.

```yaml
---
images:
  csiNodeDriver: quay.io/hpestorage/csi-driver:v0.0.0-CON-1234
  csiControllerDriver: quay.io/hpestorage/csi-driver:v0.0.0-CON-1234
```

Install the chart with the contents above in a `values.yaml` file:

```text
helm install --create-namespace -nhpe-storage my-hpe-csi-driver \
-f values.yaml \
hpe-storage/hpe-csi-driver
```

!!! note
    These are generic circumstances to illustrate the relevant steps to apply custom parameters. Be aware of the particular parameters the CSI driver has been installed with for your situation.

### Operator

The Operator manages the Helm chart with a `HPECSIDriver` resource in the chosen `Namespace`, usually "hpe-storage". Changes can be made to the `HPECSIDriver` resource during runtime using either "edit" or "patch" commands but it's recommended to manipulate the source YAML file.

Similar to the Helm chart, the `.spec.images` section needs to be manipulated.

```yaml
---
spec:
  images:
    csiNodeDriver: quay.io/hpestorage/csi-driver:v0.0.0-CON-1234
    csiControllerDriver: quay.io/hpestorage/csi-driver:v0.0.0-CON-1234
```

Visit the [Deployment section](deployment.md#upstream_kubernetes_and_others) for instructions on how to apply the `HPECSIDriver` resource.

!!! tip "Good to Know"
    It's recommended to run the CSI driver with the bundled images and only apply changes when instructed by HPE. Customers may replace images as they desire but may need to revert installations when engaging with HPE support.
