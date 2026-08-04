# Introduction

It's recommended to familiarize yourself with inspecting workloads on Kubernetes. This [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods) is very useful, and you should have it readily available.

## Sanity Checks

Once the COSI driver has been deployed using Helm, the list of pods shown below is representative of what a healthy system would look like after installation. If any of the workload deployments lists anything but `Running`, proceed to inspect the logs for the problematic workload.

```text
kubectl get pods
NAME                                             READY   STATUS    RESTARTS   AGE
hpe-cosi-provisioner-559b458788-5jc56            2/2     Running   0          8d
objectstorage-controller-7dff56f8fc-r5tdq        1/1     Running   0          8d
```

Once the COSI API objects `BucketClaim`, `Bucket` and `BucketAccess` have been created in the cluster, use the `kubectl describe` command to check the status and events, which will have information useful to help diagnose any issue. If any of the resources' statuses shows anything but `true` or if there are any warning events, inspect the logs of the COSI driver and sidecar.

Describing a `BucketClaim`.

```text fct_label="BucketClaim"
kubectl describe bucketclaim my-first-bucketclaim
Name:         my-first-bucketclaim
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  objectstorage.k8s.io/v1alpha1
Kind:         BucketClaim
Metadata:
  Creation Timestamp:  2024-09-17T04:34:55Z
  Finalizers:
    cosi.objectstorage.k8s.io/bucketclaim-protection
  Generation:        1
  Resource Version:  110670
  UID:               99dde004-4f8d-4a20-900b-e5d61e3facb9
Spec:
  Bucket Class Name:  hpe-standard-object
  Protocols:
    s3
Status:
  Bucket Name:   bc199dde004-4f8d-4a20-900b-e5d61e3facb9
  Bucket Ready:  true
Events:          <none>
```

Describing a `Bucket`.

```text fct_label="Bucket"
kubectl describe bucket bc199dde004-4f8d-4a20-900b-e5d61e3facb9
Name:         bc199dde004-4f8d-4a20-900b-e5d61e3facb9
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  objectstorage.k8s.io/v1alpha1
Kind:         Bucket
Metadata:
  Creation Timestamp:  2024-09-17T04:34:55Z
  Finalizers:
    cosi.objectstorage.k8s.io/bucket-protection
    cosi.objectstorage.k8s.io/bucketaccess-bucket-protection
  Generation:        1
  Resource Version:  111014
  UID:               e7fb69f6-c20e-4abf-a522-f1ae765b3b6a
Spec:
  Bucket Claim:
    Name:             my-first-bucketclaim
    Namespace:        default
    UID:              99dde004-4f8d-4a20-900b-e5d61e3facb9
  Bucket Class Name:  hpe-standard-object
  Deletion Policy:    Delete
  Driver Name:        cosi.hpe.com
  Parameters:
    Bucket Tags:                 mytag=myvalue, mytag2=, mytag3=myvalue3,
    Cosi User Secret Name:       hpe-object-backend
    Cosi User Secret Namespace:  default
  Protocols:
    s3
Status:
  Bucket ID:     bc199dde004-4f8d-4a20-900b-e5d61e3facb9
  Bucket Ready:  true
Events:          <none>
```

Describing a `BucketAccess`.

```text fct_label="BucketAccess"
kubectl describe bucketaccess hpe-standard-access
Name:         hpe-standard-access
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  objectstorage.k8s.io/v1alpha1
Kind:         BucketAccess
Metadata:
  Creation Timestamp:  2024-09-17T04:38:00Z
  Finalizers:
    cosi.objectstorage.k8s.io/bucketaccess-protection
  Generation:        1
  Resource Version:  111017
  UID:               37fd517f-27ac-45fa-b369-57e645967366
Spec:
  Bucket Access Class Name:  hpe-standard-access
  Bucket Claim Name:         my-first-bucket-claim
  Credentials Secret Name:   my-first-access-secret
  Protocol:                  s3
Status:
  Access Granted:  true
  Account ID:      ba-37fd517f-27ac-45fa-b369-57e645967366
Events:            <none>
```

!!! tip
    It is useful to check that the network connection between the Kubernetes cluster hosting the HPE COSI driver and HPE Data Services Cloud Console, as well as between the HPE Alletra Storage MP X 10000 system and HPE Data Services Cloud Console is intact. A poor network connection is a common cause of failure while creating or deleting `Bucket` and `BucketAccess` resources.

## Logging

Log files associated with the HPE COSI Driver posts data to the standard output stream and can be accessed using options under [kubectl logs](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/).

If the logs need to be retained for long term, use a standard logging solution for Kubernetes, such as Fluentd. Alternatively, it's advised to use the log collector script, at regular intervals to preserve logs.

### Identify the Deployments

Before retrieving logs, list the `Deployments` in the `Namespace` where the HPE COSI Driver is installed to determine the exact `Deployment` names.

```text
kubectl get deployments -n <namespace>
```

Example output:

```text
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
hpe-objectstorage-provisioner    1/1     1            1           5d
objectstorage-controller         1/1     1            1           5d
```

Use the `Deployment` names from the `NAME` column in the `kubectl logs` commands below.

### COSI Driver Logs

```text
kubectl logs -f deploy/hpe-objectstorage-provisioner -c hpe-cosi-driver -n <namespace>
```

### COSI Sidecar Logs

```text
kubectl logs -f deploy/hpe-objectstorage-provisioner -c hpe-cosi-provisioner-sidecar -n <namespace>
```

### COSI Controller Logs

```text
kubectl logs -f deploy/objectstorage-controller -n <namespace>
```

