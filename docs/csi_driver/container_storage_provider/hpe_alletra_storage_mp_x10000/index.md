# Introduction

HPE Alletra Storage MP X10000 is primarily an object storage platform. Dynamic provisioning of buckets and credentials is provided by the [HPE COSI Driver for Kubernetes](../../../cosi_driver/index.md). Since version 2.0.0.0 of the platform, it also offers NFS. The HPE Alletra Storage MP X10000 CSP offers dynamic provisioning of NFS exports through the HPE CSI Driver for Kubernetes.

[TOC]

## Platform Requirements

 An account with admin privileges is needed to create the CSP client authorization in order to use the API through the HPE CSI Driver backend `Secret`. It's also expected that the frontend data IP range has been provisioned on the platform.

### CSP Client Authorization

In order to create a `Secret` to reference in the `StorageClass`, an API authorization resource needs to be created on the platform. It's expected that a user with "admin" privileges create the resource via the CLI.

Since the CLI is logging all the visible shell activity, the password needs to be pasted into a silent variable we'll use in the next step.

```text
read -p "Paste your password: " -s MY_PASSWORD
```

Next, paste the following command and input (in the same shell).

```text
cat << EOF | vim - -c "wq! ${TEMP}/extoauthclient.yaml"
apiVersion: sc.hpe.com/v1
kind: ExtOAuthClient
metadata:
  name: hpe-csi-driver
  namespace: cm
spec:
  client_name: csp
  client_secret: "${MY_PASSWORD}"
  groups:
    - ext:file-provisioner
EOF
```

!!! hint
    The "client_secret" value needs to follow the platform password rules. Include a number, upper case letter and a special character with a minimum length of eight characters.

Create the authorization.

```text
glsctl create extoauthclient -f ${TEMP}/extoauthclient.yaml
```

Remove the authorization file from the system.

```text
rm -f ${TEMP}/extoauthclient.yaml
```

In the newly created resource, the unique client ID that was generated is going to be used by the backend `Secret` username.

Extract the client ID.

```text
glsctl describe extoauthclient hpe-csi-driver | \
grep hf_client_id | \
awk -F= '{print $2}'
```

In this example, "0123456789abcdef" is the client ID.

Now, create the `Secret` on the Kubernetes cluster where the HPE CSI Driver is installed.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: hpe-backend
  namespace: hpe-storage
stringData:
  serviceName: alletrastoragemp-x10000-nfs-csp-svc
  servicePort: "8080"
  backend: 192.168.1.100:443    # Replace with X10000 management IP/hostname
  username: 0123456789abcdef    # Replace with the generated client ID
  password: my-password-X10000  # Replace with the actual "client_secret"
```

## StorageClass Parameters

The CSP only supports dynamic provisioning of `PersistentVolumes`. No data management such as snapshot or cloning.

| Parameter                      | String  | Description |
| ------------------------------ | ------- | ----------- |
| accessProtocol                 | Text    | Mandatory, set to "nfs". Defaults to "iscsi" when unspecified. |
| accessControlList<sup>1</sup>  | Text    | A comma separated list of [access clients and networks](#access_clients_and_networks). Defaults to "*" which allows any host to mount the export. |

<small><sup>1</sup> = This parameter is mutable when set. See [using volume mutations](../../using.md#using_volume_mutations).</small>

Example default `StorageClass` ([download](examples/storageclass.yaml)):

```yaml
{% include "csi_driver/container_storage_provider/hpe_alletra_storage_mp_x10000/examples/storageclass.yaml" %}```

!!! important
    Pay attention to the `mountOptions` stanza, the NFS client and server negotiation fails unless "vers=4.1" is provided.

### Access Clients and Networks

The "accessControlList" `StorageClass parameter may hold up to 50 comma separated entries with a maximum string length of 253 characters, including the commas.

Here are some example entries that can be applied to limit access to the export.

