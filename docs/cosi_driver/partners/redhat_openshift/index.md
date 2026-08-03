# Overview
HPE and Red Hat have a long standing partnership to provide jointly supported software, platform and services with the absolute best customer experience in the industry.

Red Hat OpenShift uses open source Kubernetes and various other components to deliver a PaaS experience that benefits both developers and operations. This page serves as the authoritative source for deploying the HPE COSI Driver for Kubernetes on Red Hat OpenShift.

[TOC]

## OpenShift 4

The HPE COSI Driver is deployed using the [Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver) published on Artifact Hub, alongside the upstream SIG Storage Container Object Storage Interface (COSI) controller and `CRDs`.

!!! important
    Container Object Storage Interface (COSI) is a Kubernetes SIG Storage project and the `objectstorage.k8s.io` API is at `v1alpha1`. The API is subject to change between releases. Evaluate accordingly before using in production.

### Tested combinations

| Status        | Red Hat OpenShift | HPE COSI Driver | SIG Storage COSI | Container Storage Providers |
| ------------- | ----------------- | --------------- | ---------------- | --------------------------- |
| Supported     | 4.21              | 2.0.0           | release-0.2      | Alletra Storage MP X10000   |
| Supported     | 4.20              | 2.0.0           | release-0.2      | Alletra Storage MP X10000   |
| Supported     | 4.19              | 2.0.0           | release-0.2      | Alletra Storage MP X10000   |

