# Overview

The HPE COSI Driver for Kubernetes is deployed by using industry standard means, a Helm chart.

[TOC]

## Delivery Vehicles

Helm is currently the only supported delivery vehicle for the deployment of the HPE COSI Driver.

### Helm

[Helm](https://helm.sh) is the package manager for Kubernetes. Software is being delivered in a format designated as a "chart". Helm is a [standalone CLI](https://helm.sh/docs/intro/install/) that interacts with the Kubernetes API server using your `KUBECONFIG` file.

The official [Helm chart](https://github.com/hpe-storage/co-deployments/tree/master/helm) for the HPE COSI Driver for Kubernetes is hosted on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver). The chart supports Helm 3 from version 1.0.0 of the HPE COSI Driver. In an effort to avoid duplicate documentation, please see the chart for instructions on how to deploy the COSI driver using Helm.

- Go to the chart on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver).

## Add an HPE Storage Backend

Once the COSI driver is deployed, you must create a `Secret` with the following details before you can use the [COSI API resources](using.md).

### Secret Parameters

All parameters are mandatory and described below.

| Parameter           | Description |
| ------------------- | ------------|
| accessKey           | The access key of the S3 user with bucket creation, bucket-tagging and deletion permissions.
| secretKey           | The secret key for the S3 user who has bucket creation, bucket-tagging and deletion permissions.
| endpoint            | The S3 frontend network DNS subdomains address of the backend object storage system; that is, an HPE Alletra Storage MP X10000 system.
| glcpUserClientId    | The HPE Green Lake API client ID.
| glcpUserSecretKey   | The HPE Green Lake API client secret.
| dsccZone            | The fully qualified domain name (FQDN) of the HPE Data Services Cloud Console (DSCC) zone.
| clusterSerialNumber | The backend storage system cluster serial number.

!!! note
    The Kubernetes compute nodes where the HPE COSI Driver is allowed to run need to be able to access the DSCC zone specified.

Example `Secret` manifest named "hpe-object-backend.yaml":

```yaml fct_label="HPE COSI Driver v1.0.0"
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
  dsccZone: common.cloud.hpe.com
  clusterSerialNumber: 0000000000
```

Create the `Secret`.

```text
kubectl create -f hpe-object-backend.yaml
```

!!! tip "See Also"
    The COSI source code repository contains a parameterized script that can assist in creating a correctly formatted `Secret`. See [github.com/hpe-storage/cosi-driver/scripts/cosi_secret](https://github.com/hpe-storage/cosi-driver/tree/master/scripts/cosi_secret) for more details.

### Creating the S3 user, GLCP user and locating the S3 endpoint, DSCC zone and cluster serial number

1. To create the S3 user:
    * Follow the steps in the HPE documentation to [create an access policy](https://support.hpe.com/hpesc/docDisplay?docId=sd00004219en_us&page=objstr_access_policies_create_dscc.html).
        - Choose _All Buckets_.
        - Add custom actions for the policy: `CreateBucket`, `DeleteBucket`, `PutBucketTagging`.
    * To create the user, refer to the [HPE documentation](https://support.hpe.com/hpesc/docDisplay?docId=sd00004219en_us&page=objstr_users_create_dscc.html) for this purpose and select the access policy created in the previous step.
        - Save the user name and password. These will be used as the S3 access key and S3 secret key respectively in the COSI secret.
2. To create the HPE Green Lake API client ID and secret, refer to the following [HPE documentation](https://developer.hpe.com/blog/api-console-for-data-services-cloud-console/).
3. To locate the S3 endpoint:
    * Log into Data Services Cloud Console.
    * Select _Data Ops Manager_.
    * From the menu on the left, select _Systems_. From the list click on the name of the system you want to use for COSI operations.
    * Click on the _Networking_ tab. Under the _Frontend Network_ section, save the value of the _Network DNS Subdomains_ field.
    * The S3 endpoint can be constructed from the _Network DNS Subdomains_ value by using the format: `http://<Network DNS Subdomains>`.
4. To locate the DSCC zone and cluster serial number of the HPE Alletra Storage MP X10000 system, refer to the [HPE documentation](https://support.hpe.com/hpesc/public/docDisplay?docId=a00120892en_us&page=GUID-616CE4D4-C31A-4BFE-8F41-887C2B0B9046.html).

!!! tip
    In a real world scenario it's more practical to name the `Secret` something that makes sense for the organization. It could be the hostname of the backend or the role it carries; i.e., "hpe-alletra-sanjose-prod".

## Next Steps

Next you need to create [a BucketClass](using.md#configure_a_bucketclass).
