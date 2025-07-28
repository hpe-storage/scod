# Overview
<img src="img/redhat-certified.png" align="right" width="256" hspace="12" vspace="2" />
HPE and Red Hat have a long standing partnership to provide jointly supported software, platform and services with the absolute best customer experience in the industry.

Red Hat OpenShift uses open source Kubernetes and various other components to deliver a PaaS experience that benefits both developers and operations. This packaged experience differs slightly on how you would deploy and use the HPE CSI Driver and this page serves as the authoritative source for all things block based HPE primary storage and Red Hat OpenShift.

[TOC]

## OpenShift 4

Software deployed on OpenShift 4 follows the [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/). CSI drivers are no exception.

### Certified combinations

Software delivered through the HPE and Red Hat partnership follows a [rigorous certification process](https://redhat-connect.gitbook.io/openshift-badges/badges/container-storage-interface-csi-1) and only qualify what's listed as "Certified" in the below table.

| Status                  | Red Hat OpenShift                 | HPE CSI Operator           | Container Storage Providers                      |
| ----------------------- | --------------------------------- | -------------------------- | ------------------------------------------------ |
| Certified               | 4.19                              | 3.0.0                      | [All](../../container_storage_provider/index.md) |
| Certified               | 4.18 EUS<sup>2</sup>              | 2.5.2, 3.0.0               | [All](../../container_storage_provider/index.md) |
| Certified               | 4.17                              | 2.5.2, 3.0.0               | [All](../../container_storage_provider/index.md) |
| Certified               | 4.16 EUS<sup>2</sup>              | 2.5.1, 2.5.2, 3.0.0        | [All](../../container_storage_provider/index.md) |
| Certified               | 4.15                              | 2.4.1, 2.4.2, 2.5.1, 2.5.2, 3.0.0 | [All](../../container_storage_provider/index.md) |
| Certified               | 4.14 EUS<sup>2</sup>              | 2.4.0, 2.4.1, 2.4.2, 2.5.1, 2.5.2, 3.0.0 | [All](../../container_storage_provider/index.md) |
| EOL<sup>1</sup>         | 4.13                              | 2.4.0, 2.4.1, 2.4.2        | [All](../../container_storage_provider/index.md) |
| Certified               | 4.12 EUS<sup>2</sup>              | 2.3.0, 2.4.0, 2.4.1, 2.4.2 | [All](../../container_storage_provider/index.md) |

<small><sup>1</sup> = End of life support per [Red Hat OpenShift Life Cycle Policy](https://access.redhat.com/support/policy/updates/openshift).</small><br />
<small><sup>2</sup> = Red Hat OpenShift [Extended Update Support](https://access.redhat.com/support/policy/updates/openshift-eus).</small></br />
<!--small><sup>3</sup> = Passes the Kubernetes CSI e2e test suite on the listed CSPs using the [unsupported Helm chart install](#unsupported_helm_chart_install) method.</small-->

Check the table above periodically for future releases.

!!! seealso "Pointers"
    - Other combinations may work but will not be supported.
    - Both Red Hat Enterprise Linux and Red Hat CoreOS worker nodes are supported.
    - Instructions on this page only reflect the current stable version of the HPE CSI Operator and OpenShift.
    - **Do not attempt** to boot or image OpenShift Virtualization virtual machines from `StorageClasses` with "nfsResources: 'true'", it's not supported and it won't work. Use "RWX" `volumeMode: Block` `PVCs`.

### Security model

By default, OpenShift prevents containers from running as root. Containers are run using an arbitrarily assigned user ID. Due to these security restrictions, containers that run on Docker and Kubernetes might not run successfully on Red Hat OpenShift without modification.

Users deploying applications that require persistent storage (i.e. through the HPE CSI Driver) will need the appropriate permissions and Security Context Constraints (SCC) to be able to request and manage storage through OpenShift. Modifying container security to work with OpenShift is outside the scope of this document.

For more information on OpenShift security, see [Managing security context constraints](https://docs.openshift.com/container-platform/4.17/authentication/managing-security-context-constraints.html).

!!! note
    If you run into issues writing to persistent volumes provisioned by the HPE CSI Driver under a restricted SCC, add the `fsMode: "0770"` parameter to the `StorageClass` with RWO claims or `fsMode: "0777"` for RWX claims.

### Limitations

Since the CSI Operator only provides "Basic Install" capabilities. The following limitations apply:

- The `ConfigMap` "hpe-linux-config" that controls host configuration is immutable
- The NFS Server Provisioner can not be used with Operators deploying `PersistentVolumeClaims` as part of the installation. See [#295](https://github.com/hpe-storage/csi-driver/issues/295) on GitHub.
- Deploying the NFS Server Provisioner to a `Namespace` other than "hpe-nfs" requires a separate SCC applied to the `Namespace`. See [NFS Server Provisioner Considerations](#nfs_server_provisioner_considerations).

### Deployment

The HPE CSI Operator for OpenShift needs to be installed through the interfaces provided by Red Hat.

!!! tip
    There's a tutorial available on YouTube accessible through the [Video Gallery](../../../learn/video_gallery/index.md#install_the_hpe_csi_operator_for_kubernetes_on_red_hat_openshift) on how to install and use the HPE CSI Operator on Red Hat OpenShift.

#### Upgrading

In situations where the operator needs to be upgraded, follow the prerequisite steps in the Helm chart on Artifact Hub.

- [Upgrading the chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver#upgrading-the-chart)

!!! danger "Automatic Updates"
    Do not under any circumstance enable "Automatic Updates" for the HPE CSI Operator for OpenShift.

Once the steps have been followed for the particular version transition:

- Uninstall the `HPECSIDriver` instance
- Delete the "hpecsidrivers.storage.hpe.com" `CRD`<br />:
  `oc delete crd/hpecsidrivers.storage.hpe.com`
- [Uninstall](#uninstall_the_hpe_csi_operator) the HPE CSI Operator for OpenShift
- Proceed to installation through the [OpenShift Web Console](#openshift_web_console) or [OpenShift CLI](#openshift_cli)
- Reapply the [SCC](#scc) to ensure there hasn't been any changes.

!!! important "Good to know"
    Deleting the `HPECSIDriver` instance and uninstalling the CSI Operator does not affect any running workloads, `PersistentVolumeClaims`, `StorageClasses` or other resources created by the HPE CSI Operator. In-flight operations and new requests will be retried once the new `HPECSIDriver` has been instantiated.

#### Prerequisites

The HPE CSI Driver needs to run in privileged mode and needs access to host ports, host network and should be able to mount hostPath volumes. Hence, before deploying HPE CSI Operator on OpenShift, please create the following `SecurityContextConstraints` (SCC) to allow the CSI driver to be running with these privileges.

```text
oc new-project hpe-storage --display-name="HPE CSI Operator for OpenShift"
```

!!! important
    The rest of this implementation guide assumes the default "hpe-storage" `Namespace`. If a different `Namespace` is desired. Update the `ServiceAccount` `Namespace` in the SCC below.

<div id="scc" />Deploy or [download]({{ config.site_url}}csi_driver/partners/redhat_openshift/examples/scc/hpe-csi-scc.yaml) the SCC:

```text
oc apply -f {{ config.site_url}}csi_driver/partners/redhat_openshift/examples/scc/hpe-csi-scc.yaml
securitycontextconstraints.security.openshift.io/hpe-csi-controller-scc created
securitycontextconstraints.security.openshift.io/hpe-csi-node-scc created
securitycontextconstraints.security.openshift.io/hpe-csi-csp-scc created
securitycontextconstraints.security.openshift.io/hpe-csi-nfs-scc created
```

#### OpenShift web console

Once the SCC has been applied to the project, login to the OpenShift web console as "kubeadmin" and navigate to **Operators -> OperatorHub**.

![Search for HPE](img/webcon-1.png)
*Search for 'HPE CSI' in the search field and select the non-marketplace version.*

![Click Install](img/webcon-2.png)
*Click 'Install'.*

![Click Install](img/webcon-3.png)
*Select the Namespace where the SCC was applied, select 'Manual' Update Approval, click 'Install'.*

![Click Approve](img/webcon-3-1.png)
*Click 'Approve' to finalize installation of the Operator*

![Operator installed](img/webcon-4.png)
*The HPE CSI Operator is now installed, select 'View Operator'.*

![Create a new instance](img/webcon-5.png)
*Click 'Create Instance'.*

![Configure instance](img/webcon-6.png)
*Normally, no customizations are needed, scroll all the way down and click 'Create'.*

By navigating to the Developer view, it should now be possible to inspect the CSI driver and Operator topology.

![Operator Topology](img/webcon-7.png)

The CSI driver is now ready for use. Next, an [HPE storage backend needs to be added](../../deployment.md#add_an_hpe_storage_backend) along with a [`StorageClass`](../../using.md#base_storageclass_parameters).

#### OpenShift CLI

This provides an example Operator deployment using `oc`. If you want to use the web console, proceed to the [previous section](#openshift_web_console).

It's assumed the SCC has been applied to the project and have `kube:admin` privileges. As an example, we'll deploy to the `hpe-storage` project as described in previous steps.

First, an `OperatorGroup` needs to be created.

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: hpe-csi-driver-for-kubernetes
  namespace: hpe-storage
spec:
  targetNamespaces:
  - hpe-storage
```

Next, create a `Subscription` to the Operator.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: hpe-csi-operator
  namespace: hpe-storage
spec:
  channel: stable
  installPlanApproval: Manual
  name: hpe-csi-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
```

Next, approve the installation.

```text
oc -n hpe-storage patch $(oc get installplans -n hpe-storage -o name) -p '{"spec":{"approved":true}}' --type merge
```

The Operator will now be installed on the OpenShift cluster. Before instantiating a CSI driver, watch the roll-out of the Operator.

```text
oc rollout status deploy/hpe-csi-driver-operator -n hpe-storage
Waiting for deployment "hpe-csi-driver-operator" rollout to finish: 0 of 1 updated replicas are available...
deployment "hpe-csi-driver-operator" successfully rolled out
```

The next step is to create a `HPECSIDriver` object.

```yaml fct_label="HPE CSI Operator v3.0.0"
# oc apply -n hpe-storage -f {{ config.site_url }}csi_driver/examples/deployment/hpecsidriver-v3.0.0-sample.yaml
{% include "../../examples/deployment/hpecsidriver-v3.0.0-sample.yaml" %}```

```yaml fct_label="v2.5.2"
# oc apply -n hpe-storage -f {{ config.site_url }}csi_driver/examples/deployment/hpecsidriver-v2.5.2-sample.yaml
{% include "../../examples/deployment/hpecsidriver-v2.5.2-sample.yaml" %}```

```yaml fct_label="v2.5.1"
# oc apply -n hpe-storage -f {{ config.site_url }}csi_driver/examples/deployment/hpecsidriver-v2.5.1-sample.yaml
{% include "../../examples/deployment/hpecsidriver-v2.5.1-sample.yaml" %}```

```yaml fct_label="v2.4.2"
# oc apply -n hpe-storage -f {{ config.site_url }}csi_driver/examples/deployment/hpecsidriver-v2.4.2-sample.yaml
{% include "../../examples/deployment/hpecsidriver-v2.4.2-sample.yaml" %}```

The CSI driver is now ready for use. Next, an [HPE storage backend needs to be added](../../deployment.md#add_an_hpe_storage_backend) along with a [`StorageClass`](../../using.md#base_storageclass_parameters).

#### Additional information

At this point the CSI driver is managed like any other Operator on Kubernetes and the life-cycle management capabilities may be explored further in the [official Red Hat OpenShift documentation](https://docs.openshift.com/container-platform/4.19/operators/index.html).

#### Uninstall the HPE CSI Operator

When uninstalling an operator managed by OLM, a Cluster Admin must decide whether or not to remove the `CustomResourceDefinitions` (CRD), `APIServices`, and resources related to these types owned by the operator. By design, when OLM uninstalls an operator it does not remove any of the operator’s owned `CRDs`, `APIServices`, or `CRs` in order to prevent data loss.

!!! important
    Do not modify or remove these `CRDs` or `APIServices` if you are upgrading or reinstalling the HPE CSI driver in order to prevent data loss.

The following are `CRDs` installed by the HPE CSI driver.

```text
hpecsidrivers.storage.hpe.com
hpenodeinfos.storage.hpe.com
hpereplicationdeviceinfos.storage.hpe.com
hpesnapshotgroupinfos.storage.hpe.com
hpevolumegroupinfos.storage.hpe.com
hpevolumeinfos.storage.hpe.com
snapshotgroupclasses.storage.hpe.com
snapshotgroupcontents.storage.hpe.com
snapshotgroups.storage.hpe.com
volumegroupclasses.storage.hpe.com
volumegroupcontents.storage.hpe.com
volumegroups.storage.hpe.com
```

The following are `APIServices` installed by the HPE CSI driver.

```text
v1.storage.hpe.com
v2.storage.hpe.com
```

Please refer to the OLM Lifecycle Manager documentation on how to safely [Uninstall your operator](https://olm.operatorframework.io/docs/tasks/uninstall-operator/).

## NFS Server Provisioner Considerations

When deploying NFS servers on OpenShift there's currently two things to keep in mind for a successful deployment. Also, be understood with the [Limitations and Considerations for the NFS Server Provisioner](../../using.md#limitations_and_considerations_for_the_nfs_server_provisioner) in general.

### Non-standard hpe-nfs Namespace

If NFS servers are deployed in a different `Namespace` than the default "hpe-nfs" by using the "nfsNamespace" `StorageClass` parameter, the "hpe-csi-nfs-scc" SCC needs to be updated to include the `Namespace` `ServiceAccount`.

This example adds "my-namespace" NFS server `ServiceAccount` to the SCC:

```text
oc patch scc hpe-csi-nfs-scc --type=json -p='[{"op": "add", "path": "/users/-", "value": "system:serviceaccount:my-namespace:hpe-csi-nfs-sa" }]'
```

### Operators Requesting NFS Persistent Volume Claims

Object references in OpenShift are not compatible with the NFS Server Provisioner. If a user deploys an Operator of any kind that creates a NFS server backed `PVC`, the operation will fail. Instead, pre-provision the `PVC` manually for the Operator instance to use.

## Optional NetworkPolicies

If a `NetworkPolicy` is required for security reasons, HPE provides a boiler plate to get started.

- Download the [hpe-csi-driver-network-policy.yaml]({{ config.site_url}}csi_driver/partners/redhat_openshift/examples/network-policy/hpe-csi-driver-network-policy.yaml) manifest

In order to apply the `NetworkPolicies`, a few things needs to be configured.

There's currently one egress policy for the "hpe-storage" `Namespace` and an ingress/egress policy for the default NFS Server Provisioner `Namespace`, "hpe-nfs".

!!! note
    Do not apply the policies without configuring at least the backend(s), otherwise the CSP won't be able to reach the backend(s).

### Backend

Uncomment the particular backend that is serving the OpenShift cluster. Add more backends as needed. Also include any replication targets if applicable. In this example we're using an Alletra 5000 at IP address "192.168.1.67".

```yaml
  # Primera, Alletra 9000 and Alletra Storage MP B10000
  #- to:
  #  - ipBlock:
  #      cidr: 0.0.0.0/0
  #  ports:
  #  - port: 443
  #    protocol: TCP

  # Nimble Storage and Alletra 5000/6000
  - to:
    - ipBlock:
        cidr: 192.168.1.67/32
    ports:
    - port: 443
      protocol: TCP
    - port: 5392
      protocol: TCP

  # 3PAR
  #- to:
  #  - ipBlock:
  #      cidr: 0.0.0.0/0
  #  ports:
  #  - port: 8080
  #    protocol: TCP
  #  - port: 443
  #    protocol: TCP
  #  - port: 22
  #    protocol: TCP
```

### Control-Plane Endpoints

In order for the CSI driver and NFS Server Provisioner to communicate with the OpenShift control-plane, the endpoints needs to be configured. It's possible to leave the endpoints at "0.0.0.0/0" with the caveat that port "6443" is reachable to any arbitrary destination.

Retrieve the endpoints for the cluster with the following:

```text
$ oc get endpoints kubernetes -n default
NAME         ENDPOINTS                                                  AGE
kubernetes   192.168.1.166:6443,192.168.1.190:6443,192.168.1.222:6443   13d
```

!!! note
    Both policies have control-plane endpoints configured.

### Custom NFS Server Provisioner Namespace

By default, the NFS Server Provisioner deploys NFS servers in the "hpe-nfs" `Namespace`. If users are allowed to deploy in their own `Namespaces` or a custom `Namespace` is used, the "hpe-nfs-policy" `NetworkPolicy` needs to be deployed in the `Namespace` accordingly.

## StorageProfile for OpenShift Virtualization Source PVCs

If OpenShift Virtualization is being used and Live Migration is desired for virtual machines `PVCs` cloned from the "openshift-virtualization-os-images" `Namespace`, the `StorageProfile` needs to be updated to "ReadWriteMany".

!!! info
    These steps are not necessary on recent OpenShift EUS (v4.12.11 onwards) releases as the default `StorageProfile` for "csi.hpe.com" has been corrected upstream.

If the default `StorageClass` is named "hpe-standard", issue the following command:

```text
oc edit -n openshift-cnv storageprofile hpe-standard
```

Replace the `spec: {}` with the following:

```yaml
spec:
  claimPropertySets:
  - accessModes:
    - ReadWriteMany
    volumeMode: Block
```

Ensure there are no errors. Recreate the OS images:

```text
oc delete pvc -n openshift-virtualization-os-images --all
```

Inspect the `PVCs` and ensure they are re-created with "RWX":

```text
oc get pvc -n openshift-virtualization-os-images -w
```

!!! hint
    The "accessMode" transformation for block volumes from RWO PVC to RWX clone has been resolved in HPE CSI Driver v2.5.0. Regardless, using source RWX PVs will simplify the workflows for users.

## Live VM migrations for Alletra Storage MP B10000

With HPE CSI Operator for OpenShift v2.4.2 and older there's an issue that prevents live migrations of VMs that has `PVCs` attached that has been clones from an OS image residing on Alletra Storage MP B10000 backends including 3PAR, Primera and Alletra 9000.

Identify the `PVC` that that has been cloned from an OS image. The VM name is "centos7-silver-bedbug-14" in this case.

```text
oc get vm/centos7-silver-bedbug-14 -o jsonpath='{.spec.template.spec.volumes}' | jq
```

In this instance, the `dataVolume` is the same name as the VM. Grab the `PV` name from the `PVC` name.

```text
MY_PV_NAME=$(oc get pvc/centos7-silver-bedbug-14 -o jsonpath='{.spec.volumeName}')
```

Next, patch the `hpevolumeinfo` `CRD`.

```text
oc patch hpevolumeinfo/${MY_PV_NAME} --type=merge --patch '{"spec": {"record": {"MultiInitiator": "true"}}}'
```

The VM is now ready to be migrated.

!!! hint
    If there are multiple `dataVolumes`, each one needs to be patched.

## Unsupported Version of the Operator Install

In the event on older version of the Operator needs to be installed, the bundle can be installed directly by [installing the Operator SDK](https://console.redhat.com/openshift/downloads). Make sure a recent version of the `operator-sdk` binary is available and that no HPE CSI Driver is currently installed on the cluster.

Install a specific version prior and including v2.4.2:

```text
operator-sdk run bundle --timeout 5m -n hpe-storage quay.io/hpestorage/csi-driver-operator-bundle:v2.4.2
```

Install a specific version after and including v2.5.0:

```text
operator-sdk run bundle --security-context-config=restricted --timeout 5m -n hpe-storage quay.io/hpestorage/csi-driver-operator-bundle-ocp:v3.0.0
```

!!! important
    Once the Operator is installed, a `HPECSIDriver` instance needs to be created. Follow the steps using the [web console](#openshift_web_console) or the [CLI](#openshift_cli) to create an instance.

When the unsupported install isn't needed any longer, run:

```text
operator-sdk cleanup -n hpe-storage hpe-csi-operator
```

## Unsupported Helm Chart Install

In the event Red Hat releases a new version of OpenShift between HPE CSI Driver releases or if interest arises to run the HPE CSI Driver on an uncertified version of OpenShift, it's possible to install the CSI driver using the Helm chart instead.

It's not recommended to install the Helm chart unless it's listed as "Field Tested" in the [support matrix](#certified_combinations) above.

!!! tip
    Helm chart install is also only current method to use beta releases of the HPE CSI Driver.

### Steps to install.

- Follow the steps in the [prerequisites](#prerequisites) to apply the `SCC` in the `Namespace` (Project) you wish to install the driver.
- Install the Helm chart with the steps provided on [ArtifactHub](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver). Pay attention to which version combination has been field tested.

!!! caution "Unsupported"
    Understand that this method is not supported by Red Hat and not recommended for production workloads or clusters.
