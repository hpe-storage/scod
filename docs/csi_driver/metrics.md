# HPE CSI Info Metrics Provider for Prometheus

The HPE CSI Driver for Kubernetes may be accompanied by a Prometheus metrics endpoint to provide metadata about the volumes provisioned by the CSI driver and supporting backends. It's conventionally deployed with [HPE Storage Array Exporter for Prometheus](https://hpe-storage.github.io/array-exporter) to provide a richer set of metrics from the backend storage systems.

[TOC]

## Metrics Provided

The exporter provides two metrics, "hpestoragecsi_volume_info" and "hpestoragecsi_backend_info".

### Volume Info

| Metric                    | Type    | Description                                                 | Value |
| :------------------------ | :------ | :---------------------------------------------------------- | :---- |
| hpestoragecsi_volume_info | Gauge   | Indicates a volume whose provisioner is the HPE CSI Driver. | 1     |

This metric includes the following labels.

| Label         | Description                                                |
| :------------ | :--------------------------------------------------------- |
| backend       | Backend hostname or IP address as defined in the `Secret`. |
| pv            | `PersistentVolume` name.                                   |
| pvc           | `PersistentVolumeClaim` name.                              |
| pvc_namespace | `PersistentVolumeClaim` `Namespace`.                       |
| storage_class | `StorageClass` used to provision the `PersistentVolume`.   |
| volume        | Volume handle used by the backend storage system.          | 

### Backend Info

| Metric                     | Type   | Description                                                               | Value |
| :------------------------- | :----- | :------------------------------------------------------------------------ | :---- |
| hpestoragecsi_backend_info | Gauge  | Indicates a storage system for which the HPE CSI driver is a provisioner. | 1     |

This metric includes the following labels.

| Label   | Description                                                |
| :------ | :--------------------------------------------------------- |
| backend | Backend hostname or IP address as defined in the `Secret`. |

## Deployment

The exporter may be installed either via Helm or through YAML manifests with the object definitions. It's recommended to use Helm as it's more convenient to manage the configuration of the deployment.

!!! note
    It's recommended to add a "cluster" target label to the deployment. The label is used in the [provided Grafana dashboards](https://grafana.com/orgs/hpestorage/dashboards).

### Helm

The Helm chart is available on Artifact Hub. Instructions on how to manage and install the chart is available within the chart documentation.

- [HPE CSI Info Metrics Provider for Prometheus Helm chart](https://artifacthub.io/packages/helm/hpe-storage/hpe-csi-info-metrics)

### Advanced Install

Before beginning an advanced install, determine how Prometheus will be deployed on the Kubernetes cluster as it will dictate how the scrape target will be configured with either a `Service` annotation or a `ServiceMonitor` CRD.

Start by downloading the manifest, which needs to be modified before applying to the cluster.

#### Version 1.0.0

Supports HPE CSI Driver for Kubernetes 2.0.0 and later.

```markdown
wget https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-info-metrics/v1.0.0/hpe-csi-info-metrics.yaml
```

Optional `ServiceMonitor` definition:

```markdown
wget https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-info-metrics/v1.0.0/hpe-csi-info-metrics-service-monitor.yaml
```

#### Configuring Advanced Install

Update the main container parameters and optionally add service labels and annotations.

In the "hpe-csi-info-metrics" `Deployment` at `.spec.template.spec.containers[0].args` in "hpe-csi-info-metrics.yaml":

```markdown
          args:
            - "--telemetry.addr=:9099"
            - "--telemetry.path=/metrics"
            # IMPORTANT: Uncomment this argument to confirm your
            # acceptance of the HPE End User License Agreement at
            # https://www.hpe.com/us/en/software/licensing.html
            #- "--accept-eula"
```

Remove the `#` in front of `--accept-eula` to accept the [HPE license restrictions](https://www.hpe.com/us/en/software/licensing.html).

In the "hpe-csi-info-metrics-service" `Service`:

```markdown
metadata:
  name: hpe-csi-info-metrics-service
  namespace: hpe-storage
  labels:
    app: hpe-csi-info-metrics
    # Optionally add labels, for example to be included in Prometheus
    # metrics via a targetLabels setting in a ServiceMonitor spec
    #cluster: my-cluster
  # Optionally add annotations, for example to configure it as a
  # scrape target when using the Prometheus Helm chart's default
  # configuration.
  #annotations:
  #  "prometheus.io/scrape": "true"
```

- Apply and uncomment any custom labels. It's recommended to use a "cluster" label to use the [provided Grafana dashboards](https://grafana.com/orgs/hpestorage/dashboards).
- If Prometheus has been deployed without the Operator, uncomment the annotation.

Apply the manifest:

```markdown
kubectl apply -f hpe-csi-info-metrics.yaml
```

Optionally, if using the Prometheus Operator, add any additional labels in "hpe-csi-info-metrics-service-monitor.yaml":

```markdown
  # Corresponding labels on the CSI Info Metrics service are added to
  # the scraped metrics
  #targetLabels:
  #  - cluster
```

Apply the manifest:

```markdown
kubectl apply -f hpe-csi-info-metrics-service-monitor.yaml
```

!!! warning "Pro Tip!"
    Avoid hand editing manifests by using the Helm chart.

## Grafana Dashboards

Example Grafana dashboards, provided as is, are hosted on [grafana.com](https://grafana.com/orgs/hpestorage/dashboards).
