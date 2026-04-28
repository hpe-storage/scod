# Overview

*"Run AI infrastructure at scale. Without losing control. With our Palette management platform, we help enterprises and public sector organizations modernize and scale infrastructure for AI workloads — from core to cloud to edge, and metal to model." *<sup>1</sup>

<div align="right"><small><sup>1</sup> = quote from <a href="https://spectrocloud.com/">spectrocloud.com</a>.</small></div>
<br/>

[TOC]

## Supportability

HPE support any Spectro Cloud managed cluster that is comprised of a node OS that is supported by the HPE CSI Driver for Kubernetes. See [Compatibility & Support](../../index.md#latest_release) for details.

In addition, the custom immutable version of Ubuntu that Spectro Cloud uses for edge devices has been tested and verified by HPE. Ensure to be understood with the limitations below and the mandatory `user-data` stanzas during the ISO image build process using CanvOS to enable a successful deployment of the HPE CSI Driver for Kubernetes.

The HPE CSI Driver for Kubernetes is deployed through a [community pack](https://docs.spectrocloud.com/integrations/packs/?pack=hpe-csi-driver-for-kubernetes&tab=main) that is added to a Spectro Cloud cluster profile. Additional configuration is needed to enable a storage backend and create a `StorageClass` as highlighted in the [Create additional resources](#create_additional_resources) section.

## Limitations

Due to the minimization phase of the immutable Ubuntu image built with CanvOS, certain features can not be used by the CSI driver.

- NVMe/TCP is not supported due to missing user space tools.
- The xfsprogs package is not installed which disables all XFS filesystem support.
- The Spectro Cloud pack will be installed with `disableNodeConformance=true`, if installing on a supported mutable node, use `disableNodeConformance=false` to enable XFS and NVMe/TCP.
- Spectro Cloud edge nodes are deployed with very long names. These names are too long to be supported by any Alletra Storage MP B10000 pedigree platform. It's recommended to use the HPE provided `user-data` stanza to shorten the node name.
- The open-iscsi package is installed during the templating phase of the build process. This will result in duplicate IQN names and the CSI node driver will not start. It's recommended to use the HPE provided `user-data` stanza to use the shortened node UUID as the IQN identifier to avoid duplicate names.

## Building CanvOS edge ISOs

When building ISOs with the CanvOS utility provided by Spectro Cloud, a custom `user-data` file needs to be prepared to customize the image for the environment it's being deployed into and which Spectro Cloud tenant to connect to. The CSI driver require additional steps as highlighted in the limitations.

```yaml
#cloud-config
stylus:
  site:
    paletteEndpoint: my-org.console.spectrocloud.com
    edgeHostToken: <Your Spectro Cloud token>
    tags:
      city: "Houston"
    deviceUIDPaths:
      - name: /etc/palette/metadata-regex
        regex: "edge-[0-9a-f]+"
install:
  poweroff: true
stages:
  initramfs:
    - name: Create user and assign to sudo group
      users:
        kairos:
          groups:
            - sudo
          passwd: kairos
    - name: Shorten host UUID
      commands:
        - mkdir -p /etc/palette
        - sed -e 's/.*\(.\{12\}\)$/edge-\1/' /sys/class/dmi/id/product_uuid > /etc/palette/metadata-regex
    - name: Reset host IQN
      commands:
        - sed -e 's/.*\(.\{12\}\)$/InitiatorName=iqn.2016-04.com.open-iscsi:\1/' /sys/class/dmi/id/product_uuid > /etc/iscsi/initiatorname.iscsi
```

Learn more about the CanvOS utility and how to prepare `user-data` on [spectrocloud.com](https://docs.spectrocloud.com/clusters/edge/edgeforge-workflow/prepare-user-data/).

## Deployment Considerations

Besides adding the HPE CSI Driver for Kubernetes pack to the Spectro Cloud cluster profile, here are a few other deployment considerations.

### Using VolumeSnapshotClasses and VolumeSnapshots

It's recommended to deploy the "volume-snapshot-controller" pack provided by Spectro Cloud in order to use `VolumeSnapshotClasses` and `VolumeSnapshots`. Learn how to use [snapshots and clones](../../using#using_csi_snapshots) with the CSI driver.

### iSCSI Networking

Due to the dynamic nature of building and deploying edge nodes, it's recommended to run DHCP with persistent leases on the iSCSI data networks. These interfaces will then be ready to use immediately by the CSI driver once the cluster has been provisioned.

## Create additional resources

In order to start provisioning volumes from an HPE storage backend, a `StorageClass` and `Secret` need to be configured. These steps are outlined in the [deployment section](../../deployment.md#add_an_hpe_storage_backend) on SCOD.

!!! important
    When using CanvOS built edge images, make sure to create `StorageClasses` with `.parameters.csi.storage.k8s.io/fstype: ext4` (or ext3) set, as the default value of `xfs` is not supported on Ubuntu images built with CanvOS.
