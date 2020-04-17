# Overview
TBD

[TOC]

## Platform requirements

### HPE 3PAR and Primera Storage Platform Requirements

|  HPE 3PAR and Primera CSP for Kubernetes  |  Docker EE  |  Linux OS  |  OpenShift  |  Kubernetes  | 3PAR and Primera  |
|-------------------------------------------|------------ |------------|-------------|--------------|-------------------|
|HPE 3PAR and HPE Primera CSP v.1.0.0|Based on Kubernetes version guidelines<br><br><ul><li>For K8s 1.17: Docker Version 19.03.4 is recommended</li><li>For K8s 1.16: Docker Version 18.06.2 is recommended</li></ul>|<ul><li>CentOS: 7.7</li><li>RHEL: 7.6, 7.7 / RHCOS</li></ul>|<ul><li>OpenShift 4.2 with RHEL 7.6 or 7.7 or RHCOS as worker nodes</li></ul>| K8s 1.16, 1.17 |<ul><li>3PAR 3.3.1 MU5 (FC & iSCSI)</li><li>Primera OS: 4.0.0, 4.1.0 (FC only)</li><ul>|

!!! Important
    * Minimum 2 iSCSI IP ports should be in ready state
    * FC array should be in ready state and zoned with initiator hosts

    **Note:** 3PAR supports FC and iSCSI, Primera supports FC protocol

## Deployemnt
TBD

### Deploying to Kubernetes
TBD

### Deploying to OpenShift
TBD

## Getting Started
TBD

## Diagnostics and Troubleshooting
TBD
