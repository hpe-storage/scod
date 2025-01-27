# Overview

At this point the HPE COSI Driver and HPE Alletra Storage MP X10000 should be installed and configured.

!!! important
    All examples below assumes there's a `Secret` with the necessary credentials to connect to the backend object storage system and HPE Data Services Cloud Console. Learn how to create the `Secret` in the [Deployment section](deployment.md#add_an_hpe_storage_backend).

[TOC]

## Configure a BucketClass

The `BucketClass` defines a set of properties that can be used to configure buckets. The `BucketClass` has a variety of fields that you can configure. An example of `BucketClass` is below. `BucketClasses` are clustered resources normally created by Kubernetes administrators.

```yaml
kind: BucketClass
apiVersion: objectstorage.k8s.io/v1alpha1
metadata:
  name: hpe-standard-object
driverName: cosi.hpe.com
deletionPolicy: Delete
parameters:
  cosiUserSecretName: hpe-object-backend
  cosiUserSecretNamespace: default
```

!!! info "Clarification of `deletionPolicy`"
    The `deletionPolicy` is used to delete a bucket. It can be set to `Delete` or `Retain`, depending on whether the actual bucket in HPE Alletra Storage MP X10000 should be removed or retained. The associated COSI API resources `BucketClaim` and `Bucket` will be deleted regardless of the `deletionPolicy`. The `Bucket` resource can only be removed once all `BucketAccess` resources have been removed.

The `kubectl create` command can be used to create this resource.

```text
kubectl create -f hpe-standard-object.yaml
```

