# Overview

HPE and Red Hat have a long standing partnership to deliver enterprise software on the Red Hat OpenShift platform. The HPE COSI Driver for Kubernetes is supported by HPE on Red Hat OpenShift.

[TOC]

## OpenShift 4

The HPE COSI Driver is deployed on OpenShift with the same [Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver) used on any other Kubernetes distribution. There are no OpenShift specific installation steps.

- Go to the chart on [Artifact Hub](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver).

!!! seealso "See Also"
    - [Deployment](../../deployment.md) for the delivery vehicles and installation instructions.
    - [Add an HPE Storage Backend](../../deployment.md#add_an_hpe_storage_backend) for the `Secret` and its parameters.
    - [Creating and Locating Resources](../../deployment.md#creating_and_locating_resources) for obtaining the S3 and HPE Data Services Cloud Console values.
    - [Using](../../using.md) for `BucketClass`, `BucketClaim`, `BucketAccessClass` and `BucketAccess` configuration.
    - [Diagnostics](../../diagnostics.md) for sanity checks, logging and troubleshooting.

!!! important
    Container Object Storage Interface (COSI) is a Kubernetes SIG Storage project and the `objectstorage.k8s.io` API is at `v1alpha1`. The API is subject to change between releases. Evaluate accordingly before using in production.

### Supported combinations

| Status        | Red Hat OpenShift | Container Storage Providers |
| ------------- | ----------------- | --------------------------- |
| Supported     | 4.21              | Alletra Storage MP X10000   |
| Supported     | 4.20              | Alletra Storage MP X10000   |
| Supported     | 4.19              | Alletra Storage MP X10000   |

<small>
 <br />OpenShift support statements for the HPE COSI Driver are published in the [Compatibility and Support](../../index.md#compatibility_and_support) matrix. This page reflects that matrix.
</small>

!!! seealso "Pointers"
    - Other combinations may work but will not be supported.
    - HPE Alletra Storage MP Disconnected with X10000 is supported from HPE COSI Driver v2.0.0.

### Limitations

For generic known limitations applicable to all platforms, see [Known Limitations](../../index.md#known_limitations).

The following limitations are specific to OpenShift:

- There is no OpenShift web console integration for COSI resources. All management is performed with `oc` or the API.
- The HPE COSI Driver is not published as an Operator bundle and therefore cannot be mirrored with `oc-mirror`. Use the [air-gapped procedure](../../deployment.md#helm_for_air-gapped_environments) on the Deployment page instead.
