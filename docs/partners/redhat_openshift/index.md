# Overview

HPE and Red Hat have a long standing partnership to provide jointly supported software, platform and services with the absolute best customer experience in the industry.

Red Hat OpenShift uses open source Kubernetes and various other components to deliver a PaaS experience that benefits both developers and operations. This packaged experience differs slighty on how you would deploy and use the HPE volume drivers and this page serves as the authoritive source for all things HPE primary storage and Red Hat OpenShift.

[TOC]

## OpenShift 4

Software deployed on OpenShift 4 follows the [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/). CSI drivers are no exception.

### Certified combinations

Software delivered through the HPE and Red Hat partnership follows a rigorous certification process and only qualify what's listed in the following table.

| Status        | Red Hat OpenShift | HPE CSI Operator | Container Storage Providers |
| ------------- | ----------------- | ---------------- | --------------------------- |
| Certified     | 4.2               | 1.1.0            | Nimble, 3PAR and Primera    |
| Certified     | 4.3               | 1.1.0            | Nimble                      |

Check this table periodically for future releases.

!!! seealso "Pointers"
    Other combinations may work but will not be supported.  
    Both Red Hat Enterprise Linux and Red Hat CoreOS worker nodes are supported.

### Deployment

The HPE CSI Operator for Kubernetes needs to be installed through the interfaces provided by Red Hat. Do not follow the instructions found on OperatorHub.io. 

#### Prerequisites

The HPE CSI Driver needs to run in privileged mode and needs access to host ports, host network and should be able to mount hostPath volumes. Hence, before deploying HPE CSI Operator on OpenShift, please create the following `SecurityContextConstraints` (SCC) to allow the CSI driver to be running with these privileges.

Download the SCC to where you have access to `oc` and the OpenShift cluster:

```markdown
curl -sL https://raw.githubusercontent.com/hpe-storage/co-deployments/master/operators/hpe-csi-operator/deploy/scc.yaml > hpe-csi-scc.yaml
```

Change `my-hpe-csi-driver-operator` to the name of the project (e.g. `hpe-csi-driver` below) where the CSI Operator is being deployed.

```markdown
oc new-project hpe-csi-driver --project-name="HPE CSI Driver for Kubernetes"
sed -i 's/my-hpe-csi-driver-operator/hpe-csi-driver/g' hpe-csi-scc.yaml
```

Deploy the SCC:

```markdown
oc create -f hpe-csi-scc.yaml
securitycontextconstraints.security.openshift.io/hpe-csi-scc created
```

!!! important
    Make note of the project name as it's needed for the Operator deployment in the next steps.

#### OpenShift web console

Once the SCC has been applied to the project, login to the OpenShift web console as `kube:admin` and navigate to **Operators -> OperatorHub**.

![Search for HPE](img/webcon-1.png)
*Search for 'HPE' in the search field.*

![Select the Operator and click Install](img/webcon-2.png)
*Select the HPE CSI Operator and click 'Install'.*

![Select subscribe](img/webcon-3.png)
*In the next pane, click 'Subscribe'.*

![Operator installed](img/webcon-4.png)
*The HPE CSI Operator is now installed.*

![Create a new instance](img/webcon-5.png)
*Click the HPE CSI Operator, in the next pane, click 'Create Instance'.*

![Supply parameters](img/webcon-6.png)
*Configure the instance with the desired values<sup>!</sup>.*

!!! note "Values"
    The required parameters are `.spec.backendType`, `.spec.secret.backend` and the credentials for the backend (`.spec.secret.username` and `.spec.secret.password`).

There should also be a `StorageClass` named `hpe-standard` deployed on the cluster if the default parameters were used. See the CSI driver [using section](../../csi_driver/using.md) for details on how to use peristent storage on Kubernetes wit the HPE CSI Driver.

#### Additional information

At this point the CSI driver is managed like any other Operator on Kubernetes and the life-cycle management capabilities may be exlored further in the [official Red Hat OpenShift documentation](https://docs.openshift.com/container-platform/4.3/operators/olm-what-operators-are.html).

## OpenShift 3

Customers still using OpenShift 3 may use any of the [legacy FlexVolume drivers](../../flexvolume_driver) for managing persistent storage.
