# Introduction

It's recommended to familiarize yourself with inspecting workloads on Kubernetes. This particular [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods) is very useful to have readily available. 

## Sanity checks

Once the CSI driver has been deployed either through object configuration files, Helm or an Operator. This view should be representative of what a healthy system should look like after install. If any of the workload deployments lists anything but `Running`, proceed to inspect the logs of the problematic workload.

```markdown fct_label="HPE Nimble Storage"
kubectl get pods --all-namespaces -l 'app in (nimble-csp, hpe-csi-node, hpe-csi-controller)'
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
kube-system   hpe-csi-controller-7d9cd6b855-zzmd9   5/5     Running   0          15s
kube-system   hpe-csi-node-dk5t4                    2/2     Running   0          15s
kube-system   hpe-csi-node-pwq2d                    2/2     Running   0          15s
kube-system   nimble-csp-546c9c4dd4-5lsdt           1/1     Running   0          15s
```

```markdown fct_label="HPE 3PAR and Primera"
kubectl get pods --all-namespaces -l 'app in (primera3par-csp, hpe-csi-node, hpe-csi-controller)'
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
kube-system   hpe-csi-controller-7d9cd6b855-fqppd   5/5     Running   0          14s
kube-system   hpe-csi-node-86kh6                    2/2     Running   0          14s
kube-system   hpe-csi-node-k8p4p                    2/2     Running   0          14s
kube-system   hpe-csi-node-r2mg8                    2/2     Running   0          14s
kube-system   hpe-csi-node-vwb5r                    2/2     Running   0          14s
kube-system   primera3par-csp-546c9c4dd4-bcwc6      1/1     Running   0          14s
```

## ReadWriteMany resources

Foo

## Logging

Log files associated with the HPE CSI Driver logs data to the standard output stream. If the logs need to be retained for long term, use a standard logging solution for Kubernetes such as Fluentd. Some of the logs on the host are persisted which follow standard logrotate policies.

### CSI driver logs

Node driver
```
kubectl logs -f  daemonset.apps/hpe-csi-node  hpe-csi-driver -n kube-system
```
Controller driver
```
kubectl logs -f deployment.apps/hpe-csi-controller hpe-csi-driver -n kube-system
```

!!! tip
    The logs for both node and controller drivers are persisted at `/var/log/hpe-csi.log`

### Log level

Log levels for both CSI Controller and Node driver can be controlled using `LOG_LEVEL` environment variable. Possible values are `info`, `warn`, `error`, `debug`, and `trace`. Apply the changes using `kubectl apply -f <yaml>` command after adding this to CSI controller and node container spec as below. For Helm charts this is controlled through `logLevel` variable in `values.yaml`.

```markdown
          env:
            - name: LOG_LEVEL
              value: trace
```

### CSP logs

CSP logs can be accessed from their respective services.

```markdown fct_label="HPE Nimble Storage"
kubectl logs -f svc/nimble-csp-svc -n kube-system
```

```markdown fct_label="HPE 3PAR and Primera"
kubectl logs -f svc/primera3par-csp-svc -n kube-system
```

### Log collector

Log collector script `hpe-logcollector.sh` can be used to collect the logs from any node which has `kubectl` access to the cluster.

```markdown
curl -O https://raw.githubusercontent.com/hpe-storage/csi-driver/master/hpe-logcollector.sh
chmod 555 hpe-logcollector.sh
```

Usage:

```markdown
./hpe-logcollector.sh -h
Diagnostic Script to collect HPE Storage logs using kubectl

Usage:
     hpe-logcollector.sh [-h|--help][--node-name NODE_NAME][-n|--namespace NAMESPACE][-a|--all]
Where
-h|--help                  Print the Usage text
--node-name NODE_NAME      where NODE_NAME is kubernetes Node Name needed to collect the
                           hpe diagnostic logs of the Node
-n|--namespace NAMESPACE   where NAMESPACE is namespace of the pod deployment. default is kube-system
-a|--all                   collect diagnostic logs of all the nodes.If
                           nothing is specified logs would be collected
                           from all the nodes
```

## Tuning

HPE provides a set of well tested defaults for the CSI driver and all the supported CSPs. In certain case it may be necessary to fine tune the CSI driver to accommodate a certain workload or behavior. 

### Data path configuration

The HPE CSI Driver for Kubernetes automatically configures Linux iSCSI/multipath settings based on [config.json](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/helm/charts/hpe-csi-driver/files/config.json). In order to tune these values, edit the config map with `kubectl edit configmap hpe-linux-config -n kube-system` and restart node plugin using `kubectl delete pod -l app=hpe-csi-node` to apply.

!!! important
    HPE provide a set of general purpose default values for the IO paths, tuning is only required if prescribed by HPE.
