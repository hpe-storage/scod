# Overview 

The HPE CSI Driver is deployed by using industry standard means by using either a Helm chart or an Operator. An "advanced install" from object configuration files is provided as reference for partners, OEMs and users wanting to perform customizations and their own packaging or deployment methodoligies.

[TOC]

## Helm

[Helm](https://helm.sh) is the package manager for Kubernetes. Software is being delivered in a format designated a "chart". Helm is a [standalone CLI](https://helm.sh/docs/intro/install/) that interacts with the Kubernetes API server using your `KUBECONFIG` file.

The official Helm chart for the HPE CSI Driver for Kubernetes is hosted on [hub.helm.sh](https://hub.helm.sh/charts/hpe-storage/hpe-csi-driver). The chart supports both Helm 2 and Helm 3. In an effort to avoid duplicate documentation, please see the chart for instructions on how to deploy the CSI driver using Helm.

## Operator

The Operator pattern is based on the idea that software should be instantiated and run with a set of custom controllers in Kubernetes. It creates a native Kubernetes experience for any software running on Kubernetes.

The official HPE CSI Operator for Kubernetes is hosted on [OperatorHub.io](https://operatorhub.io/operator/hpe-csi-driver-operator). The CSI Operator images are hosted both on docker.io and officially certified containers on Red Hat Container Catalog. For Red Hat OpenShift Container Platform, please see [Installing Operators from the OperatorHub](https://docs.openshift.com/container-platform/4.3/operators/olm-adding-operators-to-cluster.html). 

Follow the documentation from the respective upstream sources on how to deploy an Operator. Instantiate the `HPECSIDriver` with the required values for `backend`, `serviceName`, `username` and `password` once the Operator is deployed. See "View YAML Example" on [OperatorHub.io](https://operatorhub.io/operator/hpe-csi-driver-operator) for an example Custom Resource Definition.

## Advanced install

This guide is primarily written to accommodate a highly manual installation on upstream Kubernetes or partner OEMs engaged with HPE to bundle the HPE CSI Driver in a custom distribution. Installation steps may vary for different vendors and flavors of Kubernetes. 

The following example walks through deployment of the **latest** CSI driver.

!!! caution "Critical"
    It's highly recommended to use either the Helm chart or Operator to install the HPE CSI Driver for Kubernetes and the associated Container Storage Providers. Only venture down manual installation if your requirements can't be met by the [Helm chart](deployment.md#helm) or [Operator](deployment.md#operator).


### Manual CSI driver install

This guide assumes using a supported HPE storage backend. Use the tabs in the code blocks to pick which platform being used.

### Create a secret with backend details
Replace the password string (`YWRtaW4=`) with a base64 encoded version of your password and replace the `backend` with the IP address of the CSP backend and save it as `hpe-secret.yaml`:

```yaml fct_label="HPE Nimble Storage"
apiVersion: v1
kind: Secret
metadata:
  name: hpe-secret
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
  name: hpe-secret
  namespace: kube-system
stringData:
  serviceName: hpe3parprimera-csp-svc
  servicePort: "8080"
  backend: 10.10.0.1
  username: 3paradm
data:
  # echo -n "3pardata" | base64
  password: M3BhcmRhdGE=
```

Create the secret using `kubectl`:

```markdown
kubectl create -f hpe-secret.yaml
secret "hpe-secret" created
```

You should now see the `hpe-secret` in the `kube-system` namespace:

```markdown
kubectl -n kube-system get secret/hpe-secret
NAME                  TYPE                                  DATA      AGE
hpe-secret            Opaque                                5         149m
```

Deploy the CSI driver and sidecars for the relevant Kubernetes version.

### Common
These object configration files are common for all versions of Kubernetes.

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
