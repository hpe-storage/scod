# Logging and diagnostics

Log files associated with the HPE CSI Driver logs data to the standard output stream. If the logs need to be retained for long term, use a standard logging solution. Some of the logs on the host are persisted which follow standard logrotate policies.

## CSI driver logs

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

## Log level

Log levels for both CSI Controller and Node driver can be controlled using `LOG_LEVEL` environment variable. Possible values are `info`, `warn`, `error`, `debug`, and `trace`. Apply the changes using `kubectl apply -f <yaml>` command after adding this to CSI controller and node container spec as below. For Helm charts this is controlled through `logLevel` variable in `values.yaml`.

```markdown
          env:
            - name: LOG_LEVEL
              value: trace
```

## Container Service Provider logs

CSP logs can be accessed as:

```markdown fct_label="HPE Nimble Storage"
kubectl logs -f svc/nimble-csp-svc -n kube-system
```

```markdown fct_label="HPE 3PAR and Primera"
kubectl logs -f svc/primera3par-csp-svc -n kube-system
```

## Log collector

Log collector script `hpe-logcollector.sh` can be used to collect the logs from any node which has kubectl access to the cluster.

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
