# Unsupported Releases

HPE supports up to three historic minor releases on supported versions of upstream and downstream Kubernetes. Releases are kept here for historic purposes.

[TOC]

#### HPE CSI Driver for Kubernetes 2.4.0

Release highlights:

* Kubernetes 1.27 and 1.28 support
* KubeVirt and OpenShift Virtualization support for Nimble/Alletra 5000/6000
* Enhanced scheduling for the NFS Server Provisioner
* Multiarch images (Linux ARM64/AMD64) for the CSI driver components and Alletra 9000 CSP
* Major updates to SIG Storage images

Upgrade considerations:

* Existing claims provisioned with the NFS Server Provisioner [needs to be upgraded](operations.md#upgrade_to_v240).

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.25-1.28<sup>1</sup></td>
  </tr>
  <tr>
    <th>Helm Chart</th>
    <td><a href="https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver/2.4.0">v2.4.0</a> on ArtifactHub</td>
  </tr>
  <tr>
    <th>Operators</th>
    <td>
     <a href="https://operatorhub.io/operator/hpe-csi-operator/stable/hpe-csi-operator.v2.4.0">v2.4.0</a> on OperatorHub<br />
     <a href="https://catalog.redhat.com/software/container-stacks/detail/5e9874643f398525a0ceb004">v2.4.0</a> via OpenShift console
    </td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>
      RHEL<sup>2</sup> 7.x, 8.x, 9.x, RHCOS 4.12-4.14<br />
      Ubuntu 16.04, 18.04, 20.04, 22.04<br />
      SLES 15 SP3, SP4, SP5
  </tr>
  <tr>
    <th>Platforms<sup>3</sup></th>
    <td>
      <!-- Alletra Storage MP (block) 10.2.x<br /-->
      Alletra OS 9000 9.3.x - 9.5.x<br />
      Alletra OS 5000/6000 6.0.0.x - 6.1.1.x<br />
      Nimble OS 5.0.10.x, 5.2.1.x, 6.0.0.x, 6.1.2.x<br />
      Primera OS 4.3.x - 4.5.x<br />
      3PAR OS 3.3.x
    </td>
  </tr>
  <tr>
    <th>Data&nbsp;protocols</th>
    <td>Fibre Channel, iSCSI</td>
  </tr>
  <tr>
    <th>Filesystems</th>
    <td>XFS, ext3/ext4, btrfs, NFSv4<sup>&ast;</sup></td>
  </tr>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v2.4.0.md>v2.4.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/around-the-storage-block/introduction-to-new-workload-paradigms-with-hpe-csi-driver-for/ba-p/7196554">Introduction to new workload paradigms with HPE CSI Driver for Kubernetes</a>
   </td>
 </tr>
</table>

<small>
 <sup>&ast;</sup> = The HPE CSI Driver for Kubernetes is a block storage driver primarily. It includes an [NFS Server Provisioner](using.md#using_the_nfs_server_provisioner) that allows "ReadWriteMany" `PersistentVolumeClaims`.<br/>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, SUSE Rancher, Mirantis Kubernetes Engine and others; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations. Lowest tested and known working version is Kubernetes 1.21.<br />
 <sup>2</sup> = The HPE CSI Driver will recognize CentOS, AlmaLinux and Rocky Linux as RHEL derives and they are supported by HPE.<br/>
 <sup>3</sup> = Learn about each data platform's team [support commitment](../legal/support/index.md#container_storage_providers).<br/>
</small>


#### HPE CSI Driver for Kubernetes 2.3.0

Release highlights:

* Introducing HPE Alletra 5000
* Security updates
* Support for Kubernetes 1.25-1.26 and Red Hat OpenShift 4.11-4.12
* Support for SLES 15 SP4, RHEL 9.1 and Ubuntu 22.04

Upgrade considerations:

* Existing claims provisioned with the NFS Server Provisioner [needs to be upgraded](operations.md#upgrade_to_v230).

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.23-1.26<sup>1</sup></td>
  </tr>
  <tr>
    <th>Helm Chart</th>
    <td><a href="https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver/2.3.0">v2.3.0</a> on ArtifactHub</td>
  </tr>
  <tr>
    <th>Operators</th>
    <td>
     <a href="https://operatorhub.io/operator/hpe-csi-operator/stable/hpe-csi-operator.v2.3.0">v2.3.0</a> on OperatorHub<br />
     v2.3.0</a> via OpenShift console
    </td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>
      RHEL<sup>2</sup> 7.x, 8.x, 9.x, RHCOS 4.10-4.12<br />
      Ubuntu 16.04, 18.04, 20.04, 22.04<br />
      SLES 15 SP2, SP3, SP4
  </tr>
  <tr>
    <th>Platforms<sup>3</sup></th>
    <td>
      Alletra OS 5000/6000 6.0.0.x - 6.1.1.x<br />
      Alletra OS 9000 9.3.x - 9.5.x<br />
      Nimble OS 5.0.10.x, 5.2.1.x, 6.0.0.x, 6.1.1.x<br />
      Primera OS 4.3.x - 4.5.x<br />
      3PAR OS 3.3.x
    </td>
  </tr>
  <tr>
    <th>Data&nbsp;protocol</th>
    <td>Fibre Channel, iSCSI</td>
  </tr>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v2.3.0.md>v2.3.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/around-the-storage-block/support-and-security-updates-for-hpe-csi-driver-for-kubernetes/ba-p/7184293">Support and security updates for HPE CSI Driver for Kubernetes</a> (release blog)
   </td>
 </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, SUSE Rancher, Mirantis Kubernetes Engine and others; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations. Lowest tested and known working version is Kubernetes 1.21.<br />
 <sup>2</sup> = The HPE CSI Driver will recognize CentOS, AlmaLinux and Rocky Linux as RHEL derives and they are supported by HPE.
 <sup>3</sup> = Learn about each data platform's team [support commitment](../legal/support/index.md#container_storage_providers).
</small>

#### HPE CSI Driver for Kubernetes 2.2.0

Release highlights:

* Support for Kubernetes 1.24 and Red Hat OpenShift 4.10
* Added Tolerations, Affinity, Labels and Node Selectors to Helm chart
* Improved automatic recovery for the NFS Server Provisioner
* Added multipath handling for Alletra 9000, Primera and 3PAR
* Volume expansion of encrypted volumes

Upgrade considerations:

* Existing [encrypted volumes needs to be migrated](operations.md#migrate_encrypted_volumes) to allow expansion
* Existing claims provisioned with the NFS Server Provisioner [needs to be upgraded](operations.md#upgrade_to_v220).

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.21-1.24<sup>1</sup></td>
  </tr>
  <tr>
    <th>Helm Chart</th>
    <td><a href="https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-driver/2.2.0">v2.2.0</a> on ArtifactHub</td>
  </tr>
  <tr>
    <th>Operators</th>
    <td>
     <a href="https://operatorhub.io/operator/hpe-csi-operator/stable/hpe-csi-operator.v2.2.1">v2.2.1</a> on OperatorHub<br />
     v2.2.1</a> via OpenShift console
    </td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>
      RHEL<sup>2</sup> 7.x & 8.x, RHCOS 4.8 & 4.10<br />
      Ubuntu 16.04, 18.04 & 20.04<br />
      SLES 15 SP2
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      Alletra OS 6000 6.0.0.x - 6.1.0.x<br />
      Alletra OS 9000 9.3.x - 9.5.x<br />
      Nimble OS 5.0.10.x, 5.2.1.x, 6.0.0.x, 6.1.0.x<br />
      Primera OS 4.3.x - 4.5.x<br />
      3PAR OS 3.3.x
    </td>
  </tr>
  <tr>
    <th>Data&nbsp;protocol</th>
    <td>Fibre Channel, iSCSI</td>
  </tr>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v2.2.0.md>v2.2.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/Around-the-Storage-Block/Updates-and-Improvements-to-HPE-CSI-Driver-for-Kubernetes/ba-p/7170878">Updates and Improvements to HPE CSI Driver for Kubernetes</a> (release blog)
   </td>
 </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, SUSE Rancher, Mirantis Kubernetes Engine and others; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations.<br />
 <sup>2</sup> = The HPE CSI Driver will recognize CentOS, AlmaLinux and Rocky Linux as RHEL derives and they are supported by HPE.
</small>

#### HPE CSI Driver for Kubernetes 2.1.1

Release highlights:

* Support for Kubernetes 1.23
* Upstream CSI sidecar updates
* Improved LUN discoverability in certain environments

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.20-1.23<sup>1</sup></td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>CentOS and RHEL 7.x & 8.x, RHCOS 4.6 & 4.8, Ubuntu 18.04 & 20.04, SLES 15 SP2
  </tr>
  <tr>
    <th>Data&nbsp;protocol</th>
    <td>Fibre Channel, iSCSI</td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      Alletra OS 6000 6.0.0.x<br />
      Alletra OS 9000 9.4.x<br />
      Nimble OS 5.0.10.x, 5.1.4.200-x, 5.2.1.x, 5.3.0.x, 5.3.1.x, 6.0.0.x<br />
      Primera OS 4.3.x, 4.4.x<br />
      3PAR OS 3.3.2
    </td>
  </tr>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v2.1.1.md>v2.1.1</a> on GitHub</td>
  </tr>
  <!--
  <tr>
   <th>Blogs</th>
   <td>
    <a href=""></a> ()<br />
    <a href=""></a> ()
   </td>
 </tr>
 -->
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, Rancher and Mirantis Kubernetes Engine; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations.
</small>

#### HPE CSI Driver for Kubernetes 2.1.0

Release highlights:

* Prometheus exporters
* Support for Red Hat OCP 4.8
* Support for Kubernetes 1.22
* Reliability/Stability enhancements
    * Peer Persistence Remote Copy enhancements
    * Volume Mutator enhancements
    * Logging enhancements

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.20-1.22<sup>1</sup></td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>CentOS and RHEL 7.x & 8.x, RHCOS 4.6 & 4.8, Ubuntu 18.04 & 20.04, SLES 15 SP2
  </tr>
  <tr>
    <th>Data&nbsp;protocol</th>
    <td>Fibre Channel, iSCSI</td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      Alletra OS 6000 6.0.0.x<br />
      Alletra OS 9000 9.3.x, 9.4.x<br />
      Nimble OS 5.0.10.x, 5.1.4.200-x, 5.2.1.x, 5.3.0.x, 5.3.1.x, 6.0.0.x<br />
      Primera OS 4.0.x, 4.1.x, 4.2.x, 4.3.x, 4.4.x<br />
      3PAR OS 3.3.1, 3.3.2
    </td>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v2.1.0.md>v2.1.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/Around-the-Storage-Block/HPE-CSI-Driver-for-Kubernetes-enhancements-with-monitoring-and/ba-p/7158137">HPE CSI Driver enhancements with monitoring and alerting</a> (release blog)<br />
    <a href="https://developer.hpe.com/blog/get-started-with-prometheus-and-grafana-on-docker-with-hpe-storage-array-exporter/">Get started with Prometheus and Grafana and HPE Storage Array Exporter</a> (tutorial)
   </td>
 </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, Rancher and Mirantis Kubernetes Engine; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations.
</small>

#### HPE CSI Driver for Kubernetes 2.0.0

Release highlights:

* Support for HPE Alletra 5000/6000 and 9000
* Host-based volume encryption
* Multitenancy for HPE Alletra 5000/6000 and Nimble Storage 

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.18-1.21<sup>1</sup></td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>CentOS and RHEL 7.x & 8.x, RHCOS 4.6, Ubuntu 18.04 & 20.04, SLES 15 SP2
  </tr>
  <tr>
    <th>Data&nbsp;protocol</th>
    <td>Fibre Channel, iSCSI</td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      Alletra OS 6000 6.0.0.x<br />
      Alletra OS 9000 9.3.0<br />
      Nimble OS 5.0.10.x,  5.1.4.200-x, 5.2.1.x, 5.3.0.x, 5.3.1.x, 6.0.0.x<br />
      Primera OS 4.0.x, 4.1.x, 4.2.x, 4.3.x<br />
      3PAR OS 3.3.1, 3.3.2
    </td>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v2.0.0.md>v2.0.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/Around-the-Storage-Block/HPE-CSI-Driver-for-Kubernetes-now-available-for-HPE-Alletra/ba-p/7136280">HPE CSI Driver for Kubernetes now available for HPE Alletra</a> (release blog)<br />
    <a href="https://developer.hpe.com/blog/multitenancy-for-kubernetes-clusters-using-hpe-alletra-6000-and-nimble-storage/">Multitenancy for Kubernetes clusters using HPE Alletra 5000/6000 and Nimble</a> (tutorial)<br />
    <a href="https://developer.hpe.com/blog/host-based-volume-encryption-with-hpe-csi-driver-for-kubernetes/">Host-based Volume Encryption with HPE CSI Driver for Kubernetes</a> (tutorial)
   </td>
 </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, Rancher and Mirantis Kubernetes Engine; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations.
</small>

#### HPE CSI Driver for Kubernetes 1.4.0

Release highlights:

* Kubernetes CSI Sidecars: Volume Group Provisioner and Volume Group Snapshotter
* NFS Server Provisioner GA
* HPE Primera Remote Copy Peer Persistence support
* Air-gap support for the Helm chart

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.17-1.20<sup>1</sup></td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>CentOS and RHEL 7.7 & 8.1, RHCOS 4.4 & 4.6, Ubuntu 18.04 & 20.04, SLES 15 SP1
  </tr>
  <tr>
    <th>Data&nbsp;protocol</th>
    <td>Fibre Channel, iSCSI </td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      NimbleOS 5.0.10.0-x, 5.1.4.200-x, 5.2.1.0-x, 5.3.0.0-x, 5.3.1.0-x<br />
      3PAR OS 3.3.1+<br />
      Primera OS 4.0+<br />
    </td>
  <tr>
    <th>Release&nbsp;notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v1.4.0.md>v1.4.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/Around-the-Storage-Block/HPE-CSI-Driver-for-Kubernetes-v1-4-0-with-expanded-ecosystem-and/ba-p/7118180">HPE CSI Driver for Kubernetes v1.4.0 now available!</a> (release blog) <br />
    <a href="https://developer.hpe.com/blog/7mBn6Yj89Wcg6VN6lzMO/synchronized-volume-snapshots-for-distributed-workloads-on-kubernetes">Synchronized Volume Snapshots for Distributed Workloads on Kubernetes</a> (tutorial)
   </td>
 </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, Rancher and Mirantis Kubernetes Engine; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations.
</small>

#### HPE CSI Driver/Operator for Kubernetes 1.3.0

Release highlights:

* Kubernetes CSI Sidecar: Volume Mutator
* Broader ecosystem support
* Native iSCSI CHAP configuration

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.15-1.18<sup>1</sup></td>
  </tr>
  <tr>
    <th>Worker OS</th>
    <td>CentOS 7.6, RHEL 7.6, RHCOS 4.3-4.4, Ubuntu 18.04, Ubuntu 20.04
  </tr>
  <tr>
    <th>Data protocol</th>
    <td>Fibre Channel, iSCSI </td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      NimbleOS 5.0.10.x, 5.1.4.200-x, 5.2.1.x, 5.3.0.x<br />
      3PAR OS 3.3.1<br/>
      Primera OS 4.0.0, 4.1.0, 4.2.0<sup>2</sup><br/>
    </td>
  <tr>
    <th>Release notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v1.3.0.md>v1.3.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td>
    <a href="https://community.hpe.com/t5/around-the-storage-block/hpe-csi-driver-for-kubernetes-1-3-0-now-available/ba-p/7099684">Around The Storage Block</a> (release)<br/>
    <a href="https://developer.hpe.com/blog/ppPAlQ807Ah8QGMNl1YE/tutorial-enabling-remote-copy-using-the-hpe-csi-driver-for-kubernetes-on">HPE DEV</a> (Remote copy peer persistence tutorial)<br/>
    <a href="https://developer.hpe.com/blog/8nlLVWP1RKFROlvZJDo9/introducing-kubernetes-csi-sidecar-containers-from-hpe">HPE DEV</a> (Introducing the volume mutator)<br/>
   </td>
 </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Container Platform and Rancher; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](partners/index.md) for other variations.<br />
 <sup>2</sup> = Only FC is supported on Primera OS prior to 4.2.0.
</small>

#### HPE CSI Driver for Kubernetes 1.2.0

Release highlights: Support for raw block volumes and inline ephemeral volumes. NFS Server Provisioner in Tech Preview (beta).

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.14-1.18</td>
  </tr>
  <tr>
    <th>Worker OS</th>
    <td>CentOS 7.6, RHEL 7.6, RHCOS 4.2-4.3, Ubuntu 16.04, Ubuntu 18.04
  </tr>
  <tr>
    <th>Data protocol</th>
    <td>Fibre Channel, iSCSI </td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      NimbleOS 5.0.10.x, 5.1.3.1000-x, 5.1.4.200-x, 5.2.1.x<br />
      3PAR OS 3.3.1<br/>
      Primera OS 4.0.0, 4.1.0 (FC only)<br/>
    </td>
  <tr>
    <th>Release notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v1.2.0.md>v1.2.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td><a href="https://community.hpe.com/t5/around-the-storage-block/hpe-csi-driver-for-kubernetes-1-2-0-available-now/ba-p/7091977">Around The Storage Block</a> (release)<br/>
       <a href="https://developer.hpe.com/blog/EE2QnZBXXwi4o7X0E4M0/using-raw-block-and-ephemeral-inline-volumes-on-kubernetes">HPE DEV</a> (tutorial for raw block and inline volumes)<br/>
       <a href="https://community.hpe.com/t5/around-the-storage-block/tech-preview-network-file-system-server-provisioner-for-hpe-csi/ba-p/7092948">Around The Storage Block</a> (NFS Server Provisioner)<br/>
       <a href="https://developer.hpe.com/blog/xABwJY56qEfNGMEo1lDj/introducing-a-nfs-server-provisioner-and-pod-monitor-for-the-hpe-csi-dri">HPE DEV</a> (tutorial for NFS)
   </td>
 </tr>
</table>

#### HPE CSI Driver for Kubernetes 1.1.1

Release highlights: Support for HPE 3PAR and Primera Container Storage Provider.

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.13-1.17</td>
  </tr>
  <tr>
    <th>Worker OS</th>
    <td>CentOS 7.6, RHEL 7.6, RHCOS 4.2-4.3, Ubuntu 16.04, Ubuntu 18.04
  </tr>
  <tr>
    <th>Data protocol</th>
    <td>Fibre Channel, iSCSI </td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      NimbleOS 5.0.8.x, 5.1.3.x, 5.1.4.x<br/>
      3PAR OS 3.3.1<br/>
      Primera OS 4.0.0, 4.1.0 (FC only)<br/>
    </td>
  <tr>
    <th>Release notes</th>
    <td>N/A</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td><a href="https://community.hpe.com/t5/hpe-storage-tech-insiders/hpe-csi-driver-for-kubernetes-1-1-1-and-hpe-3par-and-hpe-primera/ba-p/7086675">HPE Storage Tech Insiders</a> (release), <a href="https://developer.hpe.com/blog/9o7zJkqlX5cErkrzgopL/tutorial-how-to-get-started-with-the-hpe-csi-driver-and-hpe-primera-and-">HPE DEV</a> (tutorial for "primera3par" CSP)</td>
 </tr>
</table>

#### HPE CSI Driver for Kubernetes 1.1.0

Release highlights: Broader ecosystem support, official support for CSI snapshots and volume resize.

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.13-1.17</td>
  </tr>
  <tr>
    <th>Worker OS</th>
    <td>CentOS 7.6, RHEL 7.6, RHCOS 4.2-4.3, Ubuntu 16.04, Ubuntu 18.04
  </tr>
  <tr>
    <th>Data protocol</th>
    <td>Fibre Channel, iSCSI </td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>
      NimbleOS 5.0.8.x, 5.1.3.x, 5.1.4.x
    </td>
  </tr>
  <tr>
    <th>Release notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v1.1.0.md>v1.1.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td><a href=https://community.hpe.com/t5/HPE-Storage-Tech-Insiders/HPE-CSI-Driver-for-Kubernetes-1-1-0-Generally-Available/ba-p/7082995>HPE Storage Tech Insiders</a> (release), <a href=https://developer.hpe.com/blog/PklOy39w8NtX6M2RvAxW/hpe-csi-driver-for-kubernetes-snapshots-clones-and-volume-expansion>HPE DEV</a> (snapshots, clones, resize)</td>
 </tr>
</table>

#### HPE CSI Driver for Kubernetes 1.0.0

Release highlights: Initial GA release with support for Dynamic Provisioning.

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.13-1.17</td>
  </tr>
  <tr>
    <th>Worker OS</th>
    <td>CentOS 7.6, RHEL 7.6, Ubuntu 16.04, Ubuntu 18.04
  </tr>
  <tr>
    <th>Data protocol</th>
    <td>Fibre Channel, iSCSI </td>
  </tr>
  <tr>
    <th>Platforms</th>
    <td>NimbleOS 5.0.8.x, 5.1.3.x, 5.1.4.x</td>
  </tr>
  <tr>
    <th>Release notes</th>
    <td><a href=https://github.com/hpe-storage/csi-driver/blob/master/release-notes/v1.0.0.md>v1.0.0</a> on GitHub</td>
  </tr>
  <tr>
   <th>Blogs</th>
   <td><a href=https://community.hpe.com/t5/HPE-Storage-Tech-Insiders/HPE-CSI-Driver-for-Kubernetes-1-0-Released/ba-p/7076820>HPE Storage Tech Insiders</a> (release), <a href=https://developer.hpe.com/blog/n0J8kpk1DJf4y7xD2D4X/introducing-a-multi-vendor-csi-driver-for-kubernetes>HPE DEV</a> (architecture and introduction)</td>
 </tr>
</table>
