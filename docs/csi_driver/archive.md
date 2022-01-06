# Unsupported Releases

HPE supports up to three minor releases. These release are kept here for historic purposes.

[TOC]

#### HPE CSI Driver for Kubernetes 1.3.0

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
 <sup>1</sup> = For HPE Ezmeral Container Platform and Rancher; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. See [partner ecosystems](../partners) for other variations.<br />
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
