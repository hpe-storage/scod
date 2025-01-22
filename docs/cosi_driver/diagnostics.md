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

## Logging

Log files associated with the HPE COSI Driver posts data to the standard output stream and can be accessed using options under [kubectl logs](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/).

If the logs need to be retained for long term, use a standard logging solution for Kubernetes, such as Fluentd. Alternatively, it's advised to use the log collector script, at regular intervals to preserve logs.

### COSI Driver Logs

```text
kubectl logs -f deploy/hpe-objectstorage-provisioner -c hpe-cosi-driver
```

### COSI Sidecar Logs

```text
kubectl logs -f deploy/hpe-objectstorage-provisioner -c hpe-cosi-provisioner-sidecar
```

### COSI Controller Logs

```text
kubectl logs -f deploy/objectstorage-controller
```

### Log Level of Sidecar

You can control the log level for the COSI Sidecar using the `.containers.sideCar.verbosityLevel` field in [`values.yaml`](https://github.com/hpe-storage/cosi-driver/tree/master/helm/values.yaml) of the Helm chart. The values are generally small positive integers.

```yaml
containers:
  sideCar:
    # verbosityLevel specifies the verbosity of the logs that will be printed by the sidecar container
    # Small postive integer values are generally recommended.
    # Ref.: https://pkg.go.dev/k8s.io/klog/v2#V
    verbosityLevel: 5
```

### Log Collector

The log collector script `hpe-logcollector.sh` can be used to collect the logs from any node that has `kubectl` access to the cluster. Please see the script's associated [documentation](https://github.com/hpe-storage/cosi-driver/tree/master/scripts/log_collection) for more details on usage and troubleshooting.

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