| Rule                           | Example | 
| ------------------------------ | ------- |
| Host by IP address             | `192.168.1.10` `172.16.1.10` |
| Host by name<sup>1</sup>       | `my-host-01` `my-host-*` `my-host-0?` `my-host-01.example.com` | 
| Domain names<sup>1</sup>       | `*.example.com` `*.my-sub-10.example.com` `*.my-sub-1?.example.com` |
| Network by CIDR                | `192.168.1.0/24` `172.16.0.0/16` `10.0.0.0/8` |
| Network by wildcard            | `192.168.1.*` `172.16.*.*` `10.10.1?.*` |
| Network by CIDR and wildcard   | `192.168.*.0/24` `172.16.1?.*/16` `10.0.*.0/8` | 

<small><sup>1</sup> = Requires DNS properly configured on the X10000 platform.</small>

!!! Note
    While the X10000 supports IPv6, the CSI driver and CSP is not compatible with IPv6 on the X10000 at this time.

In a real world example you could for example allow production clusters access to the exports but have a subset of hosts in test and stage for auxiliary use cases.

```yaml
...
parameters:
  accessProtocol: nfs
  accessControlList: "my-k8s-worker-*-prod,my-k8s-worker-0?-staging,172.16.20.44"
                   # Allow access to all prod. 
                   # The first 10 hosts in staging are tainted for prod-like use.
                   # 172.16.20.44 is the old Jenkins box, WARNING DO NOT REMOVE.
...
```

## Static Provisioning

In order to use existing exports on the backend with the CSI driver the name and UUID of the filesystem needs to be known. In the example below, the name is "my-static-filesystem" and needs to be called out in `.metadata.name` and `.spec.csi.volumeAttributes.csi.storage.k8s.io/pv/name`. The UUID needs to be populated in `.spec.csi.volumeHandle`.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-static-filesystem
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 128Gi
  csi:
    controllerPublishSecretRef:
      name: hpe-backend
      namespace: hpe-storage
    driver: csi.hpe.com
    nodePublishSecretRef:
      name: hpe-backend
      namespace: hpe-storage
    volumeAttributes:
      csi.storage.k8s.io/pv/name: my-static-filesystem
      volumeAccessMode: mount
      description: Volume statically provisioned by HPE CSI Driver for Kubernetes
      accessProtocol: nfs
      accessControlList: "*"
    volumeHandle: 00000000-0000-0000-0000-000000000000
  mountOptions:
  - vers=4.1
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
```

Create a `PVC` and explicitly call out the `PersistentVolume` name. The size must match as well.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-static-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 128Gi
  volumeName: my-static-filesystem
  storageClassName: ""
```

## Limitations

The are the known limitations of the CSP. Please refer to the [HPE Alletra Storage MP X10000 QuickSpecs](https://www.hpe.com/us/en/collaterals/collateral.a50009215enw.html) for platform limits. Also, be familiar with the HPE CSI Driver [limitations](../../index.md#known_limitations).

- The X10000 needs to be running 2.0.0.0 or later in order to be supported by the HPE CSI Driver.
- The Kubernetes cluster needs to be running HPE CSI Driver for Kubernetes 3.2.0 or later.
- Data management features, such as snapshots and clones is not implemented yet.
- The X10000 does not have a concept of limiting capacity on an export. The `.spec.resources.requsts.storage` value in the `PersistentVolumeClaim` does not matter, hence volume expansion is not implemented.
- NFS uses the same "frontend" data IP addresses configured for the S3 protocol. `PersistentVolumes` are deterministically mapped to an IP address in the range. If the range is altered in any way, all workloads using `PersistentVolumeClaims` must be scaled down to zero on the Kubernetes cluster.
- IPv6 is currently not supported by the CSI driver and CSP with X10000.

## Support

The HPE Alletra Storage MP X10000 CSP is supported by HPE and covered by the platform support agreement. See [support](../../../legal/support/index.md#hpe_alletra_storage_mp_x10000_container_storage_provider_support) for more details.
