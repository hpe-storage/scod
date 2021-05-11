!!! error "Expired content"
    The documentation described on this page may be obsolete and contain references to unsupported and deprecated software. Please reach out to your HPE representative if you think you need any of the components referenced within.

# Overview
The HPE 3PAR and Primera Volume Plug-in for Docker leverages Ansible to deploy the 3PAR/Primera driver for Kubernetes in order to provide scalable and persistent storage for stateful applications. 

!!! important
    Using HPE 3PAR/Primera Storage with Kubernetes 1.15 and newer, please use the [HPE CSI Driver for Kubernetes](https://github.com/hpe-storage/csi-driver).

Source code is available in the [hpe-storage/python-hpedockerplugin](https://github.com/hpe-storage/python-hpedockerplugin) GitHub repo.

[TOC]

Refer to the [SPOCK](https://spock.corp.int.hpe.com/spock/utility/document.aspx?docurl=Shared%20Documents/hw/3par/3par_volume_plugin_for_docker.pdf) page for the latest support matrix for HPE 3PAR and HPE Primera Volume Plug-in for Docker.

## Platform requirements
The HPE 3PAR/Primera FlexVolume driver supports multiple backends that are based on a "container provider" architecture.

### HPE 3PAR/Primera Storage Platform Requirements
                
Ensure that you have reviewed the [System Requirements](https://github.com/hpe-storage/python-hpedockerplugin/blob/master/README.md).

| Driver  | HPE 3PAR/Primera OS Version | Release Notes  |
|-------------|----------------------------|------------------|
| v3.3.1  | 3PAR OS: 3.3.1 MU5+<br>  Primera OS: 4.0+  | [v3.3.1](https://github.com/hpe-storage/python-hpedockerplugin/releases)|


* OpenShift Container Platform 3.9, 3.10 and 3.11.
* Kubernetes 1.10 and above.
* Redhat/CentOS 7.5+

**Note:** Refer to [SPOCK](https://spock.corp.int.hpe.com/spock/utility/document.aspx?docurl=Shared%20Documents/hw/3par/3par_volume_plugin_for_docker.pdf) page for the latest support matrix for HPE 3PAR and HPE Primera Volume Plug-in for Docker.

## Deploying to Kubernetes
The recommended way to deploy and manage the HPE 3PAR and Primera Volume Plug-in for Kubernetes is to use Ansible.

Use the following steps to configure Ansible to perform the installation.

### Step 1: Install Ansible

Ensure that Ansible (v2.5 to v2.8) is installed. For more information, see [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

**NOTE:** Ansible only needs to be installed on the machine that will be performing the deployment. Ansible does not need to be installed on your Kubernetes cluster.

```shell
$ pip install ansible
$ ansible --version
ansible 2.7.12
```

##### Ansible: Connecting to remote nodes
Ansible communicates with remote machines over the SSH protocol. By default, Ansible uses native OpenSSH and connects to remote machines using your current user name, just as SSH does.

##### Ansible: Check your SSH connections
Confirm that you can connect using SSH to all the nodes in your Kubernetes cluster using the same username. If necessary, add your public SSH key to the `authorized_keys` file on those systems.

### Step 2: Clone the Github repository

```shell
$ cd ~
$ git clone https://github.com/hpe-storage/python-hpedockerplugin
```

### Step 3: Modify the Ansible hosts file

Modify the `hosts` file to define the Kubernetes/OpenShift Master and Worker nodes. Also define where the HPE etcd cluster will be deployed, this can be done within the cluster or on external servers.

```shell
$ vi python-hpedockerplugin/ansible_3par_docker_plugin/hosts
```

```yaml
[masters]
192.168.1.51

[workers]
192.168.1.52
192.168.1.53

[etcd]
192.168.1.51
192.168.1.52
192.168.1.53
```

### Step 4: Create the properties file

Create the **properties/plugin_configuration_properties.yml** based on your HPE 3PAR/Primera Storage array configuration.

```shell
$ vi python-hpedockerplugin/ansible_3par_docker_plugin/properties/plugin_configuration_properties.yml
```
**NOTE:** Some of the properties are mandatory and must be specified in the properties file while others are optional.

```yaml
INVENTORY:
  DEFAULT:
#Mandatory Parameters--------------------------------------------------------------------------------

    # Specify the port to be used by HPE 3PAR plugin etcd cluster
    host_etcd_port_number: 23790
    # Plugin Driver - iSCSI
    hpedockerplugin_driver: hpedockerplugin.hpe.hpe_3par_iscsi.HPE3PARISCSIDriver
    hpe3par_ip: <3par_array_IP>
    hpe3par_username: <3par_user>
    hpe3par_password: <3par_password>
    #Specify the 3PAR port - 8080 default
    hpe3par_port: 8080
    hpe3par_cpg: <cpg_name>

    # Plugin version - Required only in DEFAULT backend
    volume_plugin: hpestorage/legacyvolumeplugin:3.3.1
    # Dory installer version - Required for Openshift/Kubernetes setup
    # Supported versions are dory_installer_v31, dory_installer_v32
    dory_installer_version: dory_installer_v32

#Optional Parameters--------------------------------------------------------------------------------

    logging: DEBUG
    hpe3par_snapcpg: FC_r6
    #hpe3par_iscsi_chap_enabled: True
    use_multipath: True
    #enforce_multipath: False
    #vlan_tag: True

```

**Available Properties Parameters**

| Property  | Mandatory | Default Value | Description |
| ----------| ----------| ------------- | ------------|
| hpedockerplugin_driver  | Yes  | No default value  | ISCSI/FC driver  (hpedockerplugin.hpe.hpe_3par_iscsi.HPE3PARISCSIDriver/hpedockerplugin.hpe.hpe_3par_fc.HPE3PARFCDriver) |
| hpe3par_ip  | Yes  | No default value | IP address of 3PAR array |
| hpe3par_username  | Yes  | No default value | 3PAR username |
| hpe3par_password  | Yes  | No default value | 3PAR password |
| hpe3par_port  | Yes  | 8080 | 3PAR HTTP_PORT port |
| hpe3par_cpg  | Yes  | No default value | Primary user CPG |
| volume_plugin  | Yes  | No default value | Name of the docker volume image (only required with DEFAULT backend) |
| encryptor_key  | No  | No default value | Encryption key string for 3PAR password |
| logging  | No  | INFO | Log level |
| hpe3par_debug  | No  | No default value | 3PAR log level |
| suppress_requests_ssl_warning  | No  | True | Suppress request SSL warnings |
| hpe3par_snapcpg  | No  | hpe3par_cpg | Snapshot CPG |
| hpe3par_iscsi_chap_enabled  | No  | False | ISCSI chap toggle |
| hpe3par_iscsi_ips  | No  |No default value | Comma separated iscsi port IPs (only required if driver is ISCSI based) |
| use_multipath  | No  | False | Mutltipath toggle |
| enforce_multipath  | No  | False | Forcefully enforce multipath |
| ssh_hosts_key_file  | No  | /root/.ssh/known_hosts | Path to hosts key file |
| quorum_witness_ip  | No  | No default value | Quorum witness IP |
| mount_prefix  | No  | No default value | Alternate mount path prefix |
| hpe3par_iscsi_ips  | No  | No default value | Comma separated iscsi IPs. If not provided, all iscsi IPs will be read from the array and populated in hpe.conf |
| vlan_tag  | No  | False | Populates the iscsi_ips which are vlan tagged, only applicable if hpe3par_iscsi_ips is not specified |
| replication_device  | No  | No default value | Replication backend properties |
| dory_installer_version  | No  | dory_installer_v32 | Required for Openshift/Kubernetes setup. Dory installer version, supported versions are dory_installer_v31, dory_installer_v32 |
| hpe3par_server_ip_pool  | Yes  | No default value | This parameter is specific to fileshare. It can be specified as a mix of range of IPs and individual IPs delimited by comma. Each range or individual IP must be followed by the corresponding subnet mask delimited by semi-colon E.g.: IP-Range:Subnet-Mask,Individual-IP:SubnetMask|
| hpe3par_default_fpg_size  | No  | No default value | This parameter is specific to fileshare. Default fpg size, It must be in the range 1TiB to 64TiB. If not specified here, it defaults to 16TiB |

!!! Hint 
    Refer to [Replication Support](#replication_support) for details on enabling Replication support.

##### File Persona Example Configuration

```markdown
#Mandatory Parameters for Filepersona---------------------------------------------------------------
  DEFAULT_FILE:
    # Specify the port to be used by HPE 3PAR plugin etcd cluster
    host_etcd_port_number: 23790
    # Plugin Driver - File driver
    hpedockerplugin_driver: hpedockerplugin.hpe.hpe_3par_file.HPE3PARFileDriver
    hpe3par_ip: 192.168.2.50
    hpe3par_username: demo_user
    hpe3par_password: demo_pass
    hpe3par_cpg: demo_cpg
    hpe3par_port: 8080
    hpe3par_server_ip_pool: 192.168.98.3-192.168.98.10:255.255.192.0 
#Optional Parameters for Filepersona----------------------------------------------------------------
    hpe3par_default_fpg_size: 16
```

##### Multiple Backend Example Configuration

```markdown
INVENTORY:
  DEFAULT:
#Mandatory Parameters-------------------------------------------------------------------------------

    # Specify the port to be used by HPE 3PAR plugin etcd cluster
    host_etcd_port_number: 23790
    # Plugin Driver - iSCSI
    hpedockerplugin_driver: hpedockerplugin.hpe.hpe_3par_iscsi.HPE3PARISCSIDriver
    hpe3par_ip: 192.168.1.50
    hpe3par_username: 3paradm
    hpe3par_password: 3pardata
    hpe3par_port: 8080
    hpe3par_cpg: FC_r6

    # Plugin version - Required only in DEFAULT backend
    volume_plugin: hpestorage/legacyvolumeplugin:3.3.1
    # Dory installer version - Required for Openshift/Kubernetes setup
    # Supported versions are dory_installer_v31, dory_installer_v32
    dory_installer_version: dory_installer_v32

#Optional Parameters--------------------------------------------------------------------------------

    #ssh_hosts_key_file: '/root/.ssh/known_hosts'
    logging: DEBUG
    #hpe3par_debug: True
    #suppress_requests_ssl_warning: True
    #hpe3par_snapcpg: FC_r6
    #hpe3par_iscsi_chap_enabled: True
    #use_multipath: False
    #enforce_multipath: False
    #vlan_tag: True

#Additional Backend (Optional)----------------------------------------------------------------------
  
  3PAR1:
#Mandatory Parameters-------------------------------------------------------------------------------

    # Specify the port to be used by HPE 3PAR plugin etcd cluster
    host_etcd_port_number: 23790
    # Plugin Driver - Fibre Channel
    hpedockerplugin_driver: hpedockerplugin.hpe.hpe_3par_fc.HPE3PARFCDriver
    hpe3par_ip: 192.168.2.50
    hpe3par_username: 3paradm
    hpe3par_password: 3pardata
    hpe3par_port: 8080
    hpe3par_cpg: FC_r6

#Optional Parameters--------------------------------------------------------------------------------

    #ssh_hosts_key_file: '/root/.ssh/known_hosts'
    logging: DEBUG
    #hpe3par_debug: True
    #suppress_requests_ssl_warning: True
    hpe3par_snapcpg: FC_r6
    #use_multipath: False
    #enforce_multipath: False
```

### Step 5: Run the Ansible playbook
```markdown
$ cd python-hpedockerplugin/ansible_3par_docker_plugin/
$ ansible-playbook -i hosts install_hpe_3par_volume_driver.yml
```

### Step 6: Verify the installation

* Once playbook has completed successfully, the PLAY RECAP should look like below
```markdown
Installer should not show any failures and PLAY RECAP should look like below

PLAY RECAP ***********************************************************************
<Master1-IP>           : ok=85   changed=33   unreachable=0    failed=0
<Master2-IP>           : ok=76   changed=29   unreachable=0    failed=0
<Master3-IP>           : ok=76   changed=29   unreachable=0    failed=0
<Worker1-IP>           : ok=70   changed=27   unreachable=0    failed=0
<Worker2-IP>           : ok=70   changed=27   unreachable=0    failed=0
localhost              : ok=9    changed=3    unreachable=0    failed=0
```

* Verify plugin installation on all nodes.

```markdown
$ docker ps | grep plugin; ssh <Master2-IP> "docker ps | grep plugin";ssh <Master3-IP> "docker ps | grep plugin";ssh <Worker1-IP> "docker ps | grep plugin";ssh <Worker2-IP> "docker ps | grep plugin"
51b9d4b1d591        hpestorage/legacyvolumeplugin:3.3.1          "/bin/sh -c ./plugin…"   12 minutes ago      Up 12 minutes         plugin_container
a43f6d8f5080        hpestorage/legacyvolumeplugin:3.3.1          "/bin/sh -c ./plugin…"   12 minutes ago      Up 12 minutes         plugin_container
a88af9f46a0d        hpestorage/legacyvolumeplugin:3.3.1          "/bin/sh -c ./plugin…"   12 minutes ago      Up 12 minutes         plugin_container
5b20f16ab3af        hpestorage/legacyvolumeplugin:3.3.1          "/bin/sh -c ./plugin…"   12 minutes ago      Up 12 minutes         plugin_container
b0813a22cbd8        hpestorage/legacyvolumeplugin:3.3.1          "/bin/sh -c ./plugin…"   12 minutes ago      Up 12 minutes         plugin_container
```

* Verify the HPE FlexVolume driver Pod is running.

```shell
kubectl get pods -n kube-system | grep doryd
NAME                                            READY   STATUS    RESTARTS   AGE
kube-storage-controller-doryd-7dd487b446-xr6q2  1/1     Running   0          45s
```

## Using
Get started using the FlexVolume driver by setting up `StorageClass`, `PVC` API objects. See [Using](#using) for examples.

These instructions are provided as an example on how to use the HPE 3PAR/Primera Volume Plug-in with a HPE 3PAR/Primera Storage Array.

The below YAML declarations are meant to be created with `kubectl create`. Either copy the content to a file on the host where `kubectl` is being executed, or copy & paste into the terminal, like this:

```shell
kubectl create -f-
< paste the YAML >
^D (CTRL + D)
```

!!! tip
    Some of the examples supported by the HPE 3PAR/Primera FlexVolume driver are available for [HPE 3PAR/Primera Storage](https://github.com/hpe-storage/hpe3par-examples/tree/master/containers/kubernetes-openshift/samples) in the GitHub repo.

To get started, create a `StorageClass` API object referencing the `hpe-secret` and defining additional (optional) `StorageClass` parameters:

### Sample StorageClass

Sample storage classes can be found for [HPE 3PAR/Primera Storage](https://github.com/hpe-storage/hpe3par-examples/blob/master/containers/kubernetes-openshift/samples/provisioning/sc-gold.yml).

### Test and verify volume provisioning

Create a `StorageClass` with volume parameters as required. **Change the CPG per your requirements.**

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-gold
provisioner: hpe.com/hpe
parameters:
  provisioning: 'full'
  cpg: 'SSD_r6'
  fsOwner: '1001:1001'
```

Create a `PersistentVolumeClaim`. This makes sure a volume is created and provisioned on your behalf:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sc-gold-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 25Gi
  storageClassName: sc-gold
```

Check that a new `PersistentVolume` is created based on your claim:

```shell
$ kubectl get pv
NAME                                              CAPACITY     ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
sc-gold-pvc-13336da3-7ca3-11e9-826c-00505692581f  25Gi         RWO            Delete           Bound    default/pvc-gold    sc-gold                 3s
```

The above output means that the FlexVolume driver successfully provisioned a new volume and bound to the requesting `PVC` to a new `PV`. The volume is not attached to any node yet. It will only be attached to a node if a workload is scheduled to a specific node. Now let us create a `Pod` that refers to the above volume. When the `Pod` is created, the volume will be attached, formatted and mounted to the specified container:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
      name: "http-server"
    volumeMounts:
    - name: export
      mountPath: "/usr/share/nginx/html"
  volumes:
    - name: export
      persistentVolumeClaim:
        claimName: sc-gold-pvc
```

Check if the pod is running successfully:

```markdown
$ kubectl get pod pod-nginx
NAME          READY   STATUS    RESTARTS   AGE
pod-nginx     1/1     Running   0          2m29s
```

### Use case specific examples

This `StorageClass` examples help guide combinations of options when provisioning volumes.

#### Snapshot a volume

This `StorageClass` will create a snapshot of a "production" volume.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-gold-snap-mongo
provisioner: hpe.com/hpe
parameters:
  virtualCopyOf: "sc-mongo-10dc1195-779b-11e9-b787-0050569bb07c"
```

#### Clone a volume

This `StorageClass` will create clones of a "production" volume.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-gold-clone
provisioner: hpe.com/hpe
parameters:
  cloneOf: "sc-gold-2a82c9e5-6213-11e9-8d53-0050569bb07c"
```

#### Replicate a containerized volume

This `StorageClass` will add a standard backend volume to a 3PAR Replication Group. If the replicationGroup specified does not exist, the plugin will create one. See [Replication Support](#replication_support) for more details on configuring replication.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-mongodb-replicated
provisioner: hpe.com/hpe
parameters:
  provisioning: 'full'
  replicationGroup: 'mongodb-app1'
```

#### Import (cutover) a volume
This `StorageClass` will import an existing 3PAR/Primera volume to Kubernetes. The source volume needs to be offline for the import to succeed.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: import-clone-legacy-prod
provisioner: hpe.com/hpe
parameters:
  importVol: "production-db-vol"
```

#### Using overrides
The HPE Dynamic Provisioner for Kubernetes (doryd) understands a set of annotation keys a user can set on a `PVC`. If the corresponding keys exists in the list of the `allowOverrides` key in the `StorageClass`, the end-user can tweak certain aspects of the provisioning workflow. This opens up for very advanced data services.

StorageClass object:
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-gold
provisioner: hpe.com/hpe
parameters:
  provisioning: 'full'
  cpg: 'SSD_r6'
  fsOwner: '1001:1001'
  allowOverrides: provisioning,compression,cpg,fsOwner
```

PersistentVolumeClaim object:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: my-pvc
 annotations:
    hpe.com/provisioning: "thin"
    hpe.com/cpg: "FC_r6"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 25Gi
  storageClassName: sc-gold
```

This will create a `PV` thinly provisioned using the `FC-r6` cpg.

### Upgrade
In order to upgrade the driver, simply modify the `ansible_3par_docker_plugin/properties/plugin_configuration_properties_sample.yml` used for the initial deployment and modify `hpestorage/legacyvolumeplugin` to the latest image from docker hub.

For example:
```markdown
    volume_plugin: hpestorage/legacyvolumeplugin:3.3

    Change to:
    volume_plugin: hpestorage/legacyvolumeplugin:3.3.1
```

Re-run the installer.
```shell
$ ansible-playbook -i hosts install_hpe_3par_volume_driver.yml
```

### Uninstall 
Run the following to uninstall the FlexVolume driver from the cluster.

```
$ cd ~
$ cd python-hpedockerplugin/ansible_3par_docker_plugin
$ ansible-playbook -i hosts uninstall/uninstall_hpe_3par_volume_driver.yml
```

## StorageClass parameters
This section highlights all the available `StorageClass` parameters that are supported.

### HPE 3PAR/Primera Storage StorageClass parameters

A `StorageClass` is used to provision or clone an HPE 3PAR\Primera Storage-backed persistent volume. It can also be used to import an existing HPE 3PAR/Primera Storage volume or clone of a snapshot into the Kubernetes cluster. The parameters are grouped below by those same workflows.

A sample [StorageClass](https://raw.githubusercontent.com/hpe-storage/hpe3par-examples/master/containers/kubernetes-openshift/samples/provisioning/sc-gold.yml) is provided.

!!! note
    These are optional parameters.

#### Common parameters for Provisioning and Cloning

These parameters are mutable betweeen a parent volume and creating a clone from a snapshot.

| Parameter | Type | Options | Example  |
|-----------|-----------|-----------|-----------|
| size  | Integer | -  | size: "10"  |
| provisioning  |   | thin, full, dedupe  | provisioning: "thin"  |
| flash-cache  | Text  | true, false  | flash-cache: "true"  |
| compression  | boolean | true, false   | compression: "true"  |
| MountConflictDelay  | Integer | -  | MountConflictDelay: "30"  |
| qos-name  | Text  | vvset name  | qos-name: "<vvset_name>"  |
| replicationGroup  | Text  | 3PAR RCG name  | replicationGroup: "Test-RCG"  |
| fsOwner | userId:groupId | The user id and group id that should own the root directory of the filesystem. |
| fsMode | Octal digits | 1 to 4 octal digits that represent the file mode to be applied to the root directory of the filesystem. |

#### Cloning/Snapshot parameters

Either use `cloneOf` and reference a PVC in the current namespace or use `virtualCopyOf` and reference a 3PAR/Primera volume name to snapshot/clone and import into Kubernetes.

| Parameter | Type | Options | Example  |
|-----------|-----------|-----------|-----------|
| cloneOf  | Text  | volume name  | cloneOf: "<volume_name\>"  |
| virtualCopyOf  | Text  | volume name  | virtualCopyOf: "<volume_name\>"  |
| expirationHours  | Integer | option of virtualCopyOf  | expirationHours: "10"  |
| retentionHours  | Integer | option of virtualCopyOf  | retentionHours: "10"  |

#### Import parameters

Importing volumes to Kubernetes requires the source 3PAR/Primera volume to be offline. 

| Parameter | Type | Description | Example  |
|-----------|-----------|-----------|-----------|
| importVol | Text | volume name | importVol: "<volume_name\>" |

#### Replication Support

The HPE 3PAR/Primer FlexVolume driver supports array based synchronous and asynchronous replication. In order to enable replication within the FlexVolume driver, the arrays need to be properly zoned, visible to the Kubernetes cluster, and replication configured. For Peer Persistence, a quorum witness will need to be configured. 

Once the replication is enabled at the array level, the FlexVolume driver will need to be configured. 

!!! Important
    Replication support can be enabled during initial deployment through the plugin configuration file. In order to enable replication support post deployment, modify the **plugin_configuration_properties.yml** used for deployment, add the replication parameter section below, and re-run the Ansible installer.

Edit the **plugin_configuration_properties.yml** file and edit the Optional Replication Section.

```markdown
INVENTORY:
  DEFAULT:
#Mandatory Parameters-------------------------------------------------------------------------------

    # Specify the port to be used by HPE 3PAR plugin etcd cluster
    host_etcd_port_number: 23790
    # Plugin Driver - iSCSI
    hpedockerplugin_driver: hpedockerplugin.hpe.hpe_3par_iscsi.HPE3PARISCSIDriver
    hpe3par_ip: <local_3par_ip>
    hpe3par_username: <local_3par_user>
    hpe3par_password: <local_3par_password>
    hpe3par_port: 8080
    hpe3par_cpg: FC_r6

    # Plugin version - Required only in DEFAULT backend
    volume_plugin: hpestorage/legacyvolumeplugin:3.3.1
    # Dory installer version - Required for Openshift/Kubernetes setup
    dory_installer_version: dory_installer_v32

#Optional Parameters--------------------------------------------------------------------------------

    logging: DEBUG
    hpe3par_snapcpg: FC_r6
    use_multipath: False
    enforce_multipath: False

#Optional Replication Parameters--------------------------------------------------------------------
    replication_device:
      backend_id: remote_3PAR
      #Quorum Witness required for Peer Persistence only
      #quorum_witness_ip: <quorum_witness_ip>
      replication_mode: synchronous
      cpg_map: "local_CPG:remote_CPG"
      snap_cpg_map: "local_copy_CPG:remote_copy_CPG"
      hpe3par_ip: <remote_3par_ip>
      hpe3par_username: <remote_3par_user>
      hpe3par_password: <remote_3par_password>
      hpe3par_port: 8080
      #vlan_tag: False
```

Once the properties file is configured, you can proceed with the [standard installation steps](#step_5_run_the_ansible_playbook). 

## Diagnostics

This section outlines a few troubleshooting steps for the HPE 3PAR/Primera FlexVolume driver. This product is supported by HPE, please consult with your support organization prior attempting any configuration changes.

### Troubleshooting FlexVolume driver

The FlexVolume driver is a binary executed by the kubelet to perform mount/unmount/attach/detach operations as workloads request storage resources. The binary relies on communicating with a socket on the host where the volume plugin responsible for the MUAD operations perform control-plane or data-plane operations against the backend system hosting the actual volumes.

#### Locations

The driver has a configuration file where certain defaults can be tweaked to accommodate a certain behavior. Under normal circumstances, this file does not need any tweaking.

The name and the location of the binary varies based on Kubernetes distribution (the default 'exec' path) and what backend driver is being used. In a typical scenario, using 3PAR/Primera, this is expected:

* Binary: `/usr/libexec/kubernetes/kubelet-plugins/volume/exec/hpe.com~hpe/hpe`
* Config file: `/etc/hpedockerplugin/hpe.conf`

#### Connectivity
To verify the FlexVolume binary can actually communicate with the backend volume plugin, issue a faux mount request:

```
/usr/libexec/kubernetes/kubelet-plugins/volume/exec/hpe.com~hpe/hpe mount no/op '{"name":"myvol1"}'
```

If the FlexVolume driver can successfully communicate with the volume plugin socket:

```json
{"status":"Failure","message":"configured to NOT create volumes"}
```

In the case of any other output, check if the backend volume plugin is alive:

```shell
$ docker volume create -d hpe -o help=backends
```

It should output:

```shell
=================================
NAME                     STATUS
=================================
DEFAULT                   OK
```

#### ETCD
To verify the etcd members on nodes.
```shell
$ /usr/bin/etcdctl --endpoints http://<Master1-IP>:23790 member list 
```

It should output:
```shell
b70ca254f54dd23: name=<Worker2-IP> peerURLs=http://<Worker2-IP>:23800 clientURLs=http://<Worker2-IP>:23790 isLeader=true
236bf7d5cc7a32d4: name=<Worker1-IP> peerURLs=http://<Worker1-IP>:23800 clientURLs=http://<Worker1-IP>:23790 isLeader=false
445e80419ae8729b: name=<Master1-IP> peerURLs=http://<Master1-IP>:23800 clientURLs=http://<Master1-IP>:23790 isLeader=false
e340a5833e93861e: name=<Master3-IP> peerURLs=http://<Master3-IP>:23800 clientURLs=http://<Master3-IP>:23790 isLeader=false
f5b5599d719d376e: name=<Master2-IP> peerURLs=http://<Master2-IP>:23800 clientURLs=http://<Master2-IP>:23790 isLeader=false
```

#### HPE 3PAR/Primera FlexVolume and Dynamic Provisioner driver (doryd) logs

Log files associated with the HPE 3PAR/Primera FlexVolume driver logs data to the standard output stream. If the logs need to be retained for long term, use a standard logging solution. Some of the logs on the host are persisted which follow standard logrotate policies.

HPE 3PAR/Primera FlexVolume logs: (per node)
```shell
$ docker logs -f plugin_container
```

Dynamic Provisioner logs:

```shell
kubectl logs -f kube-storage-controller-doryd -n kube-system
```

The logs are persisted at `/var/log/hpe-dynamic-provisioner.log`