### Log Level of Sidecar

You can control the log level for the COSI Sidecar using the `.containers.sideCar.verbosityLevel` field in [`values.yaml`](https://github.com/hpe-storage/co-deployments/blob/master/helm/values/cosi-driver/v1.0.0/values.yaml) of the Helm chart. The values are generally small positive integers.

```yaml
containers:
  sideCar:
    # Verbosity level: Small postive integer values are generally recommended.
    # Ref.: https://pkg.go.dev/k8s.io/klog/v2#V
    # containers.sideCar.verbosityLevel -- Specifies the verbosity of the logs that will be printed by the sidecar container
    verbosityLevel: 5
```

### Log Collector

The log collector script `hpe-logcollector.sh` can be used to collect the logs from any node that has `kubectl` access to the cluster. Please see the script's associated [documentation](https://github.com/hpe-storage/cosi-driver/tree/main/scripts/log_collection) for more details on usage and troubleshooting.

Download the script and provide execute permissions:

```text
wget https://raw.githubusercontent.com/hpe-storage/cosi-driver/main/scripts/log_collection/hpe-logcollector.sh
chmod +x hpe-logcollector.sh
```

Usage:

```text
./hpe-logcollector.sh -h
Collect HPE storage diagnostic logs using kubectl.

Usage:
     hpe-logcollector.sh [-h|--help] [-n|--namespace NAMESPACE]
Options:
-h|--help                  Print this usage text
-n|--namespace NAMESPACE   Collect logs from HPE COSI Driver Deployment in Namespace
                           NAMESPACE (default: default)
```

## Troubleshooting

### Interpreting Bucket Creation Failures

The COSI sidecar retries provisioning until it succeeds. The error surfaced in the COSI driver log identifies where the request failed.

| Symptom in the driver log | Failure domain | Action |
| ------------------------- | -------------- | ------ |
| `lookup <fqdn> ...` | Name resolution | The cluster cannot resolve the S3 endpoint or the HPE Data Services Cloud Console zone. Verify cluster DNS. |
| `dial tcp <ip>:<port>: connect: no route to host` | Network path | Verify the compute nodes can reach the object storage system and HPE Data Services Cloud Console. |
| `dial tcp <ip>:<port>: connect: connection refused` | Wrong port or service down | Confirm the S3 port with the storage administrator. |
| `x509: certificate signed by unknown authority` | Missing CA trust | Supply `onPremCloudCA` in the `Secret` for HPE Alletra Storage MP Disconnected with X10000 deployments. |
| HTTP `403` with `AccessDenied` | Credentials or access policy | Verify the `Secret` contents and that the S3 user's access policy grants `CreateBucket`, `DeleteBucket` and `PutBucketTagging`. |
| HTTP `503` with `ServiceUnavailable` | Backend rejected the request | See [Backend Errors](#backend_errors). |
| `Invalid bucket parameters: ...` | `BucketClass` parameters | The driver rejected the parameter combination before contacting the array. See [Optional BucketClass Parameters](using.md#optional_bucketclass_parameters). |

!!! hint
    Every completed gRPC call is logged with a `grpc.code` and a `grpc.time_ms` field. A successful `DriverCreateBucket` completes in a few hundred milliseconds, while `DriverGrantBucketAccess` legitimately takes considerably longer as it creates an S3 user and access policy in HPE Data Services Cloud Console. Failures returned in a few milliseconds indicate an immediate rejection, such as a refused connection or an error returned by the object storage system. Failures taking seconds indicate a timeout, typically name resolution or a silently dropped network path.

!!! note
    Bucket parameter validation errors are raised by the driver before any request is sent to the array. Correcting the `BucketClass` is not sufficient on its own, the `BucketClaim` has to be recreated.

### Backend Errors

An HTTP `503` with an S3 XML error body means the request reached the object storage system and was rejected by it. The driver logs the full response, including a `RequestId` and `HostId`.

```text
<Error><Code>ServiceUnavailable</Code><Message>The request has failed due to a temporary server failure.</Message>...<RequestId>18C838DCB62839E8</RequestId><HostId>cc2fe58e-...</HostId></Error>
```

Common causes, in order of likelihood:

1. Unsupported bucket feature combination. Retest with a `BucketClass` containing only `cosiUserSecretName` and `cosiUserSecretNamespace`, then add one parameter at a time.
2. Insufficient permissions. The S3 user may authenticate but lack an access policy granting `CreateBucket`.
3. Capacity or quota exhaustion on the backing storage pool.
4. Backend service degradation.

!!! hint
    Provide the `RequestId` and `HostId` to the storage administrator or HPE Support. The object storage system's own logs contain the specific failure reason behind the generic `503`.

### Stuck Resource Deletion

The COSI API resources each carry a protection finalizer, `cosi.objectstorage.k8s.io/bucketclaim-protection` on a `BucketClaim`, `cosi.objectstorage.k8s.io/bucket-protection` on a `Bucket` and `cosi.objectstorage.k8s.io/bucketaccess-protection` on a `BucketAccess`. If the backend is unreachable, deletion hangs.

Resolve the underlying connectivity or credential failure first. A `Bucket` also cannot be removed until every `BucketAccess` referencing it has been deleted.

```text
kubectl get bucketclaim,bucket,bucketaccess -A
```

!!! danger
    Removing a finalizer bypasses backend cleanup. The bucket and the generated S3 user are orphaned on the object storage system and must be removed manually. Only remove a finalizer when the backend is confirmed unreachable or the bucket has already been deleted out of band.