!!! important "Important"
    The [`Secret`](deployment.md#add_an_hpe_storage_backend) should be created before referring to it in the BucketClass.

### Optional BucketClass Parameters

Optional `BucketClass` `.parameters`.

| Parameter                     | String         | Description |
| ----------------------------- | -------------- | ----------- |
| bucketTags                    | Text           | The tags to be applied on a bucket. The key-value pairs must be comma separated. In a tag, the key is required while the value is optional. Example: `.parameters.bucketTags: mytag1=myval1, mytag2, mytag3=myval3`. |

## Create a BucketClaim

The `BucketClaim` resource will be used to create a `Bucket` in COSI according to the properties specified in the `BucketClass` it refers to. Examples of Greenfield and Brownfield provisioning have been provided below. `BucketClaims` are namespaced resources normally created by applications and Kubernetes users.

### Greenfield Bucket Provisioning

You can use the following manifest to create a new bucket on the object storage system.

```yaml
kind: BucketClaim
apiVersion: objectstorage.k8s.io/v1alpha1
metadata:
  name: my-first-bucketclaim
spec:
  bucketClassName: hpe-standard-object
  protocols:
  - s3
```

The `kubectl create` command can be used to create this resource.

```text
kubectl create -f my-first-bucketclaim.yaml
```

The `kubectl create` command will create the `BucketClaim` and `Bucket` on the cluster. Check the `Status` and `Events` field of the resources' description to verify if the bucket was created successfully.

```text
kubectl get bucketclaim,bucket
NAME                               AGE
bucketclaim/my-first-bucketclaim   10d

NAME                                             AGE
bucket/bc199dde004-4f8d-4a20-900b-e5d61e3facb9   10d
```

!!! important "Important"
    The [`Secret`](deployment.md#add_an_hpe_storage_backend) and [`BucketClass`](using.md#configure_a_bucketclass) should be created before provisioning a new bucket.

The `Bucket` created by COSI can be deleted by deleting the associated `BucketClaim` using the `kubectl delete` command.

```text
kubectl delete bucketclaim my-first-bucketclaim
```

### Brownfield Bucket Provisioning

To map an existing bucket to a `BucketClaim` and `Bucket`, refer the following manifests. In the examples provided here the name of the existing bucket is "brownfield-bucket-name".

```yaml
apiVersion: objectstorage.k8s.io/v1alpha1
kind: Bucket
metadata:
  name: my-existing-bucket
spec:
  bucketClaim: {}
  bucketClassName: hpe-standard-object
  deletionPolicy: Delete
  driverName: cosi.hpe.com
  existingBucketID: my-existing-bucket
  parameters:
    cosiUserSecretName: hpe-object-backend
    cosiUserSecretNamespace: default
  protocols:
  - s3
```

!!! note
    A `Bucket` is a clustered resource and is normally created by a Kubernetes administrator.

Create a `BucketClaim` referring to the pre-existing `Bucket`.

```yaml
apiVersion: objectstorage.k8s.io/v1alpha1
kind: BucketClaim
metadata:
  name: my-brownfield-bucketclaim
spec:
  bucketClassName: hpe-standard-object
  existingBucketName: my-existing-bucket
  protocols:
  - s3
```

The `kubectl create` command can be used to create this resource.

```text
kubectl create -f my-existing-bucket.yaml -f my-brownfield-bucketclaim.yaml
```

The above command will create the `BucketClaim` and `Bucket` on the cluster that maps to the existing bucket. Check the `Status` and `Events` field of the resources' description to verify if the mapping was successful.

```text
kubectl get bucketclaim,bucket
NAME                                    AGE
bucketclaim/my-brownfield-bucketclaim   72s

NAME                        AGE
bucket/my-existing-bucket   5s
```

## BucketAccessClass Configuration

The `BucketAccessClass` defines a set of properties that can be used to request bucket-specific access credentials. The `BucketAccessClass` has a variety of fields that can be configured according to the user's preferences. An example has been provided below. `BucketAccessClasses` are clustered resources and normally created by a Kubernetes administrator.

```yaml
kind: BucketAccessClass
apiVersion: objectstorage.k8s.io/v1alpha1
metadata:
  name: hpe-standard-access
driverName: cosi.hpe.com
authenticationType: Key
parameters:
  cosiUserSecretName: hpe-object-backend
  cosiUserSecretNamespace: default
```

!!! important
    The `.parameters.cosiUserSecretName` must reference the same `Secret` used to provision the `BucketClaim` and it must exist prior to creating a `BucketAccessClass` and `BucketClass`. See [add an HPE storage backend for more details](deployment.md#add_an_hpe_storage_backend).

The `kubectl create` command can be used to create this resource.

```text
kubectl create -f hpe-standard-access.yaml
```

## BucketAccess Configuration

The `BucketAccess` resource can be used to create the s3 protocol user credentials to access a particular bucket according to the properties specified in the `BucketAccessClass`. The `BucketAccess` has a variety of fields that can be configured according to the user's preferences. An example has been provided below. `BucketAccess` resources are namespaced and normally created by Kubernetes users to grant `BucketClaim` access to a workload.

```yaml
kind: BucketAccess
apiVersion: objectstorage.k8s.io/v1alpha1
metadata:
  name: my-first-bucketaccess
spec:
  bucketAccessClassName: hpe-standard-access
  credentialsSecretName: my-first-access-secret
  bucketClaimName: my-first-bucketclaim
  protocol: s3
```

The `kubectl create` command can be used to create this resource.

```text
kubectl create -f my-first-bucketaccess.yaml
```

The `kubectl create` will create the `BucketAccess` and a `Secret` in the `default` namespace of the cluster. Check the `Status` and `Events` field of the `BucketAccess` resource's description to verify if the access was granted successfully.

```text
kubectl get bucketaccess,secret
NAME                                  AGE
bucketaccess/my-first-bucketaccess    10d

NAME                              TYPE        DATA   AGE
secret/my-first-access-secret     Opaque      1      10d
```

To inspect the `Secret`, retrieve the content with `kubectl`, decode with `base64` and present with `jq`.

```yaml
kubectl get secret my-first-access-secret -o jsonpath='{.data.BucketInfo}' | base64 --decode | jq
{
  "metadata": {
    "name": "bc-37fd517f-27ac-45fa-b369-57e645967366",
    "creationTimestamp": null
  },
  "spec": {
    "bucketName": "bc199dde004-4f8d-4a20-900b-e5d61e3facb9",
    "authenticationType": "Key",
    "secretS3": {
      "endpoint": "http://192.168.1.100:8080",
      "region": "us-east-1",
      "accessKeyID": "user_ba-37fd517f-27ac-45fa-b369-57e645967366",
      "accessSecretKey": "qLe3DM0Gyoy5I7AMRqRN1Dv2gNWvvq+9EX2Qul+5tWXxgtL5DJ14i+K5UEdh+Orp"
    },
    "secretAzure": null,
    "protocols": [
      "s3"
    ]
  }
}
```

!!! tip
    The `Secret` generated here can be used by any COSI-enabled containerized application on the Kubernetes cluster.

The credentials in this `Secret` can be revoked by deleting the associated `BucketAccess` using the `kubectl delete` command.

```text
kubectl delete bucketaccess my-first-bucketaccess
```

### Attaching an Access Secret to an Object Storage Workload

The `Secret` specified in `credentialsSecretName` of the `BucketAccess` can be mounted as a volume to any containerized application running in a `Pod` on the Kubernetes cluster. The s3 user credentials and other relevant information will be found in the  `mountPath` to access the bucket. The workload running in the `Pod` can read this information to create an s3 client, read and write objects to the bucket using the library functions available in the [`aws` sdk package](https://aws.amazon.com/developer/tools/) for the programming language of your choice. For more details, refer to [official Kubernetes Enhancement Proposals documentation](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/1979-object-storage-support#attaching-bucket-information-to-pods) on using the COSI driver with a `Pod`.

As an example, let us create a `Job` that runs a Python script to put and get an object in the `Bucket` using the mounted `Secret`.

The Python `aws` package `boto3` will be installed in the init container. The Python script is stored in a `ConfigMap` and is mounted to the init container, which will copy it to `PYTHONPATH=/app` where `boto3` is installed. `my-first-access-secret` is mounted as a volume to the main container `my-cosi-app` in the `mountPath: /data/cosi` from where the script will read the bucket-specific s3 user credentials to create the s3 client session.

```yaml
{% include "cosi_driver/examples/using/my-cosi-app.yaml" %}
```

!!! tip
    You may need to set `http_proxy` and `https_proxy` environment variables of the init container to `pip install` the `boto3` package.

Create the `Job` and `ConfigMap`.

```text
kubectl create -f {{ config.site_url }}cosi_driver/examples/using/my-cosi-app.yaml
```

Wait for the `Job` to be completed.

```text fct_label="Job"
kubectl get job
NAME          COMPLETIONS   DURATION   AGE
my-cosi-app   1/1           15s        2m5s
```

The status of the `Pod` can also be tracked till completion.

```text fct_label="Pod"
kubectl get pods
NAME                READY   STATUS      RESTARTS       AGE
my-cosi-app-tlr7l   0/1     Completed   0             2m9s
```

Once the `Job` has completed, the results of the object operations tested in the script can be found in the logs.

```text
kubectl logs jobs/my-cosi-app
Defaulted container "my-cosi-app" out of: my-cosi-app, install-boto3 (init)
Loading bucket info...
Creating S3 client session...
Creating object in S3 bucket...
Object created
Testing the get-object operation on bucket: bc169822df7-532e-43a2-978d-90631719f78e, object body: Hello, World!
Script completed successfully.
```

## Further Reading

The [official Kubernetes documentation](https://kubernetes.io/blog/2022/09/02/cosi-kubernetes-object-storage-management/) contains comprehensive documentation on how the COSI API resources work together.
