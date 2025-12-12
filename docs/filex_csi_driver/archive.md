# Unsupported Releases

HPE supports up to three historic minor releases on supported versions of upstream and downstream Kubernetes. Releases are kept here for historic purposes.

[TOC]

#### HPE GreenLake for File Storage CSI Driver v1.0.0-beta3

Release highlights:

* Public beta release

<table>
  <tr>
    <th>Kubernetes</th>
    <td>1.28-1.32<sup>1</sup></td>
  </tr>
  <tr>
    <th>Helm Chart</th>
    <td><a href="https://artifacthub.io/packages/helm/hpe-storage/hpe-greenlake-file-csi-driver/1.0.0-beta3">v1.0.0-beta3</a> on ArtifactHub</td>
  </tr>
  <tr>
    <th>Operators</th>
    <td>
     <a href="https://catalog.redhat.com/software/container-stacks/detail/670e9898a6db4f1b3b89ff59">v1.0.0-beta3</a> via OpenShift console
    </td>
  </tr>
  <tr>
    <th>Worker&nbsp;OS</th>
    <td>
      Red Hat Enterprise Linux<sup>2</sup> 7.x, 8.x, 9.x, Red Hat CoreOS 4.14-4.17<br />
      Ubuntu 16.04, 18.04, 20.04, 22.04, 24.04<br />
      SUSE Linux Enterprise Server 15 SP4, SP5, SP6 and SLE Micro<sup>4</sup> equivalents
  </tr>
  <tr>
    <th>Platforms<sup>3</sup></th>
    <td>
      HPE GreenLake for File Storage MP OS 1.3 or later
    </td>
  </tr>
  <tr>
    <th>Data&nbsp;Protocols</th>
    <td>NFSv3 and NFSv4.1</td>
  </tr>
</table>

<small>
 <sup>1</sup> = For HPE Ezmeral Runtime Enterprise, SUSE Rancher, Mirantis Kubernetes Engine and others; Kubernetes clusters must be deployed within the currently supported range of "Worker OS" platforms listed in the above table. Lowest tested and known working version is Kubernetes 1.21.<br />
 <sup>2</sup> = The HPE CSI Driver will recognize CentOS, AlmaLinux and Rocky Linux as RHEL derives and they are supported by HPE. While RHEL 7 and its derives will work, the host OS have been EOL'd and support is limited.<br/>
 <sup>3</sup> = Learn about each data platform's team [support commitment](../legal/support/index.md).<br/>
 <sup>4</sup> = SLE Micro nodes may need to be conformed manually, run `transactional-update -n pkg install nfs-client` and reboot if the CSI node driver doesn't start.<br/>
</small>
