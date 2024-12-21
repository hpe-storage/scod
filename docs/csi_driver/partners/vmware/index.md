# VMware vSphere Container Storage Plug-in

VMware vSphere Container Storage Plug-in also known as the upstream vSphere CSI Driver exposes vSphere storage and features to Kubernetes users and was introduced in vSphere 6.7 U3. The term Cloud Native Storage (CNS) is the vCenter abstraction point and is made up of two parts, a Container Storage Interface (CSI) driver for Kubernetes used to provision storage on vSphere and the CNS Control Plane within vCenter allowing visibility to persistent volumes through the CNS UI within vCenter.

CNS fully supports Storage Policy-Based Management (SPBM) to provision volumes. SPBM is a feature of VMware vSphere that allows an administrator to match VM workload requirements against storage array capabilities, with the help of VM Storage Profiles. This storage profile can have multiple array capabilities and data services, depending on the underlying storage you use. HPE primary storage (HPE GreenLake for Block Storage, Primera, Nimble Storage, Nimble Storage dHCI, and 3PAR) has the largest user base of vVols in the market, due to its simplicity to deploy and ease of use.

[TOC]

### Feature Comparison

Volume parameters available to the vSphere Container Storage Plug-in will be dependent upon options exposed through the vSphere SPBM and may not include all volume features available. Please refer to the [HPE Primera: VMware ESXi Implementation Guide](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00001341en_us&docLocale=en_US&page=index.html) (includes HPE Alletra Storage MP B10000, Alletra 9000 and 3PAR) or [VMware vSphere Virtual Volumes on HPE Nimble Storage Implementation Guide](https://www.hpe.com/psnow/doc/a00044881enw) (includes HPE Alletra 5000/6000 and dHCI) for list of available features.

For a list of available volume parameters in the HPE CSI Driver for Kubernetes, refer to the respective [CSP](../../container_storage_provider/index.md).

| Feature                                                   | HPE CSI Driver | vSphere Container Storage Plug-in |
| --------------------------------------------------------- | -------------- | --------------------------------- |
| vCenter Cloud Native Storage (CNS) UI Support             | No             | GA                                |
| Dynamic Block PV Provisioning (ReadWriteOnce access mode) | GA             | GA (vVOL)                         |
| Dynamic File Provisioning (ReadWriteMany access mode)     | GA             | GA (vSan Only)                    |
| Volume Snapshots (CSI)                                    | GA             | GA (vSphere 7.0u3)                |
| Volume Cloning from VolumeSnapshot (CSI)                  | GA             | GA                                |
| Volume Cloning from PVC (CSI)                             | GA             | GA                                |
| Volume Expansion (CSI)                                    | GA             | GA (vSphere 7.0u2)                |
| RWO Raw Block Volume (CSI)                                | GA             | GA                                |
| RWX/ROX Raw Block Volume (CSI)                            | GA             | No                                |
| Generic Ephemeral Volumes (CSI)                           | GA             | GA                                |
| Inline Ephemeral Volumes (CSI)                            | GA             | No                                |
| Topology (CSI)                                            | No             | GA                                |
| Volume Health (CSI)                                       | No             | GA (vSan only)                    |
| CSI Controller multiple replica support                   | No             | GA                                |
| Windows support                                           | No             | GA                                |
| Volume Encryption                                         | GA             | GA (via VMcrypt)                  |
| Volume Mutator<sup>1</sup>                                | GA             | No                                |
| Volume Groups<sup>1</sup>                                 | GA             | No                                |
| Snapshot Groups<sup>1</sup>                               | GA             | No                                |
| Peer Persistence Replication<sup>3</sup>                  | GA             | No<sup>4</sup>                    |

<small>
 <sup>1</sup> = Feature comparison based upon HPE CSI Driver for Kubernetes 2.4.0 and the vSphere Container Storage Plug-in 3.1.2<br />
 <sup>2</sup> = HPE and VMware fully support features listed as GA for their respective CSI drivers.<br />
 <sup>3</sup> = The HPE Remote Copy Peer Persistence feature of the HPE CSI Driver for Kubernetes is only available with HPE Alletra Storage MP B10000, Alletra 9000, Primera and 3PAR storage systems.<br />
 <sup>4</sup> = Peer Persistence is an HPE Storage specific platform feature that isn't abstracted up to the vSphere Container Storage Plug-in. Peer Persistence works with the vSphere Container Storage Plug-in when using VMFS datastores.
</small>

Please refer to [Compatibility Matrices for vSphere Container Storage Plug-in](https://docs.vmware.com/en/VMware-vSphere-Container-Storage-Plug-in/3.0/vmware-vsphere-csp-getting-started/GUID-D4AAD99E-9128-40CE-B89C-AD451DA8379D.html) for the most up-to-date information.

### Important Considerations

The HPE CSI Driver for Kubernetes is only supported on specific versions of worker node operating systems and Kubernetes versions, these requirements applies to any worker VM running on vSphere.

Some Kubernetes distributions, when running on vSphere may *only* support the vSphere Container Storage Plug-in, such an example is VMware Tanzu. Ensure the Kubernetes distribution being used support 3rd party CSI drivers (such as the HPE CSI Driver) and fulfill the requirements in [Features and Capabilities](../../index.md#features_and_capabilities) before deciding which CSI driver to use.

HPE does not test or qualify the vSphere Container Storage Plug-in for any particular storage backend besides point solutions<sup>1</sup>. As long as the storage platform is supported by vSphere, VMware will [support](#support) the vSphere Container Storage Plug-in.

!!! notice "VMware vSphere with Tanzu and HPE Alletra dHCI<sup>1</sup>"
    HPE provides a turnkey solution for Kubernetes using VMware Tanzu and HPE Alletra dHCI. [Learn more](https://www.hpe.com/psnow/doc/a00130343enw).

### Deployment

When considering to use block storage within Kubernetes clusters running on VMware, customers need to evaluate which data protocol (FC or iSCSI) is primarily used within their virtualized environment. This will help best determine which CSI driver can be deployed within your Kubernetes clusters.

!!! error "Important"
    Due to limitations when exposing physical hardware (i.e. Fibre Channel Host Bus Adapters) to virtualized guest OSs and if iSCSI is not an available, HPE recommends the use of the VMware vSphere Container Storage Plug-in to deliver block-based persistent storage from HPE GreenLake for Block Storage, Alletra, Primera, Nimble Storage, Nimble Storage dHCI or 3PAR arrays to Kubernetes clusters within VMware environments for customers who are using the Fibre Channel protocol.

    The HPE CSI Driver for Kubernetes does not support N_Port ID Virtualization (NPIV).

| Protocol | HPE CSI Driver for Kubernetes | vSphere Container Storage Plug-in |
| -------- | :---------------------------: | :-------------------------------: |
| FC       | **Not** supported             | Supported<sup>&ast;</sup>         |
| NVMe-oF  | **Not** supported             | Supported<sup>&ast;</sup>         |
| iSCSI    | Supported                     | Supported<sup>&ast;</sup>         |

<small><sup>&ast;</sup> = Limited to the SPBM implementation of the underlying storage array.</small>

Learn how to deploy the vSphere Container Storage Plug-in:

- [Version 2.x](https://docs.vmware.com/en/VMware-vSphere-Container-Storage-Plug-in/2.0/vmware-vsphere-csp-getting-started/GUID-6DBD2645-FFCF-4076-80BE-AD44D7141521.html)<sup>1</sup> (Kubernetes 1.23-1.25)
- [Version 3.x](https://docs.vmware.com/en/VMware-vSphere-Container-Storage-Plug-in/3.0/vmware-vsphere-csp-getting-started/GUID-6DBD2645-FFCF-4076-80BE-AD44D7141521.html) (Kubernetes 1.24 onwards)

<small><sup>1</sup> = The HPE authored deployment guide for vSphere Container Storage Plug-in 2.4 has been [preserved here](legacy.md).</small>

!!! tip
    Most non-vanilla Kubernetes distributions when deployed on vSphere manage and support the vSphere Container Storage Plug-in directly. That includes Red Hat OpenShift, SUSE Rancher, Charmed Kubernetes (Canonical), Google Anthos and Amazon EKS Anywhere.

### Support

VMware provides enterprise grade support for the vSphere Container Storage Plug-in. Please use [VMware Support Services](https://www.vmware.com/support/file-sr.html) to file a customer support ticket to engage the VMware global support team.

For support information on the HPE CSI Driver for Kubernetes, visit [Support](../../../legal/support/index.md). For support with other HPE related technologies, visit the [Hewlett Packard Enterprise Support Center](https://support.hpe.com/).
