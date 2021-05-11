!!! error "Expired content"
    The documentation described on this page may be obsolete and contain references to unsupported and deprecated software. Please reach out to your HPE representative if you think you need any of the components referenced within.

# Introduction

This is the documentation for [HPE Nimble Storage Volume Plugin for Docker](https://hub.docker.com/plugins/nimble). It allows dynamic provisioning of Docker Volumes on standalone Docker Engine or Docker Swarm nodes.

[TOC]

## Requirements

- Docker Engine 17.09 or greater
- If using Docker Enterprise Edition 2.x, the plugin is only supported in swarmmode
- Recent Red Hat, Debian or Ubuntu-based Linux distribution
- NimbleOS 5.0.8/5.1.3 or greater

| Plugin      | HPE Nimble Storage Version | Release Notes    |
|-------------|----------------------------|------------------|
| 3.0.0      | 5.0.8.x and 5.1.3.x onwards | [v3.0.0](https://github.com/hpe-storage/common-host-utils/blob/master/cmd/dockervolumed/managedplugin/release-notes/v3.0.0.md)|
| 3.1.0      | 5.0.8.x and 5.1.3.x onwards | [v3.1.0](https://github.com/hpe-storage/common-host-utils/blob/master/cmd/dockervolumed/managedplugin/release-notes/v3.1.0.md)|

!!! note
    Docker does not support certified and managed Docker Volume plugins with Docker EE Kubernetes. If you want to use Kubernetes on Docker EE with HPE Nimble Storage, please use the [HPE Volume Driver for Kubernetes FlexVolume Plugin](https://github.com/hpe-storage/flexvolume-driver) or the [HPE CSI Driver for Kubernetes](https://github.com/hpe-storage/csi-driver) depending on your situation.

## Limitations
HPE Nimble Storage provides a Docker certified plugin delivered through the Docker Store. HPE Nimble Storage also provides a Docker Volume plugin for Windows Containers, it's available on [HPE InfoSight](https://infosight.hpe.com/org/urn%3Ainfosight%3A605d9baf-c394-4882-9742-a44bd8678cad/resources/nimble/software/Integration%20Kits/HPE%20Nimble%20Storage%20Volume%20Plugin%20for%20Docker%20Windows%20Containers) along with its [documentation](https://infosight.hpe.com/InfoSight/media/software/active/18/291/pubs_Volume_Plugin_for_Docker_on_Windows_Containers_Guide_Docker_2_0_0.pdf). Certain features and capabilities are not available through the managed plugin. Please understand these limitations before deploying either of these plugins.

The managed plugin does NOT provide:

- Support for Docker's release of Kubernetes in Docker Enterprise Edition 2.x
- Support for older versions of NimbleOS (all versions below 5.x)
- Support for Windows Containers

The managed plugin does provide a simple way to manage HPE Nimble Storage on your Docker hosts using Docker's interface to install and manage the plugin.

## Installation

### Plugin privileges
In order to create connections, attach devices and mount file systems, the plugin requires more privileges than a standard application container. These privileges are enumerated during installation. These permissions need to be granted for the plugin to operate correctly.

```shell
Plugin "nimble" is requesting the following privileges:
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

Setting up the plugin varies between Linux distributions. The following workflows have been tested using a Nimble iSCSI group array at **192.168.171.74** with `PROVIDER_USERNAME` **admin** and `PROVIDER_PASSWORD` **admin**:

These procedures **require** root privileges.

Red Hat 7.5+, CentOS 7.5+:
```shell
yum install -y iscsi-initiator-utils device-mapper-multipath
docker plugin install --disable --grant-all-permissions --alias nimble store/nimblestorage/nimble:3.1.0
docker plugin set nimble PROVIDER_IP=192.168.1.1 PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin
docker plugin enable nimble
systemctl daemon-reload
systemctl enable iscsid multipathd
systemctl start iscsid multipathd
```

Ubuntu 16.04 LTS and Ubuntu 18.04 LTS:
```shell
apt-get install -y open-iscsi multipath-tools xfsprogs
modprobe xfs
sed -i"" -e "\$axfs" /etc/modules
docker plugin install --disable --grant-all-permissions --alias nimble store/nimblestorage/nimble:3.1.0
docker plugin set nimble PROVIDER_IP=192.168.1.1 PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin glibc_libs.source=/lib/x86_64-linux-gnu
docker plugin enable nimble
systemctl daemon-reload
systemctl restart open-iscsi multipath-tools
```

Debian 9.x (stable):
```shell
apt-get install -y open-iscsi multipath-tools xfsprogs
modprobe xfs
sed -i"" -e "\$axfs" /etc/modules
docker plugin install --disable --grant-all-permissions --alias nimble store/nimblestorage/nimble:3.1.0
docker plugin set nimble PROVIDER_IP=192.168.1.1 PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin iscsiadm.source=/usr/bin/iscsiadm glibc_libs.source=/lib/x86_64-linux-gnu
docker plugin enable nimble
systemctl daemon-reload
systemctl restart open-iscsi multipath-tools
```

**NOTE:** To use the plugin on Fibre Channel environments use the `PROTOCOL=FC` environment variable.

### Making changes
The `docker plugin set` command can only be used on the plugin if it is disabled. To disable the plugin, use the `docker plugin disable` command. For example:

```
docker plugin disable nimble
```

List of parameters which are supported to be settable by the plugin.

|  Parameter              |  Description                                                |  Default |
|-------------------------|-------------------------------------------------------------|----------|
| `PROVIDER_IP`           | HPE Nimble Storage array ip                                 |`""`      |
| `PROVIDER_USERNAME`     | HPE Nimble Storage array username                           |`""`      |
| `PROVIDER_PASSWORD`     | HPE Nimble Storage array password                           |`""`      |
| `PROVIDER_REMOVE`       | Unassociate Plugin from HPE Nimble Storage array            | `false`  |
| `LOG_LEVEL`             | Log level of the plugin (`info`, `debug`, or `trace`)       | `debug`  |
| `SCOPE`                 | Scope of the plugin (`global` or `local`)                   | `global` |
| `PROTOCOL`              | Scsi protocol supported by the plugin (`iscsi` or `fc`)     | `iscsi`  |

## Security considerations
The HPE Nimble Storage credentials are visible to any user who can execute `docker plugin inspect nimble`. To limit credential visibility, the variables should be unset after certificates have been generated. The following set of steps can be used to accomplish this:

Add the credentials

```shell
docker plugin set PROVIDER_IP=192.168.1.1 PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin
```

Start the plugin

```shell
docker plugin enable nimble
```

Stop the plugin

```shell
docker plugin disable nimble
```

Remove the credentials

```shell
docker plugin set nimble PROVIDER_USERNAME="true" PROVIDER_PASSWORD="true"
```

Start the plugin

```shell
docker plugin enable nimble
```

!!! note
    Certificates are stored in `/etc/hpe-storage/` on the host and will be preserved across plugin updates.

In the event of reassociating the plugin with a different HPE Nimble Storage group, certain procedures need to be followed:

Disable the plugin

```shell
docker plugin disable nimble
```

Set new paramters

```shell
docker plugin set nimble PROVIDER_REMOVE=true
```

Enable the plugin

```shell
docker plugin enable nimble
```

Disable the plugin

```shell
docker plugin disable nimble
```

The plugin is now ready for re-configuration

```shell
docker plugin set nimble PROVIDER_IP=< New IP address > PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin PROVIDER_REMOVE=false
```

**Note:** The `PROVIDER_REMOVE=false` parameter must be set if the plugin ever has been unassociated from a HPE Nimble Storage group.

## Configuration files and options
The configuration directory for the plugin is `/etc/hpe-storage` on the host. Files in this directory are preserved between plugin upgrades. The `/etc/hpe-storage/volume-driver.json` file contains three sections, `global`, `defaults` and `overrides`. The global options are plugin runtime parameters and doesn't have any end-user configurable keys at this time.

The `defaults` map allows the docker host administrator to set default options during volume creation. The docker user may override these default options with their own values for a specific option.

The `overrides` map allows the docker host administrator to enforce a certain option for every volume creation. The docker user may not override the option and any attempt to do so will be silently ignored.

These maps are essential to discuss with the HPE Nimble Storage administrator. A common pattern is that a default protection template is selected for all volumes to fulfill a certain data protection policy enforced by the business it's serving. Another useful option is to override the volume placement options to allow a single HPE Nimble Storage array to provide multi-tenancy for docker environments.

**Note:** `defaults` and `overrides` are dynamically read during runtime while `global` changes require a plugin restart.

Below is an example `/etc/hpe-storage/volume-driver.json` outlining the above use cases:

```json
{
  "global": {
    "nameSuffix": ".docker"
  },
  "defaults": {
    "description": "Volume provisioned by Docker",
    "protectionTemplate": "Retain-90Daily"
  },
  "overrides": {
    "folder": "docker-prod"
  }
}
```

For an exhaustive list of options use the `help` option from the docker CLI:

```markdown
$ docker volume create -d nimble -o help
Nimble Storage Docker Volume Driver: Create Help
Create or Clone a Nimble Storage backed Docker Volume or Import an existing
Nimble Volume or Clone of a Snapshot into Docker.

Universal options:
  -o mountConflictDelay=X X is the number of seconds to delay a mount request
                           when there is a conflict (default is 0)

Create options:
  -o sizeInGiB=X          X is the size of volume specified in GiB
  -o size=X               X is the size of volume specified in GiB (short form
                          of sizeInGiB)
  -o fsOwner=X            X is the user id and group id that should own the
                           root directory of the filesystem, in the form of
                           [userId:groupId]
  -o fsMode=X             X is 1 to 4 octal digits that represent the file
                           mode to be applied to the root directory of the
                           filesystem
  -o description=X        X is the text to be added to volume description
                          (optional)
  -o perfPolicy=X         X is the name of the performance policy (optional)
                          Performance Policies: Exchange 2003 data store,
                          Exchange log, Exchange 2007 data store,
                          SQL Server, SharePoint,
                          Exchange 2010 data store, SQL Server Logs,
                          SQL Server 2012, Oracle OLTP,
                          Windows File Server, Other Workloads,
                          DockerDefault, General, MariaDB,
                          Veeam Backup Repository,
                          Backup Repository

  -o pool=X               X is the name of pool in which to place the volume
                          Needed with -o folder (optional)
  -o folder=X             X is the name of folder in which to place the volume
                          Needed with -o pool (optional).
  -o encryption           indicates that the volume should be encrypted
                          (optional, dedupe and encryption are mutually
                          exclusive)
  -o thick                indicates that the volume should be thick provisioned
                          (optional, dedupe and thick are mutually exclusive)
  -o dedupe               indicates that the volume should be deduplicated
  -o limitIOPS=X          X is the IOPS limit of the volume. IOPS limit should
                          be in range [256, 4294967294] or -1 for unlimited.
  -o limitMBPS=X          X is the MB/s throughput limit for this volume. If
                          both limitIOPS and limitMBPS are specified, limitMBPS
                          must not be hit before limitIOPS
  -o destroyOnRm          indicates that the Nimble volume (including
                          snapshots) backing this volume should be destroyed
                          when this volume is deleted
  -o syncOnUnmount        only valid with "protectionTemplate", if the
                          protectionTemplate includes a replica destination,
                          unmount calls will snapshot and transfer the last
                          delta to the destination. (optional)
 -o protectionTemplate=X  X is the name of the protection template (optional)
                          Protection Templates: General, Retain-90Daily,
                          Retain-30Daily,
                          Retain-48Hourly-30Daily-52Weekly

Clone options:
  -o cloneOf=X            X is the name of Docker Volume to create a clone of
  -o snapshot=X           X is the name of the snapshot to base the clone on
                          (optional, if missing, a new snapshot is created)
  -o createSnapshot       indicates that a new snapshot of the volume should be
                          taken and used for the clone (optional)
  -o destroyOnRm          indicates that the Nimble volume (including
                          snapshots) backing this volume should be destroyed
                          when this volume is deleted
  -o destroyOnDetach      indicates that the Nimble volume (including
                          snapshots) backing this volume should be destroyed
                          when this volume is unmounted or detached

Import Volume options:
  -o importVol=X          X is the name of the Nimble Volume to import
  -o pool=X               X is the name of the pool in which the volume to be
                          imported resides (optional)
  -o folder=X             X is the name of the folder in which the volume to be
                          imported resides (optional)
  -o forceImport          forces the import of the volume.  Note that
                          overwrites application metadata (optional)
  -o restore              restores the volume to the last snapshot taken on the
                          volume (optional)
  -o snapshot=X           X is the name of the snapshot which the volume will
                          be restored to, only used with -o restore (optional)
  -o takeover             indicates the current group will takeover the
                          ownership of the Nimble volume and volume collection
                          (optional)
  -o reverseRepl          reverses the replication direction so that writes to
                          the Nimble volume are replicated back to the group
                          where it was replicated from (optional)

Import Clone of Snapshot options:
  -o importVolAsClone=X   X is the name of the Nimble Volume and Nimble
                          Snapshot to clone and import
  -o snapshot=X           X is the name of the Nimble snapshot to clone and
                          import (optional, if missing, will use the most
                          recent snapshot)
  -o createSnapshot       indicates that a new snapshot of the volume should be
                          taken and used for the clone (optional)
  -o pool=X               X is the name of the pool in which the volume to be
                          imported resides (optional)
  -o folder=X             X is the name of the folder in which the volume to be
                          imported resides (optional)
  -o destroyOnRm          indicates that the Nimble volume (including
                          snapshots) backing this volume should be destroyed
                          when this volume is deleted
  -o destroyOnDetach      indicates that the Nimble volume (including
                          snapshots) backing this volume should be destroyed
                          when this volume is unmounted or detached
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
These are some basic examples on how to use the HPE Nimble Storage Volume Plugin for Docker.

### Create a Docker Volume
Using `docker volume create`.

!!! note
    The plugin applies a set of default options when you `create` new volumes unless you override them using the `volume create -o key=value` option flags.

Create a Docker volume with a custom description:

```shell
docker volume create -d nimble -o description="My volume description" --name myvol1 
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
docker volume create -d nimble -o cloneOf=myvol1 --name=myvol1-clone
```

(Optional) Select a snapshot on which to base the clone.

```shell
docker volume create -d nimble -o snapshot=mysnap1 -o cloneOf=myvol1 --name=myvol2-clone
```

### Provisioning Docker Volumes
There are several ways to provision a Docker volume depending on what tools are used:

- Docker Engine (CLI)
- Docker Compose file with either Docker UCP or Docker Engine

The Docker Volume plugin leverages the existing Docker CLI and APIs, therefor all native Docker tools may be used to provision a volume.

!!! note
    The plugin applies a set of default volume create options. Unless you override the default options using the volume option flags, the defaults are applied when you create volumes. For example, the default volume size is 10GiB.  
Config file `volume-driver.json`, which is stored at `/etc/hpe-storage/volume-driver.json:`

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

### Import a volume to Docker

**Before you begin**
Take the volume you want to import offline before importing it. For information about how to take a volume offline, refer to either the `CLI Administration Guide` or the `GUI Administration Guide` on [HPE InfoSight](https://infosight.hpe.com). Use the `create` command with the `importVol` option to import an HPE Nimble Storage volume to Docker and name it.

Import the HPE Nimble Storage volume named `mynimblevol` as a Docker volume named `myvol3-imported`.

```shell
docker volume create –d nimble -o importVol=mynimblevol --name=myvol3-imported
```

### Import a volume snapshot to Docker

Use the create command with the `importVolAsClone` option to import a HPE Nimble Storage volume snapshot as a Docker volume. Optionally, specify a particular snapshot on the HPE Nimble Storage volume using the snapshot option.

Import the HPE Nimble Storage snapshot `aSnapshot` on the volume `importMe` as a Docker volume named `importedSnap`.

```shell
docker volume create -d nimble -o importVolAsClone=mynimblevol -o snapshot=mysnap1 --name=myvol4-clone
```

!!! note
    If no snapshot is specified, the latest snapshot on the volume is imported.

### Restore an offline Docker Volume with specified snapshot
It's important that the volume to be restored is in an offline state on the array.

If the volume snapshot is not specified, the last volume snapshot is used.

```shell
docker volume create -d nimble -o importVol=myvol1.docker -o forceImport -o restore -o snapshot=mysnap1 --name=myvol1-restored
```

### List volumes
List Docker volumes.

```shell
docker volume ls
DRIVER                     VOLUME NAME
nimble:latest              myvol1
nimble:latest              myvol1-clone
```

### Remove a Docker Volume
When you remove volumes from Docker control they are set to the offline state on the array. Access to the volumes and related snapshots using the Docker Volume plugin can be reestablished.

!!! note
    To delete volumes from the HPE Nimble Storage array using the remove command, the volume should have been created with a `-o destroyOnRm` flag.

**Important:** Be aware that when this option is set to true, volumes and all related snapshots are deleted from the group, and can no longer be accessed by the Docker Volume plugin.

Remove the volume named `myvol1`.

```shell
docker volume rm myvol1
```

## Uninstall
The plugin can be removed using the `docker plugin rm` command. This command will not remove the configuration directory (`/etc/hpe-storage/`).

```shell
docker plugin rm nimble
```

!!! important
    If this is the last plugin to reference the Nimble Group and to completely remove the configuration directory, follow the steps as below

```shell
docker plugin set nimble PROVIDER_REMOVE=true
docker plugin enable nimble
docker plugin rm nimble
```

## Troubleshooting

The config directory is at `/etc/hpe-storage/`. When a plugin is installed and enabled, the Nimble Group certificates are created in the config directory.

```shell
ls -l /etc/hpe-storage/
total 16
-r-------- 1 root root 1159 Aug  2 00:20 container_provider_host.cert
-r-------- 1 root root 1671 Aug  2 00:20 container_provider_host.key
-r-------- 1 root root 1521 Aug  2 00:20 container_provider_server.cert
```

Additionally there is a config file `volume-driver.json` present at the same location. This file can be edited
to set default parameters for create volumes for docker.

#### Log file location

The docker plugin logs are located at `/var/log/hpe-docker-plugin.log`

## Upgrade from older plugins

Upgrading from 2.5.1 or older plugins, please follow the below steps

Ubuntu 16.04 LTS and Ubuntu 18.04 LTS:

```shell
docker plugin disable nimble:latest –f
docker plugin upgrade --grant-all-permissions  nimble store/hpestorage/nimble:3.0.0 --skip-remote-check
docker plugin set nimble PROVIDER_IP=192.168.1.1 PROVIDER_USERNAME=admin PROVIDER_PASSWORD=admin glibc_libs.source=/lib/x86_64-linux-gnu
docker plugin enable nimble:latest
```

Red Hat 7.5+, CentOS 7.5+, Oracle Enterprise Linux 7.5+ and Fedora 28+:

```shell
docker plugin disable nimble:latest –f
docker plugin upgrade --grant-all-permissions  nimble store/hpestorage/nimble:3.0.0 --skip-remote-check
docker plugin enable nimble:latest
```

!!! important
    In Swarm Mode, drain the existing running containers to the node where the plugin is upgraded.
