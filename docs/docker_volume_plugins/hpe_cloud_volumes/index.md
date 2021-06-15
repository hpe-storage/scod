!!! error "Expired content"
    The documentation described on this page may be obsolete and contain references to unsupported and deprecated software. Please reach out to your HPE representative if you think you need any of the components referenced within.

# Introduction

This is the documentation for [HPE Cloud Volumes Plugin for Docker](https://hub.docker.com/plugins/cvblock). It allows dynamic provisioning of Docker Volumes on standalone Docker Engine or Docker Swarm nodes.

[TOC]

## Requirements

- Docker Engine 17.09 or greater
- If using Docker Enterprise Edition 2.x, the plugin is only supported in swarmmode
- Recent Red Hat, Debian or Ubuntu-based Linux distribution
- US regions only

| Plugin      | Release Notes    |
|-------------|------------------|
| 3.1.0       | [v3.1.0](https://github.com/hpe-storage/common-host-utils/blob/master/cmd/dockervolumed/managedplugin/release-notes/v3.1.0.md)|

!!! note
    Docker does not support certified and managed Docker Volume plugins with Docker EE Kubernetes. If you want to use Kubernetes on Docker EE with HPE Nimble Storage, please use the [HPE Volume Driver for Kubernetes FlexVolume Plugin](https://github.com/hpe-storage/flexvolume-driver) or the [HPE CSI Driver for Kubernetes](https://github.com/hpe-storage/csi-driver) depending on your situation.

## Limitations
HPE Cloud Volumes provides a Docker certified plugin delivered through the Docker Store. Certain features and capabilities are not available through the managed plugin. Please understand these limitations before deploying either of these plugins.

The managed plugin does NOT provide:

- Support for Docker's release of Kubernetes in Docker Enterprise Edition 2.x
- Support for Windows Containers

The managed plugin does provide a simple way to manage HPE Cloud Volumes integration on your Docker instances using Docker's interface to install and manage the plugin.

## Installation

### Plugin privileges
In order to create connections, attach devices and mount file systems, the plugin requires more privileges than a standard application container. These privileges are enumerated during installation. These permissions need to be granted for the plugin to operate correctly.

```shell
Plugin "cvblock" is requesting the following privileges:
 - network: [host]
 - mount: [/dev]
 - mount: [/run/lock]
 - mount: [/sys]
 - mount: [/etc]
 - mount: [/var/lib]
 - mount: [/var/run/docker.sock]
 - mount: [/sbin/iscsiadm]
 - mount: [/lib/modules]
 - mount: [/usr/lib64]
 - allow-all-devices: [true]
 - capabilities: [CAP_SYS_ADMIN CAP_SYS_MODULE CAP_MKNOD]
```
### Host configuration and installation

Setting up the plugin varies between Linux distributions.

These procedures **requires** root privileges on the cloud instance.

Red Hat 7.5+, CentOS 7.5+:
```shell
yum install -y iscsi-initiator-utils device-mapper-multipath
docker plugin install --disable --grant-all-permissions --alias cvblock store/hpestorage/cvblock:3.1.0
docker plugin set cv PROVIDER_IP=cloudvolumes.hpe.com PROVIDER_USERNAME=<access_key> PROVIDER_PASSWORD=<access_secret>
docker plugin enable cvblock
systemctl daemon-reload
systemctl enable iscsid multipathd
systemctl start iscsid multipathd
```

Ubuntu 16.04 LTS and Ubuntu 18.04 LTS:
```shell
apt-get install -y open-iscsi multipath-tools xfsprogs
modprobe xfs
sed -i"" -e "\$axfs" /etc/modules
docker plugin install --disable --grant-all-permissions --alias cvblock store/hpestorage/cvblock:3.1.0
docker plugin set cv PROVIDER_IP=cloudvolumes.hpe.com PROVIDER_USERNAME=<access_key> PROVIDER_PASSWORD=<access_secret> glibc_libs.source=/lib/x86_64-linux-gnu
docker plugin enable cvblock
systemctl daemon-reload
systemctl restart open-iscsi multipath-tools
```

Debian 9.x (stable):
```shell
apt-get install -y open-iscsi multipath-tools xfsprogs
modprobe xfs
sed -i"" -e "\$axfs" /etc/modules
docker plugin install --disable --grant-all-permissions --alias cvblock store/hpestorage/cvblock:3.1.0
docker plugin set cv PROVIDER_IP=cloudvolumes.hpe.com PROVIDER_USERNAME=<access_key> PROVIDER_PASSWORD=<access_secret> iscsiadm.source=/usr/bin/iscsiadm glibc_libs.source=/lib/x86_64-linux-gnu
docker plugin enable cvblock
systemctl daemon-reload
systemctl restart open-iscsi multipath-tools
```

### Making changes
The `docker plugin set` command can only be used on the plugin if it is disabled. To disable the plugin, use the `docker plugin disable` command. For example:

```shell
docker plugin disable cvblock
```

List of parameters which are supported to be settable by the plugin

|  Parameter              |  Description                                                |  Default |
|-------------------------|-------------------------------------------------------------|----------|
| `PROVIDER_IP`           | HPE Cloud Volumes portal                                    |`""`      |
| `PROVIDER_USERNAME`     | HPE Cloud Volumes username                                  |`""`      |
| `PROVIDER_PASSWORD`     | HPE Cloud Volumes password                                  |`""`      |
| `PROVIDER_REMOVE`       | Unassociate Plugin from HPE Cloud Volumes                   | `false`  |
| `LOG_LEVEL`             | Log level of the plugin (`info`, `debug`, or `trace`)       | `debug`  |
| `SCOPE`                 | Scope of the plugin (`global` or `local`)                   | `global` |

In the event of reassociating the plugin with a different HPE Cloud Volumes portal, certain procedures need to be followed:

Disable the plugin

```shell
docker plugin disable cvblock
```

Set new paramters

```shell
docker plugin set cvblock PROVIDER_REMOVE=true
```

Enable the plugin

```shell
docker plugin enable cvblock
```

Disable the plugin

```shell
docker plugin disable cvblock
```

The plugin is now ready for re-configuration

```shell
docker plugin set cvblock PROVIDER_IP=< New portal address > PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin PROVIDER_REMOVE=false
```

!!! note
    The `PROVIDER_REMOVE=false` parameter must be set if the plugin ever has been unassociated from a HPE Cloud Volumes portal.

## Configuration files and options
The configuration directory for the plugin is `/etc/hpe-storage` on the host. Files in this directory are preserved between plugin upgrades. The `/etc/hpe-storage/volume-driver.json` file contains three sections, `global`, `defaults` and `overrides`. The global options are plugin runtime parameters and doesn't have any end-user configurable keys at this time.

The `defaults` map allows the docker host administrator to set default options during volume creation. The docker user may override these default options with their own values for a specific option.

The `overrides` map allows the docker host administrator to enforce a certain option for every volume creation. The docker user may not override the option and any attempt to do so will be silently ignored.

!!! note
    `defaults` and `overrides` are dynamically read during runtime while `global` changes require a plugin restart.

Example config file in `/etc/hpe-storage/volume-driver.json`:
```json
{
  "global": {
            "snapPrefix": "BaseFor",
            "initiators": ["eth0"],
            "automatedConnection": true,
            "existingCloudSubnet": "10.1.0.0/24",
            "region": "us-east-1",
            "privateCloud": "vpc-data",
            "cloudComputeProvider": "Amazon AWS"
  },
  "defaults": {
            "limitIOPS": 1000,
            "fsOwner": "0:0",
            "fsMode": "600",
            "description": "Volume provisioned by the HPE Volume Driver for Kubernetes FlexVolume Plugin",
            "perfPolicy": "Other",
            "protectionTemplate": "twicedaily:4",
            "encryption": true,
            "volumeType": "PF",
            "destroyOnRm": true
  },
  "overrides": {
  }
}
```

For an exhaustive list of options use the `help` option from the docker CLI:

```shell
$ docker volume create -d cvblock -o help
```

## Node fencing
If you are considering using any Docker clustering technologies for your Docker deployment, it is important to understand the fencing mechanism used to protect data. Attaching the same Docker Volume to multiple containers on the same host is fully supported. Mounting the same volume on multiple hosts is not supported.

Docker does not provide a fencing mechanism for nodes that have become disconnected from the Docker Swarm. This results in the isolated nodes continuing to run their containers. When the containers are rescheduled on a surviving node, the Docker Engine will request that the Docker Volume(s) be mounted. In order to prevent data corruption, the Docker Volume Plugin will stop serving the Docker Volume to the original node before mounting it on the newly requested node.

During a mount request, the Docker Volume Plugin inspects the ACR (Access Control Record) on the volume. If the ACR does not match the initiator requesting to mount the volume, the ACR is removed and the volume taken offline. The volume is now fenced off and other nodes are unable to access any data in the volume.

The volume then receives a new ACR matching the requesting initiator, and it is mounted for the container requesting the volume. This is done because the volumes are formatted with XFS, which is not a clustered filesystem and can be corrupted if the same volume is mounted to multiple hosts.

The side effect of a fenced node is that I/O hangs indefinitely, and the initiator is rejected during login. If the fenced node rejoins the Docker Swarm using Docker SwarmKit, the swarm tries to shut down the services that were rescheduled elsewhere to maintain the desired replica set for the service. This operation will also hang indefinitely waiting for I/O.

We recommend running a dedicated Docker host that does not host any other critical applications besides the Docker Engine. Doing this supports a safe way to reboot a node after a grace period and have it start cleanly when a hung task is detected. Otherwise, the node can remain in the hung state indefinitely.

The following kernel parameters control the system behavior when a hung task is detected:

```markdown
# Reset after these many seconds after a panic
kernel.panic = 5

# I do consider hung tasks reason enough to panic
kernel.hung_task_panic = 1

# To not panic in vain, I'll wait these many seconds before I declare a hung task
kernel.hung_task_timeout_secs = 150
```

Add these parameters to the `/etc/sysctl.d/99-hung_task_timeout.conf` file and reboot the system.

!!! important
    Docker SwarmKit declares a node as failed after five (5) seconds. Services are then rescheduled and up and running again in less than ten (10) seconds. The parameters noted above provide the system a way to manage other tasks that may appear to be hung and avoid a system panic.

## Usage
These are some basic examples on how to use the HPE Cloud Volumes Plugin for Docker.

### Create a Docker Volume
Using `docker volume create`.

!!! note
    The plugin applies a set of default options when you `create` new volumes unless you override them using the `volume create -o key=value` option flags.

Create a Docker volume with a custom description:

```shell
docker volume create -d cvblock -o description="My volume description" --name myvol1 
```

(Optional) Inspect the new volume:

```shell
docker volume inspect myvol1
```

(Optional) Attach the volume to an interactive container.

```shell
docker run -it --rm -v myvol1:/data bash
```

The volume is mounted inside the container on `/data`.

### Clone a Docker Volume
Use the `docker volume create` command with the `cloneOf` option to clone a Docker volume to a new Docker volume.

Clone the Docker volume named `myvol1` to a new Docker volume named `myvol1-clone`.

```shell
docker volume create -d cvblock -o cloneOf=myvol1 --name=myvol1-clone
```

(Optional) Select a snapshot on which to base the clone.

```shell
docker volume create -d cvblock -o snapshot=mysnap1 -o cloneOf=myvol1 --name=myvol2-clone
```

### Provisioning Docker Volumes
There are several ways to provision a Docker volume depending on what tools are used:

- Docker Engine (CLI)
- Docker Compose file with either Docker UCP or Docker Engine

The Docker Volume plugin leverages the existing Docker CLI and APIs, therefor all native Docker tools may be used to provision a volume.

!!! note
    The plugin applies a set of default volume create options. Unless you override the default options using the volume option flags, the defaults are applied when you create volumes. For example, the default volume size is 10GiB.  

Config file `volume-driver.json`, which is stored at `/etc/hpe-storage/volume-driver.json`:

```json
{
    "global":   {},
    "defaults": {
                 "sizeInGiB":"10",
                 "limitIOPS":"-1",
                 "limitMBPS":"-1",
                 "perfPolicy": "DockerDefault",
                },
    "overrides":{}
}
```

### Import a Volume to Docker

Take the volume you want to import offline before importing it. For information about how to take a volume offline, refer to the [HPE Cloud Volumes documentation](https://docs.cloudvolumes.hpe.com/help/). Use the `create` command with the `importVol` option to import an HPE Cloud Volume to Docker and name it.

Import the HPE Cloud Volume named `mycloudvol` as a Docker volume named `myvol3-imported`.

```shell
docker volume create –d cvblock -o importVol=mycloudvol --name=myvol3-imported
```

### Import a volume snapshot to Docker

Use the create command with the `importVolAsClone` option to import a HPE Cloud Volume snapshot as a Docker volume. Optionally, specify a particular snapshot on the HPE Cloud Volume using the snapshot option.

Import the HPE Cloud Volumes snapshot `aSnapshot` on the volume `importMe` as a Docker volume named `importedSnap`.

```shell
docker volume create -d cvblock -o importVolAsClone=mycloudvol -o snapshot=mysnap1 --name=myvol4-clone
```

!!! note
    If no snapshot is specified, the latest snapshot on the volume is imported.

### Restore an offline Docker Volume with specified snapshot
It's important that the volume to be restored is in an offline state on the array.

If the volume snapshot is not specified, the last volume snapshot is used.

```shell
docker volume create -d cvblock -o importVol=myvol1.docker -o forceImport -o restore -o snapshot=mysnap1 --name=myvol1-restored
```

### List volumes
List Docker volumes.

```shell
docker volume ls
DRIVER                     VOLUME NAME
cvblock:latest              myvol1
cvblock:latest              myvol1-clone
```

### Remove a Docker Volume
When you remove volumes from Docker control they are set to the offline state on the array. Access to the volumes and related snapshots using the Docker Volume plugin can be reestablished.

!!! note
    To delete volumes from the HPE Cloud Volumes portal using the remove command, the volume should have been created with a `-o destroyOnRm` flag.

**Important:** Be aware that when this option is set to true, volumes and all related snapshots are deleted from the group, and can no longer be accessed by the Docker Volume plugin.

Remove the volume named `myvol1`.

```shell
docker volume rm myvol1
```

## Uninstall
The plugin can be removed using the `docker plugin rm` command. This command will not remove the configuration directory (`/etc/hpe-storage/`).

```shell
docker plugin rm cvblock
```

## Troubleshooting

The config directory is at `/etc/hpe-storage/`. When a plugin is installed and enabled, the HPE Cloud Volumes certificates are created in the config directory.

```shell
ls -l /etc/hpe-storage/
total 16
-r-------- 1 root root 1159 Aug  2 00:20 container_provider_host.cert
-r-------- 1 root root 1671 Aug  2 00:20 container_provider_host.key
-r-------- 1 root root 1521 Aug  2 00:20 container_provider_server.cert
```

Additionally there is a config file `volume-driver.json` present at the same location. This file can be edited
to set default parameters for create volumes for docker.

### Log file location

The docker plugin logs are located at `/var/log/hpe-docker-plugin.log`
