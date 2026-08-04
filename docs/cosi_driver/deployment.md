# Overview

The HPE COSI Driver for Kubernetes is deployed by using industry standard means, a Helm chart.

[TOC]

## Delivery Vehicles

Helm is currently the only supported delivery vehicle for the deployment of the HPE COSI Driver.

### Helm

[Helm](https://helm.sh) is the package manager for Kubernetes. Software is being delivered in a format designated as a "chart". Helm is a [standalone CLI](https://helm.sh/docs/intro/install/) that interacts with the Kubernetes API server using your `KUBECONFIG` file.

The official [Helm chart](https://github.com/hpe-storage/co-deployments/tree/master/helm/charts/hpe-cosi-driver) for the HPE COSI Driver for Kubernetes is hosted on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver). The chart supports Helm 3 from version 1.0.0 of the HPE COSI Driver. In an effort to avoid duplicate documentation, please see the chart for instructions on how to deploy the COSI driver using Helm.

- Go to the chart on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver).

!!! note "HPE Alletra Storage MP Disconnected Deployments"
    When deploying against an HPE Alletra Storage MP Disconnected with X10000 instance, the `glcpCommonCloud` Helm chart value must be set with the `sso-` prefix before the instance hostname. Note that this `sso-` prefix applies to the `glcpCommonCloud` Helm chart value, and is distinct from the `dscc-api-` prefix used for the `dsccZone` field in the `Secret` (see below). The two prefixes apply to different fields and must not be interchanged.

### Helm for Air-gapped Environments

In the event of deploying the HPE COSI Driver in a secure air-gapped environment, the images used by the Helm chart and the upstream SIG Storage COSI controller need to be mirrored to a private registry.

Establish a working directory on a bastion Linux host that has HTTP access to the Internet, the private registry and the Kubernetes cluster where the COSI driver needs to be installed. The bastion host is assumed to have the `docker`, `helm` and `kubectl` command installed. It's also assumed throughout that the user executing `docker` has logged in to the private registry and that pulling images from the private registry is allowed anonymously by the Kubernetes compute nodes.

Create a working directory and set environment variables referenced throughout the procedure.

```text
mkdir hpe-cosi-driver
cd hpe-cosi-driver
export MY_REGISTRY=registry.enterprise.example.com
export MY_COSI_DRIVER=2.0.0
```

Next, create a list with the COSI driver images.

```text
helm repo add hpe-storage https://hpe-storage.github.io/co-deployments/
helm repo update
helm template hpe-storage/hpe-cosi-driver --version ${MY_COSI_DRIVER} \
| grep 'image:' | awk '{print $2}' | tr -d '"' | sort | uniq > images
```

The upstream SIG Storage COSI controller is deployed separately from the Helm chart and its image needs to be added to the list.

```text
kubectl kustomize "github.com/kubernetes-sigs/container-object-storage-interface//?ref=release-0.2" \
| grep 'image:' | awk '{print $2}' | sort | uniq >> images
```

Pull, tag and push the images to the private registry.

```text
cat images | xargs -n 1 docker pull
awk '{ print $1" "$1 }' images | sed -E -e "s/ quay.io| registry.k8s.io| gcr.io/ ${MY_REGISTRY}/" | xargs -n 2 docker tag
sed -E -e "s/quay.io|registry.k8s.io|gcr.io/${MY_REGISTRY}/" images | xargs -n 1 docker push
```

!!! tip
    Depending on what kind of private registry being used, the base repositories `hpestorage`, `sig-storage` and `k8s-staging-sig-storage` might need to be created and given write access to the user pushing the images.

All images used by the HPE COSI Driver Helm chart are parameterized individually with the fully qualified URL. Create a `values.yaml` file with the mirrored locations.

```yaml
containers:
  cosiDriver:
    image: registry.enterprise.example.com/hpestorage/cosi-driver:v2.0.0
  sideCar:
    image: registry.enterprise.example.com/sig-storage/objectstorage-sidecar:v0.2.2
```

Install the chart with the `values.yaml` file.

```text
helm install my-hpe-cosi-driver hpe-storage/hpe-cosi-driver \
-n default --version ${MY_COSI_DRIVER} \
-f values.yaml
```

!!! note
    If the private registry requires authentication, create a pull `Secret` in the `Namespace` and reference it with the `regSecretName` chart value.

The SIG Storage COSI controller manifests need the image reference replaced before being applied.

```text
kubectl kustomize "github.com/kubernetes-sigs/container-object-storage-interface//?ref=release-0.2" \
| sed -e "s/container-object-storage-system/default/g" \
| sed -E -e "s|gcr.io|${MY_REGISTRY}|" \
| kubectl apply -f -
```

!!! important
    - The SIG Storage COSI controller image is published to a staging registry and the tag encodes a build date and commit rather than a semantic version. Mirror the exact tag emitted by the command above and pin the mirrored copy.
    - If the client running `helm` is in the air-gapped environment as well, the [docs](https://github.com/hpe-storage/co-deployments/tree/master/docs) directory needs to be hosted on a web server in the air-gapped environment, and then use below command instead.
    `helm repo add hpe-storage https://my-web-server.internal/docs`
    - Regardless of the deployment being air-gapped, the Kubernetes compute nodes where the HPE COSI Driver runs need network access to the object storage system S3 endpoint and to the HPE Data Services Cloud Console zone specified in the `Secret`.

## Add an HPE Storage Backend

Once the COSI driver is deployed, you must create a `Secret` with the following details before you can use the [COSI API resources](using.md).

### Secret Parameters

| Parameter           | Applies To                          | Description |
| ------------------- | ----------------------------------- | ------------|
| accessKey           | All                                 | The access key of the S3 user with bucket creation, bucket-tagging and deletion permissions.
| secretKey           | All                                 | The secret key for the S3 user who has bucket creation, bucket-tagging and deletion permissions.
| endpoint            | All                                 | The S3 frontend network DNS subdomains address of the backend object storage system; that is, an HPE Alletra Storage MP X10000 system.
| glcpUserClientId    | All                                 | The HPE Green Lake API client ID.
| glcpUserSecretKey   | All                                 | The HPE Green Lake API client secret.
| dsccZone            | All                                 | The fully qualified domain name (FQDN) of the HPE Data Services Cloud Console zone.
| clusterSerialNumber | All                                 | The backend storage system cluster serial number.
| glcpWorkspaceId     | HPE Alletra Storage MP X10000       | The HPE GreenLake workspace ID.
| onPremCloudCA       | HPE Alletra Storage MP Disconnected with X10000 | A Base64-encoded CA certificate for the HPE Alletra Storage MP Disconnected with X10000 instance. Required when the CA certificate is not present in the cluster's trusted certificate store. If the CA certificate is already available in the cluster's truststore, this parameter can be omitted.

!!! note
    - For HPE Alletra Storage MP Disconnected with X10000 deployments, prefix the `dsccZone` instance hostname with `dscc-api-`.
    - The Kubernetes compute `Nodes` where the HPE COSI Driver is allowed to run need to be able to access the Data Services Cloud Console zone specified.

Example `Secret` manifest for an HPE Alletra Storage MP X10000 deployment:

```yaml fct_label="HPE Alletra Storage MP X10000"
apiVersion: v1
kind: Secret
metadata:
  name: hpe-object-backend
  namespace: default
stringData:
  accessKey: testuser
  secretKey: testkey
  endpoint: http://192.168.1.100:8080
  glcpUserClientId: 00000000-0000-0000-0000-000000000000
  glcpUserSecretKey: 00000000000000000000000000000000
  glcpWorkspaceId: 00000000000000000000000000000000
  dsccZone: us1.data.cloud.hpe.com
  clusterSerialNumber: 0000000000
```

```yaml fct_label="Disconnected"
apiVersion: v1
kind: Secret
metadata:
  name: hpe-object-backend
  namespace: default
stringData:
  accessKey: testuser
  secretKey: testkey
  endpoint: http://192.168.1.100:8080
  glcpUserClientId: 00000000-0000-0000-0000-000000000000
  glcpUserSecretKey: 00000000000000000000000000000000
  dsccZone: dscc-api.example.lab.nimblestorage.com
  clusterSerialNumber: 0000000000
  # Optional: include if the DSCC CA certificate is not in the cluster's truststore
  # onPremCloudCA: <base64-encoded-ca-certificate>
```

Create the `Secret`.

```text
kubectl create -f hpe-object-backend.yaml
```

!!! tip "See Also"
    The COSI source code repository contains a parameterized script that can assist in creating a correctly formatted `Secret`. See [github.com/hpe-storage/cosi-driver/scripts/cosi_secret](https://github.com/hpe-storage/cosi-driver/tree/main/scripts/cosi_secret) for more details.

### Creating and Locating Resources

1. To create the S3 user:
    * Follow the steps in the HPE documentation to [create an access policy](https://support.hpe.com/hpesc/docDisplay?docId=sd00004219en_us&page=objstr_access_policies_create_dscc.html).
        - Choose _All Buckets_.
        - Add custom actions for the policy: `CreateBucket`, `DeleteBucket`, `PutBucketTagging`.
    * To create the user, refer to the [HPE documentation](https://support.hpe.com/hpesc/docDisplay?docId=sd00004219en_us&page=objstr_users_create_dscc.html) for this purpose and select the access policy created in the previous step.
        - Save the user name and password. These will be used as the S3 access key and S3 secret key respectively in the COSI secret.
2. To create the HPE Green Lake API client ID and secret, refer to the following [HPE documentation](https://support.hpe.com/hpesc/public/docDisplay?docId=a00120892en_us&page=GUID-23E6EE78-AAB7-472C-8D16-7169938BE628.html).
3. To locate the Data Services Cloud Console zone FQDN:
    * Log into HPE Data Services Cloud Console.
    * On the _Services_ page, click _My Services_ to view all services available in your workspace.
    * Select the service that your HPE Alletra Storage MP X10000 device is assigned to and click _Launch_.
    * After the service is launched, save the value of the URL from the browser. E.g.: `https://console-us1.data.cloud.hpe.com`.
    * After dropping the prefix `https://console-`, the Data Services Cloud Console zone FQDN value to be used in the `Secret` should have the following format: `us1.data.cloud.hpe.com`.
    * Supported Data Services Cloud Console zone FQDNs as of June 2026 are:
        - us1.data.cloud.hpe.com
        - jp1.data.cloud.hpe.com
        - eu1.data.cloud.hpe.com
        - uk1.data.cloud.hpe.com
        - uae1.data.cloud.hpe.com
    * For the latest list of supported zones, refer to the [HPE documentation](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007065en_us&page=GUID-D6312444-1C1A-434D-A0D7-986DEF6FFCEB.html&docLocale=en_US#ariaid-title1).
4. To locate the S3 endpoint:
    * Log into HPE Data Services Cloud Console.
    * Launch the service that your HPE Alletra Storage MP X10000 device is assigned to.
    * Select _Data Ops Manager_.
    * From the menu on the left, select _Systems_. From the list click on the name of the system you want to use for COSI operations.
    * Click on the _Networking_ tab. Under the _Frontend Network_ section, save the value of the _Network DNS Subdomains_ field.
    * The S3 endpoint can be constructed from the _Network DNS Subdomains_ value by using the format: `http://<Network DNS Subdomains>`.
5. To locate the cluster serial number of the HPE Alletra Storage MP X10000 system, refer to the following [HPE documentation](https://support.hpe.com/hpesc/public/docDisplay?docId=a00120892en_us&page=GUID-616CE4D4-C31A-4BFE-8F41-887C2B0B9046.html).
6. To view the workspace ID and manage the workspace (required from 2.0.0):
    * Log into the HPE Data Services Cloud Console UI.
    * Navigate to _Quick links_ &rarr; _Manage Workspace_.
    * The _Workspace ID_ is displayed on the page and is used as the `glcpWorkspaceId` field in the `Secret`.
    * For more details, refer to the [Manage workspace](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00005271en_us&page=GUID-DD8699BF-D17E-4A1C-863A-AE7ED0AA8C88.html) documentation.
7. To obtain the CA certificate (required for HPE Alletra Storage MP Disconnected with X10000 setups):
    * Retrieve the CA certificate from the HPE Data Services Cloud Console instance being used. For the download steps, refer to the [Downloading your CA certificates](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00005271en_us&page=GUID-F0ADEE0C-7BB5-4010-B290-FF700A6B7878.html) documentation.
    * Encode the certificate in Base64 format and use the resulting value as the `onPremCloudCA` field in the `Secret`.

!!! note
    Steps 6 and 7 are applicable only from the HPE COSI Driver for Kubernetes v2.0.0 onwards. These steps are not applicable to any versions prior.

!!! tip
    In a real world scenario it's more practical to name the `Secret` something that makes sense for the organization. It could be the hostname of the backend or the role it carries; i.e., "hpe-alletra-sanjose-prod".

## Next Steps

Next you need to create [a BucketClass](using.md#configure_a_bucketclass).
