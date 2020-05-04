# Overview 

The HPE CSI Driver is deployed by using industry standard means, either a Helm chart or an Operator. An "advanced install" from object configuration files is provided as reference for partners, OEMs and users wanting to perform customizations and their own packaging or deployment methodoligies.

[TOC]

## Helm

[Helm](https://helm.sh) is the package manager for Kubernetes. Software is being delivered in a format designated as a "chart". Helm is a [standalone CLI](https://helm.sh/docs/intro/install/) that interacts with the Kubernetes API server using your `KUBECONFIG` file.

The official Helm chart for the HPE CSI Driver for Kubernetes is hosted on [hub.helm.sh](https://hub.helm.sh/charts/hpe-storage/hpe-csi-driver). The chart supports both Helm 2 and Helm 3. In an effort to avoid duplicate documentation, please see the chart for instructions on how to deploy the CSI driver using Helm.

- Go to the chart on [hub.helm.sh](https://hub.helm.sh/charts/hpe-storage/hpe-csi-driver).

## Operator

The [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) is based on the idea that software should be instantiated and run with a set of custom controllers in Kubernetes. It creates a native experience for any software running in Kubernetes.

The official HPE CSI Operator for Kubernetes is hosted on [OperatorHub.io](https://operatorhub.io/operator/hpe-csi-driver-operator). The CSI Operator images are hosted both on docker.io and officially certified containers on Red Hat Container Catalog.

### Red Hat OpenShift Container Platform

The HPE CSI Operator for Kubernetes is a fully certified Operator for OpenShift. There are a few tweaks needed and there's a separate section for OpenShift.

- See [Red Hat OpenShift](../partners/redhat_openshift/index.md) in the partner ecosystem section 

### Upstream Kubernetes and others

Follow the documentation from the respective upstream distributions on how to deploy an Operator. In most cases, the Operator Lifecyle Manager (OLM) needs to be installed separately.

As an example, we'll deploy version `0.14.1` of the OLM to be able to manage the HPE CSI Operator. Familiarize yourself while is the latest stable release on the [OLM GitHub project's release page](https://github.com/operator-framework/operator-lifecycle-manager/releases).

```markdown
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/0.14.1/install.sh | bash -s 0.14.1
```

Install the HPE CSI Operator.

```markdown
kubectl create -f https://operatorhub.io/install/hpe-csi-driver-operator.yaml
```

The Operator will be installed in `my-hpe-csi-driver-operator` namespace. Watch it come up by inspecting the `ClusterServiceVersion` (CSV).

```markdown
kubectl get csv -n my-hpe-csi-driver-operator
```

Next, a `HPECSIDriver` object needs to be instantiated. Create a file named `hpe-csi-operator.yaml` and populate it according to which CSP is being deployed.

```markdown fct_label="HPE Nimble Storage"
apiVersion: storage.hpe.com/v1
kind: HPECSIDriver
metadata:
  name: csi-driver
spec:
  crd:
    nodeInfo:
      create: false
  cspName: nimble-csp
  flavor: kubernetes
  imagePullPolicy: Always
  images:
    csiDriverImage: 'hpestorage/csi-driver:v1.1.0'
    cspImage: 'hpestorage/nimble-csp:v1.1.0'
  logLevel: info
  secret:
    backend: 192.168.1.1
    create: true
    name: nimble-secret
    password: admin
    serviceName: nimble-csp-svc
    servicePort: '8080'
    username: admin
  storageClass:
    allowVolumeExpansion: true
    create: true
    defaultClass: false
    name: hpe-standard
    parameters:
      accessProtocol: iscsi
      fsType: xfs
      volumeDescription: Volume created by the HPE CSI Driver for Kubernetes
```

```markdown fct_label="HPE 3PAR and Primera"
apiVersion: storage.hpe.com/v1
kind: HPECSIDriver
metadata:
  name: csi-driver
spec:
  crd:
    nodeInfo:
      create: false
  cspName: primera3par-csp
  flavor: kubernetes
  imagePullPolicy: Always
  images:
    csiDriverImage: 'hpestorage/csi-driver:v1.1.1'
    cspImage: 'hpestorage/hpe3parprimera-csp:v1.0.0'
  logLevel: info
  secret:
    backend: 10.1.1.1
    create: true
    name: primera3par-secret
    password: 3pardata
    serviceName: primera3par-csp-svc
    servicePort: '8080'
    username: 3paradm
  storageClass:
    allowVolumeExpansion: true
    create: true
    defaultClass: false
    name: hpe-standard
    parameters:
      accessProtocol: iscsi
      fsType: ext4
      cpg: SSD_r6
      provisioning_type: tpvv
```

Create a `HPECSIDriver` with the manifest.

```markdown
kubectl create -f hpe-csi-operator.yaml
```

The CSI driver is now ready for use. Proceed to the [next section to learn about using](using.md) the driver.

## Advanced install

This guide is primarily written to accommodate a highly manual installation on upstream Kubernetes or partner OEMs engaged with HPE to bundle the HPE CSI Driver in a custom distribution. Installation steps may vary for different vendors and flavors of Kubernetes. 

The following example walks through deployment of the **latest** CSI driver.

!!! caution "Critical"
    It's highly recommended to use either the Helm chart or Operator to install the HPE CSI Driver for Kubernetes and the associated Container Storage Providers. Only venture down manual installation if your requirements can't be met by the [Helm chart](deployment.md#helm) or [Operator](deployment.md#operator).


### Manual CSI driver install

This guide assumes using a supported HPE storage backend. Use the tabs in the code blocks to pick which platform being used.

### Create a secret with backend details
Replace the password string (`YWRtaW4=`) with a base64 encoded version of your password and replace the `backend` with the IP address of the CSP backend and save it as `secret.yaml`:

```yaml fct_label="HPE Nimble Storage"
apiVersion: v1
kind: Secret
metadata:
  name: nimble-secret
  namespace: kube-system
stringData:
  serviceName: nimble-csp-svc
  servicePort: "8080"
  backend: 192.168.1.1
  username: admin
data:
  # echo -n "admin" | base64
  password: YWRtaW4=
```

```yaml fct_label="HPE 3PAR and Primera"
apiVersion: v1
kind: Secret
metadata:
  name: primera3par-secret
  namespace: kube-system
stringData:
  serviceName: primera3par-csp-svc
  servicePort: "8080"
  backend: 10.10.0.1
  username: 3paradm
data:
  # echo -n "3pardata" | base64
  password: M3BhcmRhdGE=
```
!!! Note
    If you are deploying 3PAR or Primera and Nimble CSPs in the same cluster, each `secret` name must be unique.

Create the secret using `kubectl`:

```markdown
kubectl create -f secret.yaml
```

You should now see the `Secret` in the `kube-system` namespace:

```markdown fct_label="HPE Nimble Storage"
kubectl -n kube-system get secret/nimble-secret
NAME                     TYPE                                  DATA      AGE
nimble-secret            Opaque                                5         149m
```

```markdown fct_label="HPE 3PAR and Primera"
kubectl -n kube-system get secret/primera3par-secret
NAME                          TYPE                                  DATA      AGE
primera3par-secret            Opaque                                5         147m
```

Deploy the CSI driver and sidecars for the relevant Kubernetes version.

### Common
These object configuration files are common for all versions of Kubernetes.

Worker node IO settings:

```markdown 
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/hpe-linux-config.yaml
```

Container Storage Provider: 

```markdown fct_label="HPE Nimble Storage"
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/nimble-csp.yaml
```

```markdown fct_label="HPE 3PAR and Primera"
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/3par-primera-csp.yaml
```

!!! important
    The above instructions assumes you have an array with a supported platform OS installed. Please see the requirements section of the respective [CSP](../container_storage_provider/index.md).

### Kubernetes 1.13

```markdown
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/hpe-csi-k8s-1.13.yaml
```

### Kubernetes 1.14

```markdown
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/hpe-csi-k8s-1.14.yaml
```

### Kubernetes 1.15

```markdown
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/hpe-csi-k8s-1.15.yaml
```

### Kubernetes 1.16

```markdown
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/hpe-csi-k8s-1.16.yaml
```

### Kubernetes 1.17

```markdown
kubectl create -f https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v1.1.0/hpe-csi-k8s-1.17.yaml
```

Depending on which version being deployed, different API objects gets created.