<small>
 <br />OpenShift support statements for the HPE COSI Driver are published in the [Compatibility and Support](../../index.md#compatibility_and_support) matrix. This page reflects that matrix.
</small>

!!! seealso "Pointers"
    - Other combinations may work but will not be supported.
    - Both Red Hat Enterprise Linux and Red Hat CoreOS worker nodes are supported.
    - Single Node OpenShift (SNO) works along with a 3 Node HA Cluster.
    - HPE Alletra Storage MP Disconnected with X10000 is supported from HPE COSI Driver v2.0.0.
    - The COSI driver does not require access to block or file storage backends. It communicates with the object storage endpoint over S3 and with HPE Data Services Cloud Console over HTTPS.

### Security model

By default, OpenShift prevents containers from running as root and assigns an arbitrary user ID from the `Namespace` `UID` range. Unlike the HPE CSI Driver, the HPE COSI Driver requires no elevated privileges whatsoever. It does not use host networking, host ports, host paths, or privileged mode.

The chart ships a `Deployment` that is already compliant with the OpenShift `restricted-v2` SCC:

- `spec.securityContext.runAsNonRoot: true` on the `Pod`.
- `allowPrivilegeEscalation: false`, `capabilities.drop: [ALL]` and `seccompProfile.type: RuntimeDefault` on both containers.
- The only volume is an `emptyDir` mounted at `/var/lib/cosi` for the COSI gRPC socket.

**No SCC changes are required.** This was verified on OpenShift 4.20.22, where both workloads run without granting any SCC and without modifying the chart. The chart creates two `ServiceAccounts`, `hpe-cosi-provisioner-sa` for the driver and `hpe-cosi-provisioner-pre-upgrade` for the upgrade hook `Job`.

Two workloads do not declare a `securityContext` of their own, the upstream SIG Storage controller `Deployment` and the chart's pre-upgrade hook `Job`. Neither requires elevated access.

!!! note
    The validation above was performed in the `default` `Namespace`, which OpenShift labels `pod-security.kubernetes.io/enforce: privileged`. That is not a strict test of restricted admission. When deploying into a dedicated project, which defaults to enforcing the `restricted` Pod Security profile, confirm the admitting SCC with:

        oc get pod -n <namespace> -l app.kubernetes.io/name=hpe-cosi-driver -o jsonpath='{.items[*].metadata.annotations.openshift\.io/scc}'

If a `Pod` fails to admit with a message referencing security context constraints, see [SCC troubleshooting](#scc_troubleshooting).

### Limitations

- The `objectstorage.k8s.io` API is `v1alpha1`. `Bucket`, `BucketClaim`, `BucketAccess`, `BucketClass` and `BucketAccessClass` resources may not be portable across COSI releases.
- Only the `s3` protocol is supported.
- There is no OpenShift web console integration for COSI resources. All management is performed with `oc` or the API.
- The HPE COSI Driver is not published as an Operator bundle and therefore cannot be mirrored with `oc-mirror`. See [Disconnected install](#disconnected_install).
- Deleting a `BucketClaim` backed by a `BucketClass` with `deletionPolicy: Delete` removes the bucket and its contents on the backend. There is no undo.
- See the [known limitations](../../index.md#known_limitations) common to all platforms. Notably, creating `BucketClaim` or `BucketAccess` resources in parallel can cause failures, and `Bucket` failure events may only be visible in the "default" `Namespace`.

## Deployment

### Prerequisites

- An OpenShift 4.19 or later cluster with `cluster-admin` privileges.
- Helm v3.11 or later on the workstation running `oc`.
- Network reachability from the cluster pod network to the HPE Alletra Storage MP X10000 S3 endpoint.
- Network reachability from the cluster pod network to the HPE Data Services Cloud Console zone.
- DNS resolution of both endpoints from within the cluster.
- An S3 user with an access policy granting `CreateBucket`, `DeleteBucket` and `PutBucketTagging`.
- HPE GreenLake API client ID and secret, the workspace ID, the Data Services Cloud Console zone FQDN and the array cluster serial number.

!!! seealso "See Also"
    See [Creating and Locating Resources](../../deployment.md#creating_and_locating_resources) for step by step instructions on obtaining each of the values above.

#### Verify backend reachability

Before deploying, confirm the cluster can both resolve and reach the object storage endpoint. This avoids the most common class of deployment failure.

```text
oc run cosi-preflight --rm -it --restart=Never \
  --image=registry.access.redhat.com/ubi9/ubi-minimal -- \
  sh -c 'getent hosts <s3-endpoint> && curl -sS -o /dev/null -w "%{http_code}\n" http://<s3-endpoint>:8080/'
```

An HTTP `403` is the expected result. An unauthenticated S3 request is denied by design, which confirms both DNS and TCP connectivity to a live S3 service. A healthy HPE Alletra Storage MP X10000 answers in a couple of milliseconds with an `AccessDenied` body:

```text
HTTP=403 time=0.002312s
<?xml version="1.0" encoding="UTF-8"?>
<Error><Code>AccessDenied</Code><Message>Access Denied.</Message><Resource>/</Resource><RequestId>18C847208D64DC61</RequestId><HostId>0f124ade-cfc2-4b41-b65b-9eddb3c18b81</HostId></Error>
```

| Result | Meaning |
| ------ | ------- |
| `403` with an `AccessDenied` body | Endpoint healthy and reachable. Proceed. |
| DNS lookup failure | Cluster DNS cannot resolve the endpoint. See [DNS resolution](#dns_resolution). |
| `Connection refused` | Nothing listening on that port. Verify the correct S3 port with the storage administrator. |
| `no route to host` or timeout | Network path blocked. See [Network connectivity](#network_connectivity). |

Repeat the test against the Data Services Cloud Console zone, which the driver contacts for access management.

```text
oc run dscc-preflight --rm -it --restart=Never \
  --image=registry.access.redhat.com/ubi9/ubi-minimal -- \
  curl -sS -o /dev/null -w '%{http_code}\n' https://<dscc-zone>/
```

!!! note
    If a cluster-wide HTTP proxy is configured, exclude the on-premises storage endpoint from proxying. S3 traffic to an on-premises array should never traverse a proxy. See [Proxy configuration](#proxy_configuration).

### Create a project

```text
oc new-project hpe-cosi --display-name="HPE COSI Driver"
```

!!! important
    The examples on this page use the "default" `Namespace` to stay consistent with the rest of the HPE COSI Driver documentation. If a dedicated project is used, substitute the `Namespace` consistently in the `Secret`, the `cosiUserSecretNamespace` parameter of the `BucketClass` and `BucketAccessClass`, the `helm install -n` flag and the `sed` expression used for the SIG Storage manifests.

### Deploy the SIG Storage COSI controller

The COSI `CRDs` and the central object storage controller are provided by Kubernetes SIG Storage and must be deployed before the HPE COSI Driver.

```text
oc kustomize "github.com/kubernetes-sigs/container-object-storage-interface//?ref=release-0.2" \
  | sed -e "s/container-object-storage-system/default/g" \
  | oc apply -f -
```

The `sed` expression replaces the upstream default `Namespace`, `container-object-storage-system`, with the target `Namespace`.

!!! caution "Applying the upstream manifests to `default` modifies the `default` Namespace"
    The upstream kustomization contains a `Namespace` object. After the `sed`, that object is named `default`, so `oc apply` patches the cluster's existing `default` `Namespace` rather than creating a new one. It gains COSI's labels and annotations, including `app.kubernetes.io/name: container-object-storage-interface-controller` and a `kubectl.kubernetes.io/last-applied-configuration` describing the COSI namespace. The accompanying warning about a missing `last-applied-configuration` annotation is a symptom of this, not a harmless cosmetic message.

    On OpenShift the `default` `Namespace` is also labelled `pod-security.kubernetes.io/enforce: privileged`, so workloads placed there are not subject to restricted Pod Security admission. Deploying into a dedicated project avoids both effects and is the recommended approach for anything beyond a lab.

To target a dedicated project instead, substitute its name in the `sed` expression:

```text
oc kustomize "github.com/kubernetes-sigs/container-object-storage-interface//?ref=release-0.2" \
  | sed -e "s/container-object-storage-system/hpe-cosi/g" \
  | oc apply -f -
```

Verify the `CRDs` are registered:

```text
oc get crd | grep objectstorage.k8s.io
bucketaccessclasses.objectstorage.k8s.io                          2026-08-03T06:40:07Z
bucketaccesses.objectstorage.k8s.io                               2026-08-03T06:40:07Z
bucketclaims.objectstorage.k8s.io                                 2026-08-03T06:40:07Z
bucketclasses.objectstorage.k8s.io                                2026-08-03T06:40:07Z
buckets.objectstorage.k8s.io                                      2026-08-03T06:40:07Z
```

All five `CRDs` must be present. The second column is the creation timestamp.

The `namePrefix` in the upstream kustomization results in a `Deployment` named `container-object-storage-controller`. Watch it roll out:

```text
oc rollout status deploy/container-object-storage-controller -n default
```

!!! important "Upgrading the controller"
    During a SIG Storage COSI controller upgrade, existing controller resources are not automatically replaced. This results in duplicate `Deployments` and `Pods`, typically stuck in `ImagePullBackOff` or `CrashLoopBackOff`. Delete the existing controller resources before deploying a new version.

### Install the HPE COSI Driver Helm chart

Add the HPE Helm repository:

```text
helm repo add hpe-storage https://hpe-storage.github.io/co-deployments/
helm repo update
helm search repo hpe-storage/hpe-cosi-driver --versions
```

Install the chart:

```text
helm install -n default my-hpe-cosi-driver hpe-storage/hpe-cosi-driver
```

Monitor the deployment:

```text
oc get pods -n default -w
NAME                                                  READY   STATUS    RESTARTS   AGE
container-object-storage-controller-96d94686b-ztjzf   1/1     Running   0          66s
hpe-cosi-provisioner-558c5c47c6-j259p                 2/2     Running   0          24s
```

The `hpe-cosi-provisioner` `Pod` contains two containers, `hpe-cosi-driver` and `hpe-cosi-provisioner-sidecar`. Both must be `Running` before proceeding.

Confirm the release:

```text
helm ls -n default
```

#### Chart values

Review all available values before installing:

```text
helm show values hpe-storage/hpe-cosi-driver
```

The values below are the ones most likely to require customization on OpenShift. The full table is published in the [chart documentation](https://artifacthub.io/packages/helm/hpe-storage/hpe-cosi-driver).

| Parameter                          | Description                                                    | Default |
| ---------------------------------- | -------------------------------------------------------------- | ------- |
| `accessManagement.proxy`           | Proxy URL used by the driver for HPE GreenLake API access       | `""` |
| `accessManagement.glcpCommonCloud` | HPE GreenLake common cloud URL                                  | `global.api.greenlake.hpe.com` |
| `containers.cosiDriver.image`      | Fully qualified registry path of the COSI driver image          | `quay.io/hpestorage/cosi-driver:v2.0.0` |
| `containers.sideCar.image`         | Fully qualified registry path of the SIG Storage sidecar image  | `registry.k8s.io/sig-storage/objectstorage-sidecar:v0.2.2` |
| `containers.sideCar.verbosityLevel`| `klog` verbosity of the sidecar container                       | `5` |
| `regSecretName`                    | `Secret` holding private registry credentials for the images    | `""` |
| `resources`                        | CPU and memory requests and limits. See the caution below        | `{}` |

!!! caution "Do not set `resources` in chart v2.0.0"
    Setting `resources` to a non-empty value causes the `Deployment` template to emit the `resources` block in the middle of the driver container's `env` list, producing invalid YAML. Leave `resources` at its default `{}` until this is fixed in a later chart release. The failure is client side, so nothing is applied to the cluster and no partial deployment is left behind.

Setting it causes `helm install` and `helm upgrade` to fail with:

```text
Error: YAML parse error on hpe-cosi-driver/templates/deployment.yaml: error converting YAML to JSON: yaml: line 69: did not find expected key
```

!!! note "HPE Alletra Storage MP Disconnected with X10000"
    Set `accessManagement.glcpCommonCloud` with an `sso-` prefix before the instance hostname. This is distinct from the `dscc-api-` prefix used for the `dsccZone` field in the `Secret`. The two prefixes apply to different fields and must not be interchanged. See the [deployment documentation](../../deployment.md#delivery_vehicles).

Example values file:

```
accessManagement:
  proxy: "http://proxy.example.com:8080"
  glcpCommonCloud: "sso-instance.example.com"
```

```text
helm install -n default my-hpe-cosi-driver hpe-storage/hpe-cosi-driver -f cosi-values.yaml
```

To apply a value to an existing release:

```text
helm upgrade -n default my-hpe-cosi-driver hpe-storage/hpe-cosi-driver \
  --reuse-values \
  --set accessManagement.proxy="http://proxy.example.com:8080"
```

Verify what was applied:

```text
helm get values -n default my-hpe-cosi-driver
```

!!! note
    The chart enables a pre-upgrade hook, `preUpgradeHookEnabled`, that deletes the `Deployment` before an upgrade. Expect the provisioner `Pod` to be recreated rather than rolled during `helm upgrade`. The hook runs as a short lived `Job` named `hpe-cosi-provisioner-pre-upgrade` under its own `ServiceAccount` of the same name, and calls the Kubernetes API directly to remove the `Deployment`.

### Add an object storage backend

Create a `Secret` with the S3 credentials and HPE Data Services Cloud Console details. See [Add an HPE Storage Backend](../../deployment.md#add_an_hpe_storage_backend) for the full parameter reference and step-by-step instructions for locating each value.

```text
oc apply -f hpe-object-backend.yaml
```

!!! caution
    Do not commit this manifest to source control with real values. Use a sealed secret, an external secrets operator, or apply it out of band.

### Create a BucketClass

Create a `BucketClass` to define storage properties for provisioned buckets. See [Configure a BucketClass](../../using.md#configure_a_bucketclass) for the full parameter reference.

```text
oc apply -f hpe-standard-object.yaml
```

### Create a BucketClaim

Create a `BucketClaim` to provision a bucket. See [Create a BucketClaim](../../using.md#create_a_bucketclaim) for greenfield and brownfield provisioning examples.

```text
oc apply -f my-first-bucketclaim.yaml
oc get bucketclaim,bucket -n default -w
```

!!! note
    Create `BucketClaim` resources serially. Creating them in parallel is a [known limitation](../../index.md#known_limitations).

### Grant workload access

Create a `BucketAccessClass` and a `BucketAccess` to grant a workload credentials to a provisioned bucket. See [Using](../../using.md) for the full resource reference and credential mounting examples.

```text
oc apply -f hpe-standard-access.yaml
oc apply -f my-first-bucketaccess.yaml
oc get bucketaccess,secret -n default
```

## Troubleshooting

### Driver logs

The `hpe-cosi-provisioner` `Deployment` runs two containers. Specify the container explicitly:

```text
oc logs deploy/hpe-cosi-provisioner -c hpe-cosi-driver -n default --tail=50 -f
```

The SIG Storage sidecar logs are useful when claims are not being picked up at all:

```text
oc logs deploy/hpe-cosi-provisioner -c hpe-cosi-provisioner-sidecar -n default --tail=50 -f
```

The central controller reconciles `BucketClaim` into `Bucket`:

```text
oc logs deploy/container-object-storage-controller -n default --tail=50
```

If the release was installed with a modified `deployment.name` or container names, list them first:

```text
oc get deploy -n default
oc get deploy/hpe-cosi-provisioner -n default -o jsonpath='{.spec.template.spec.containers[*].name}'
```

Increase sidecar verbosity when the driver is never invoked:

```text
helm upgrade -n default my-hpe-cosi-driver hpe-storage/hpe-cosi-driver \
  --reuse-values --set containers.sideCar.verbosityLevel=8
```

!!! tip
    The [log collector script](../../diagnostics.md#log_collector) gathers all COSI logs in one pass and is the artifact to attach to an HPE Support case.

### Inspecting COSI resources

Status and events on the COSI resources identify most failures without reading logs:

```text
oc describe bucketclaim/my-first-bucketclaim -n default
oc describe bucket/<bucket-name>
oc describe bucketaccess/my-first-bucketaccess -n default
```

!!! note
    A warning event persists for one hour even after the underlying error is resolved. Trust `Bucket Ready: true` and `Access Granted: true` in `Status` over a stale event. Events for a `Bucket` failure may only appear in the "default" `Namespace`. Both are [known limitations](../../index.md#known_limitations).

### Interpreting bucket creation failures

The sidecar retries provisioning until it succeeds. The error surfaced in the driver log identifies the failure domain.

| Symptom in the driver log | Failure domain | Action |
| ------------------------- | -------------- | ------ |
| `lookup <fqdn> on 172.30.0.10:53: ...` | Cluster DNS | See [DNS resolution](#dns_resolution) |
| `dial tcp <ip>:<port>: connect: no route to host` | Network path | See [Network connectivity](#network_connectivity) |
| `dial tcp <ip>:<port>: connect: connection refused` | Wrong port, or S3 service down | Confirm the S3 port with the storage administrator |
| `proxyconnect tcp ...` | Proxy misconfiguration | See [Proxy configuration](#proxy_configuration) |
| `x509: certificate signed by unknown authority` | Missing CA trust | Supply `onPremCloudCA` in the `Secret` for Disconnected deployments |
| HTTP `403` with `AccessDenied` | Credentials or S3 access policy | Verify the `Secret` and that the access policy grants `CreateBucket`, `DeleteBucket` and `PutBucketTagging` |
| HTTP `503` with `ServiceUnavailable` | Backend rejected the request | See [Backend 503 errors](#backend_503_errors) |

!!! hint
    Every completed gRPC call is logged with a `grpc.code` and a `grpc.time_ms` field, which together isolate the failure domain quickly. A successful `DriverCreateBucket` completes in a few hundred milliseconds, while `DriverGrantBucketAccess` legitimately takes many seconds because it creates an S3 user and access policy in HPE Data Services Cloud Console. Sub-10ms failures indicate an immediate rejection, such as a refused connection, an ICMP unreachable, or an S3 error returned by the array. Multi-second failures with no HPE Data Services Cloud Console activity in the log indicate a timeout, typically DNS or a silently dropped network path.

### DNS resolution

OpenShift manages CoreDNS through the DNS Operator. **Do not edit the CoreDNS `ConfigMap` directly.** The operator reverts manual changes.

Inspect the current configuration:

```text
oc get dns.operator/default -o yaml
```

To add a forwarder for a zone that cluster DNS cannot resolve:

```yaml
spec:
  servers:
    - name: storage-zone
      zones:
        - storage.example.com
      forwardPlugin:
        upstreams:
          - 10.0.0.53
          - 10.0.0.54
```

Verify from within the cluster:

```text
oc run dnstest --rm -it --restart=Never \
  --image=registry.access.redhat.com/ubi9/ubi-minimal -- \
  getent hosts <s3-endpoint>
```

### Network connectivity

`no route to host` originating from a `Pod` indicates the pod network cannot reach the storage subnet. Work outward to isolate the break:

```text
# From a node
oc debug node/<node-name> -- chroot /host curl -sS -o /dev/null -w '%{http_code}\n' http://<s3-endpoint>:8080/

# From a pod
oc run nettest --rm -it --restart=Never \
  --image=registry.access.redhat.com/ubi9/ubi-minimal -- \
  curl -sS -o /dev/null -w '%{http_code}\n' http://<s3-endpoint>:8080/
```

If the node succeeds but the `Pod` fails, inspect any `NetworkPolicy` or `AdminNetworkPolicy` in the `Namespace` and confirm egress to the storage subnet is permitted. If both fail, the break is upstream of the cluster and should be raised with the network team.

### Proxy configuration

On OpenShift, cluster-wide proxy settings are managed through the `Proxy` resource. Do not configure proxy environment variables on nodes directly.

```text
oc get proxy/cluster -o yaml
```

Ensure the storage subnet and endpoint FQDN are present in `spec.noProxy`:

```text
oc patch proxy/cluster --type=merge \
  -p '{"spec":{"noProxy":"<existing-entries>,.storage.example.com,10.0.0.0/8"}}'
```

!!! caution
    Changing the cluster `Proxy` resource triggers a rollout of cluster operators and, on some releases, a rolling reboot of nodes. Plan accordingly.

The `accessManagement.proxy` chart value is separate. It is passed to the driver as the `PROXY` environment variable and governs the driver's HPE GreenLake API access, not S3 data path traffic to the array.

### Backend 503 errors

An HTTP `503` with an S3 XML error body means the request reached the array and was rejected by it. The driver logs the full response, including a `RequestId` and `HostId`:

```text
<Error><Code>ServiceUnavailable</Code><Message>The request has failed due to a temporary server failure.</Message>...<RequestId>18C838DCB62839E8</RequestId><HostId>cc2fe58e-...</HostId></Error>
```

Common causes, in order of likelihood:

1. **Unsupported bucket feature combination.** `locking` without `versioning: Enabled`, or a `retentionMode` the array is not licensed for, will fail. Retest with a `BucketClass` containing only `cosiUserSecretName` and `cosiUserSecretNamespace`, then add one parameter at a time.
2. **Insufficient permissions.** The S3 user may authenticate but lack an access policy granting `CreateBucket`.
3. **Capacity or quota exhaustion** on the backing storage pool.
4. **Array side service degradation.**

Confirm the S3 service is healthy independently of COSI:

```text
curl -sv http://<s3-endpoint>:8080/ 2>&1 | tail -20
```

A `403 AccessDenied` returned in a few milliseconds confirms the S3 service is up and responsive, isolating the fault to the specific bucket create operation.

!!! hint
    Provide the `RequestId` and `HostId` to the storage administrator or HPE Support. The array's own logs contain the specific failure reason behind the generic `503`.

### SCC troubleshooting

The chart requires no SCC changes. If a `Pod` nonetheless fails to admit, the cause is almost always a cluster policy that restricts the default `restricted-v2` SCC, or a custom `securityContext` override.

```text
oc get events -n default --field-selector reason=FailedCreate
oc describe replicaset -n default -l app.kubernetes.io/name=hpe-cosi-driver
```

Confirm which SCC admitted the running `Pod`:

```text
oc get pod -n default -l app.kubernetes.io/name=hpe-cosi-driver \
  -o jsonpath='{.items[*].metadata.annotations.openshift\.io/scc}'
```

!!! note
    An empty result does not indicate a problem. Pods in a `Namespace` that is exempt from restricted Pod Security admission, which includes the `default` `Namespace` on OpenShift, may carry no `openshift.io/scc` annotation at all. If the `Pod` is `Running`, it was admitted. Run this check in a dedicated project to see a meaningful SCC name.

If an SCC must be granted explicitly, target the chart's `ServiceAccount` and use the least privileged SCC that resolves the issue:

```text
oc adm policy add-scc-to-user restricted-v2 -z hpe-cosi-provisioner-sa -n default
oc rollout restart deploy/hpe-cosi-provisioner -n default
```

!!! caution
    The HPE COSI Driver has no legitimate requirement for the `privileged` or `anyuid` SCCs. Granting them indicates a misdiagnosis.

### Stuck BucketClaim deletion

COSI resources each carry a protection finalizer, verified as `cosi.objectstorage.k8s.io/bucketclaim-protection` on a `BucketClaim`, `cosi.objectstorage.k8s.io/bucket-protection` on a `Bucket` and `cosi.objectstorage.k8s.io/bucketaccess-protection` on a `BucketAccess`. If the backend is unreachable, deletion hangs.

Resolve the underlying connectivity or credential failure first. A `Bucket` also cannot be removed until every `BucketAccess` referencing it is deleted.

```text
oc get bucketclaim,bucket,bucketaccess -A
oc patch bucketclaim/<name> -n default -p '{"metadata":{"finalizers":null}}' --type=merge
```

!!! danger
    Removing a finalizer bypasses backend cleanup. The bucket and the generated S3 user are orphaned on the array and must be removed manually. Only do this when the backend is confirmed unreachable or the bucket has already been deleted out of band.

## Uninstall

Remove workload resources first, in dependency order:

```text
oc delete bucketaccess --all -n default
oc delete bucketclaim --all -n default
oc delete bucketaccessclass,bucketclass --all
```

!!! danger
    A `BucketClass` with `deletionPolicy: Delete` removes buckets and all contained objects on the array. Change the policy to `Retain` before deleting the claims if the data must be preserved.

Uninstall the Helm release:

```text
helm uninstall -n default my-hpe-cosi-driver
```

Remove the SIG Storage COSI controller and `CRDs`:

```text
oc kustomize "github.com/kubernetes-sigs/container-object-storage-interface//?ref=release-0.2" \
  | sed -e "s/container-object-storage-system/default/g" \
  | oc delete -f - --ignore-not-found
```

!!! danger "This deletes the target Namespace"
    The upstream kustomization includes a `Namespace` object, so the `sed` value determines which `Namespace` is deleted. When a dedicated project such as `hpe-cosi` was used, this command **deletes that entire project and everything in it**, not just the COSI resources. Confirm nothing else lives there first.

    When the "default" `Namespace` was used, deletion reports `namespaces "default" is forbidden: this namespace may not be deleted`. That is expected and can be ignored, and all other resources are still removed. The COSI labels and annotations applied to the `default` `Namespace` during install are not reverted and have to be removed by hand if they are unwanted.

Finally, remove the backend `Secret`:

```text
oc delete secret hpe-object-backend -n default
```

## Disconnected install

The HPE COSI Driver is not published as an Operator bundle and therefore cannot be mirrored with `oc-mirror`. Mirror the chart and images manually.

Pull the chart on a connected workstation:

```text
helm pull hpe-storage/hpe-cosi-driver --untar
```

Identify the required images:

```text
helm template hpe-cosi-driver/ | grep 'image:'
```

Mirror both images to the internal registry:

```text
oc image mirror \
  quay.io/hpestorage/cosi-driver:v2.0.0 \
  <internal-registry>/hpestorage/cosi-driver:v2.0.0

oc image mirror \
  registry.k8s.io/sig-storage/objectstorage-sidecar:v0.2.2 \
  <internal-registry>/sig-storage/objectstorage-sidecar:v0.2.2
```

The chart takes fully qualified image references, not a separate repository and tag. Install from the local chart directory with both images overridden:

```text
helm install -n default my-hpe-cosi-driver ./hpe-cosi-driver \
  --set containers.cosiDriver.image=<internal-registry>/hpestorage/cosi-driver:v2.0.0 \
  --set containers.sideCar.image=<internal-registry>/sig-storage/objectstorage-sidecar:v0.2.2
```

If the internal registry requires authentication, create a pull `Secret` in the `Namespace` and reference it:

```text
oc create secret docker-registry internal-registry-creds \
  --docker-server=<internal-registry> \
  --docker-username=<user> --docker-password=<password> -n default

helm upgrade -n default my-hpe-cosi-driver ./hpe-cosi-driver \
  --reuse-values --set regSecretName=internal-registry-creds
```

The SIG Storage controller image must be mirrored as well. Render the manifests, mirror the image, then edit the reference before applying:

```text
oc kustomize "github.com/kubernetes-sigs/container-object-storage-interface//?ref=release-0.2" > cosi-controller.yaml
grep 'image:' cosi-controller.yaml
```

!!! caution
    The SIG Storage `release-0.2` controller is published to the Kubernetes **staging** registry, `gcr.io/k8s-staging-sig-storage`, and the tag is a release candidate build that encodes a date and commit rather than a semantic version. Staging images carry no availability guarantee and may be garbage collected. Mirror the exact tag emitted by the command above rather than assuming a version string, and pin the mirrored copy so a future upstream change cannot break a rebuild.

At the time of writing the controller image resolves to:

```text
gcr.io/k8s-staging-sig-storage/objectstorage-controller:v20250905-controllerv0.2.0-rc1-100-gd904c62
```

Both the S3 endpoint and the HPE Data Services Cloud Console instance must remain reachable from the cluster in a disconnected deployment. For HPE Alletra Storage MP Disconnected with X10000, set `accessManagement.glcpCommonCloud` with the `sso-` prefix and supply `onPremCloudCA` in the `Secret` if the instance CA is not already trusted by the cluster.
