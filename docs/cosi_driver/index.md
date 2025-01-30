# Introduction

A Container Object Storage Interface ([COSI](https://github.com/kubernetes-sigs/container-object-storage-interface-spec)) Driver for HPE Alletra Storage MP X10000. The HPE COSI Driver for Kubernetes allows you to use the HPE Object Storage Provider (OSP) to perform bucket management operations on storage resources. The COSI architecture allows you to integrate HPE Alletra Storage MP X10000 Object Storage with a COSI-compatible containerized application running on the Kubernetes cluster. The COSI driver follows the gRPC [specification](https://github.com/kubernetes-sigs/container-object-storage-interface/blob/main/proto/cosi.proto) provided by Kubernetes.

![HPE COSI Driver for Kubernetes architecture](img/cosi_driver_architecture-1.0.0.png)

!!! tip
    The HPE COSI Driver for Kubernetes is vendor-specific and works only with the HPE Alletra Storage MP X10000 OSP.

## Table of Contents

[TOC]

## Features and Capabilities

Below is the official table for COSI features that HPE has officially tested and validated against the [platform matrix](#compatibility_and_support).

| Feature                                | K8s maturity | Since K8s version | HPE COSI Driver |
|----------------------------------------|--------------|-------------------|-----------------|
| Bucket Creation                        | Alpha        | 1.25              | 1.0.0           |
| Bucket Deletion                        | Alpha        | 1.25              | 1.0.0           |
| Bucket Tagging                         | Alpha        | 1.25              | 1.0.0           |
| Granting Bucket Access                 | Alpha        | 1.25              | 1.0.0           |
| Revoking Bucket Access                 | Alpha        | 1.25              | 1.0.0           |

Refer to the [official table](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) of feature gates in the Kubernetes docs to determine the availability of alpha features. File any issues, questions or feature requests [here](https://github.com/hpe-storage/cosi-driver/issues). You may also join the HPE Slack community to chat with people close to this project on the `#Alletra` and `#Kubernetes` channels. Sign up at [slack.hpedev.io](https://slack.hpedev.io/) and log in at [hpedev.slack.com](https://hpedev.slack.com).

!!! tip
    Familiarize yourself with the basic requirements below for running the COSI driver on your Kubernetes cluster before you install it. HPE strongly recommends you install the COSI driver with a [Helm chart](deployment.md#helm).

## Compatibility and Support

HPE has tested the following combinations and included them as part of the official support services for the first COSI driver release.

<a name="latest_release"></a>
### HPE COSI Driver for Kubernetes v1.0.0

Release highlights:

* Support for Kubernetes v1.25 to v1.31.
* Implementation of bucket creation, configuration (bucket tagging), lifecycle and access management.
* A log collector script that can be used to collect logs from any node.

<table>
  <tr>
    <th>Kubernetes</th>
    <td>v1.25-v1.31</td>
  </tr>
  <tr>
    <th>Helm Chart</th>
    <td><a href="https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver/1.0.0">v1.0.0</a> on ArtifactHub</td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      HPE Alletra Storage MP X10000
    </td>
  </tr>
  <tr>
    <th>HPE Alletra Storage MP X10000 OS</th>
    <td>
      R1
    </td>
  </tr>
  <tr>
    <th>Protocols</th>
    <td>S3</td>
  </tr>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href="https://github.com/hpe-storage/cosi-driver/tree/main/release-notes/v1.0.0.md">v1.0.0</a> on GitHub</td>
  </tr>
</table>

### Release Archive

HPE does not currently have any archived releases of the HPE COSI Driver.

## Known Limitations

* The HPE COSI driver can be deployed only in the `default` namespace due to a [bug](https://github.com/kubernetes-sigs/container-object-storage-interface-provisioner-sidecar/issues/140) in creating events in the COSI API objects when deployed in non-default namespaces.
* Creating `BucketClaim` or `BucketAccess` objects in parallel can cause failures in the COSI driver. A [bug](https://github.com/kubernetes-sigs/container-object-storage-interface-api/issues/101) has been filed to address this issue.
* A warning event is created in the `Bucket` or `BucketAccess` resources when an error occurs, and has a life-span of one hour. During this period, if the error is resolved the Status will show `Bucket Ready: true` or `Access Granted: true` in the `Bucket` or `BucketAccess` respectively, but the warning event will persist till an hour lapses. A [bug](https://github.com/kubernetes-sigs/container-object-storage-interface-api/issues/103) has been raised to resolve this ambiguity.
* Recreation of `BucketClaim` or `BucketAccess` objects doesn't work intermittently, as gRPC request is not sent to the COSI driver. This [pull request](https://github.com/kubernetes-retired/container-object-storage-interface-api/pull/86) will address the issue.
