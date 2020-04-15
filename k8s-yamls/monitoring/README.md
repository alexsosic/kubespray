# Monitoring tools for k8s

## Prometheus, Alertmanager and Grafana

We use Prometheus to get:

- Proactive monitoring
- Cluster visibility and capacity planning
- Trigger alerts and notification
- Metrics dashboards

We are installing `Prometheus`, `Alertmanager` and `Grafana` using the coreos `Prometheus operator`.

We'll install the monitoring infrastructure in the `monitoring` namespace:

```
$ kubectl create namespace monitoring
namespace/monitoring create
```

We can now install the operator:

```
$ helm install stable/prometheus-operator --name prometheus-operator --namespace monitoring

NAME:   prometheus-operator
E0303 13:44:42.695445   31401 portforward.go:372] error copying from remote stream to local connection: readfrom tcp4 127.0.0.1:60917->127.0.0.1:60923: write tcp4 127.0.0.1:60917->127.0.0.1:60923: write: broken pipe
LAST DEPLOYED: Tue Mar  3 13:42:50 2020
NAMESPACE: monitoring
STATUS: DEPLOYED

RESOURCES:
==> v1/Alertmanager
NAME                              AGE
prometheus-operator-alertmanager  45s

==> v1/ClusterRole
NAME                                              AGE
prometheus-operator-grafana-clusterrole           45s
prometheus-operator-operator                      45s
prometheus-operator-operator-psp                  45s
prometheus-operator-prometheus                    45s
prometheus-operator-prometheus-psp                45s
psp-prometheus-operator-kube-state-metrics        45s
psp-prometheus-operator-prometheus-node-exporter  45s

==> v1/ClusterRoleBinding
NAME                                              AGE
prometheus-operator-grafana-clusterrolebinding    45s
prometheus-operator-operator                      45s
prometheus-operator-operator-psp                  45s
prometheus-operator-prometheus                    45s
prometheus-operator-prometheus-psp                45s
psp-prometheus-operator-kube-state-metrics        45s
psp-prometheus-operator-prometheus-node-exporter  45s

==> v1/ConfigMap
NAME                                                   AGE
prometheus-operator-apiserver                          46s
prometheus-operator-cluster-total                      46s
prometheus-operator-controller-manager                 46s
prometheus-operator-etcd                               46s
prometheus-operator-grafana                            46s
prometheus-operator-grafana-config-dashboards          46s
prometheus-operator-grafana-datasource                 46s
prometheus-operator-grafana-test                       46s
prometheus-operator-k8s-coredns                        46s
prometheus-operator-k8s-resources-cluster              46s
prometheus-operator-k8s-resources-namespace            46s
prometheus-operator-k8s-resources-node                 46s
prometheus-operator-k8s-resources-pod                  46s
prometheus-operator-k8s-resources-workload             46s
prometheus-operator-k8s-resources-workloads-namespace  46s
prometheus-operator-kubelet                            46s
prometheus-operator-namespace-by-pod                   46s
prometheus-operator-namespace-by-workload              46s
prometheus-operator-node-cluster-rsrc-use              46s
prometheus-operator-node-rsrc-use                      46s
prometheus-operator-nodes                              46s
prometheus-operator-persistentvolumesusage             46s
prometheus-operator-pod-total                          46s
prometheus-operator-pods                               46s
prometheus-operator-prometheus                         46s
prometheus-operator-proxy                              46s
prometheus-operator-scheduler                          46s
prometheus-operator-statefulset                        46s
prometheus-operator-workload-total                     46s

==> v1/DaemonSet
NAME                                          AGE
prometheus-operator-prometheus-node-exporter  45s

==> v1/Deployment
NAME                                    AGE
prometheus-operator-grafana             45s
prometheus-operator-kube-state-metrics  45s
prometheus-operator-operator            45s

==> v1/Pod(related)
NAME                                                     AGE
prometheus-operator-grafana-56b5c55587-nrp4s             45s
prometheus-operator-kube-state-metrics-8467c5bdbc-jmd6p  45s
prometheus-operator-operator-7997cc54f4-nwts8            45s
prometheus-operator-prometheus-node-exporter-6bsr7       45s
prometheus-operator-prometheus-node-exporter-6mpqd       45s
prometheus-operator-prometheus-node-exporter-7bw4b       45s
prometheus-operator-prometheus-node-exporter-f478g       45s
prometheus-operator-prometheus-node-exporter-lmt88       45s

==> v1/Prometheus
NAME                            AGE
prometheus-operator-prometheus  45s

==> v1/PrometheusRule
NAME                                                      AGE
prometheus-operator-alertmanager.rules                    43s
prometheus-operator-etcd                                  43s
prometheus-operator-general.rules                         44s
prometheus-operator-k8s.rules                             43s
prometheus-operator-kube-apiserver-error                  43s
prometheus-operator-kube-apiserver.rules                  43s
prometheus-operator-kube-prometheus-node-recording.rules  44s
prometheus-operator-kube-scheduler.rules                  43s
prometheus-operator-kubernetes-absent                     43s
prometheus-operator-kubernetes-apps                       43s
prometheus-operator-kubernetes-resources                  43s
prometheus-operator-kubernetes-storage                    44s
prometheus-operator-kubernetes-system                     43s
prometheus-operator-kubernetes-system-apiserver           43s
prometheus-operator-kubernetes-system-controller-manager  44s
prometheus-operator-kubernetes-system-kubelet             43s
prometheus-operator-kubernetes-system-scheduler           44s
prometheus-operator-node-exporter                         43s
prometheus-operator-node-exporter.rules                   43s
prometheus-operator-node-network                          43s
prometheus-operator-node-time                             43s
prometheus-operator-node.rules                            43s
prometheus-operator-prometheus                            44s
prometheus-operator-prometheus-operator                   43s

==> v1/Role
NAME                              AGE
prometheus-operator-alertmanager  45s
prometheus-operator-grafana-test  45s

==> v1/RoleBinding
NAME                              AGE
prometheus-operator-alertmanager  45s
prometheus-operator-grafana-test  45s

==> v1/Secret
NAME                                           AGE
alertmanager-prometheus-operator-alertmanager  46s
prometheus-operator-grafana                    46s

==> v1/Service
NAME                                          AGE
prometheus-operator-alertmanager              45s
prometheus-operator-coredns                   45s
prometheus-operator-grafana                   45s
prometheus-operator-kube-controller-manager   45s
prometheus-operator-kube-etcd                 45s
prometheus-operator-kube-proxy                45s
prometheus-operator-kube-scheduler            45s
prometheus-operator-kube-state-metrics        45s
prometheus-operator-operator                  45s
prometheus-operator-prometheus                45s
prometheus-operator-prometheus-node-exporter  45s

==> v1/ServiceAccount
NAME                                          AGE
prometheus-operator-alertmanager              46s
prometheus-operator-grafana                   46s
prometheus-operator-grafana-test              46s
prometheus-operator-kube-state-metrics        46s
prometheus-operator-operator                  46s
prometheus-operator-prometheus                46s
prometheus-operator-prometheus-node-exporter  46s

==> v1/ServiceMonitor
NAME                                         AGE
prometheus-operator-alertmanager             43s
prometheus-operator-apiserver                43s
prometheus-operator-coredns                  43s
prometheus-operator-grafana                  43s
prometheus-operator-kube-controller-manager  43s
prometheus-operator-kube-etcd                43s
prometheus-operator-kube-proxy               43s
prometheus-operator-kube-scheduler           43s
prometheus-operator-kube-state-metrics       43s
prometheus-operator-kubelet                  43s
prometheus-operator-node-exporter            43s
prometheus-operator-operator                 43s
prometheus-operator-prometheus               43s

==> v1beta1/ClusterRole
NAME                                    AGE
prometheus-operator-kube-state-metrics  45s

==> v1beta1/ClusterRoleBinding
NAME                                    AGE
prometheus-operator-kube-state-metrics  45s

==> v1beta1/MutatingWebhookConfiguration
NAME                           AGE
prometheus-operator-admission  45s

==> v1beta1/PodSecurityPolicy
NAME                                          AGE
prometheus-operator-alertmanager              46s
prometheus-operator-grafana                   46s
prometheus-operator-grafana-test              46s
prometheus-operator-kube-state-metrics        46s
prometheus-operator-operator                  46s
prometheus-operator-prometheus                46s
prometheus-operator-prometheus-node-exporter  46s

==> v1beta1/Role
NAME                         AGE
prometheus-operator-grafana  45s

==> v1beta1/RoleBinding
NAME                         AGE
prometheus-operator-grafana  45s

==> v1beta1/ValidatingWebhookConfiguration
NAME                           AGE
prometheus-operator-admission  43s


NOTES:
The Prometheus Operator has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prometheus-operator"

Visit https://github.com/coreos/prometheus-operator for instructions on how
to create & configure Alertmanager and Prometheus instances using the Operator.
```

Grafana, AlertManager and Prometheus need *Persistent Volumes* to permanently store their data. The persistent volumes are claimed at pod startup using *Persistent Volume Claims*.

For development purposes we are using *Hostpath*-based Persistent Volumes created manually on the k8s master and worker nodes:

```
$ kubectl apply -f monitoring/kube-prometheus-pv.yaml
persistentvolume/kube-prometheus-alertmanager configured
persistentvolume/kube-prometheus-prometheus configured
persistentvolume/kube-prometheus-grafana configured
```

Let's check if the persistent volumes have been created correctly:

```
$ kubectl get pv --show-labels
NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                                                       STORAGECLASS                   REASON    AGE       LABELS
kube-prometheus-alertmanager   10Gi       RWO            Recycle          Bound     monitoring/alertmanager-kube-prometheus-db-alertmanager-kube-prometheus-0   kube-prometheus-alertmanager             14m       component=master,release=kube-prometheus
kube-prometheus-grafana        10Gi       RWO            Recycle          Bound     monitoring/kube-prometheus-grafana                                          kube-prometheus-grafana                  14m       component=master,release=kube-prometheus
kube-prometheus-prometheus     10Gi       RWO            Recycle          Bound     monitoring/prometheus-kube-prometheus-db-prometheus-kube-prometheus-0       kube-prometheus-prometheus               14m       component=master,release=kube-prometheus
```

We are ready to install the `prometheus` custom resource through the operator:

```
$ helm install stable/prometheus --name prometheus -f monitoring/prometheus-values.yaml --namespace monitoring
NAME:   kube-prometheus
LAST DEPLOYED: Fri Sep 21 12:46:17 2018
NAMESPACE: monitoring
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/PodSecurityPolicy
NAME                                 DATA   CAPS      SELINUX   RUNASUSER  FSGROUP    SUPGROUP  READONLYROOTFS  VOLUMES
kube-prometheus-alertmanager         false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
kube-prometheus-exporter-kube-state  false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
kube-prometheus-exporter-node        false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim,hostPath
kube-prometheus-grafana              false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim,hostPath
kube-prometheus                      false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim

==> v1/ServiceAccount
NAME                                 SECRETS  AGE
kube-prometheus-exporter-kube-state  1        4s
kube-prometheus-exporter-node        1        4s
kube-prometheus-grafana              1        4s
kube-prometheus                      1        4s

==> v1beta1/ClusterRole
NAME                                     AGE
psp-kube-prometheus-alertmanager         4s
kube-prometheus-exporter-kube-state      4s
psp-kube-prometheus-exporter-kube-state  4s
psp-kube-prometheus-exporter-node        4s
psp-kube-prometheus-grafana              4s
kube-prometheus                          4s
psp-kube-prometheus                      4s

==> v1beta1/ClusterRoleBinding
NAME                                     AGE
psp-kube-prometheus-alertmanager         4s
kube-prometheus-exporter-kube-state      4s
psp-kube-prometheus-exporter-kube-state  4s
psp-kube-prometheus-exporter-node        4s
psp-kube-prometheus-grafana              4s
kube-prometheus                          4s
psp-kube-prometheus                      4s

==> v1beta1/RoleBinding
NAME                                 AGE
kube-prometheus-exporter-kube-state  4s

==> v1beta1/Deployment
NAME                                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
kube-prometheus-exporter-kube-state  1        1        1           0          3s
kube-prometheus-grafana              1        1        1           0          3s

==> v1/Secret
NAME                          TYPE    DATA  AGE
alertmanager-kube-prometheus  Opaque  1     4s
kube-prometheus-grafana       Opaque  2     4s

==> v1/ConfigMap
NAME                     DATA  AGE
kube-prometheus-grafana  10    4s

==> v1/Service
NAME                                              TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)              AGE
kube-prometheus-alertmanager                      ClusterIP  10.104.8.207    <none>       9093/TCP             4s
kube-prometheus-exporter-kube-controller-manager  ClusterIP  None            <none>       10252/TCP            4s
kube-prometheus-exporter-kube-dns                 ClusterIP  None            <none>       10054/TCP,10055/TCP  4s
kube-prometheus-exporter-kube-etcd                ClusterIP  None            <none>       4001/TCP             4s
kube-prometheus-exporter-kube-scheduler           ClusterIP  None            <none>       10251/TCP            4s
kube-prometheus-exporter-kube-state               ClusterIP  10.100.255.137  <none>       80/TCP               3s
kube-prometheus-exporter-node                     ClusterIP  10.107.255.133  <none>       9100/TCP             3s
kube-prometheus-grafana                           ClusterIP  10.102.162.78   <none>       80/TCP               3s
kube-prometheus                                   ClusterIP  10.109.179.227  <none>       9090/TCP             3s

==> v1beta1/DaemonSet
NAME                           DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
kube-prometheus-exporter-node  6        6        0      6           0          <none>         3s

==> v1/Alertmanager
NAME             AGE
kube-prometheus  3s

==> v1/PrometheusRule
kube-prometheus-alertmanager                      3s
kube-prometheus-exporter-kube-controller-manager  3s
kube-prometheus-exporter-kube-etcd                3s
kube-prometheus-exporter-kube-scheduler           3s
kube-prometheus-exporter-kube-state               3s
kube-prometheus-exporter-kubelets                 3s
kube-prometheus-exporter-kubernetes               3s
kube-prometheus-exporter-node                     3s
kube-prometheus-rules                             3s
kube-prometheus                                   3s

==> v1/ServiceMonitor
kube-prometheus-alertmanager                      3s
kube-prometheus-exporter-kube-controller-manager  3s
kube-prometheus-exporter-kube-dns                 3s
kube-prometheus-exporter-kube-etcd                3s
kube-prometheus-exporter-kube-scheduler           3s
kube-prometheus-exporter-kube-state               3s
kube-prometheus-exporter-kubelets                 3s
kube-prometheus-exporter-kubernetes               3s
kube-prometheus-exporter-node                     3s
kube-prometheus-grafana                           3s
kube-prometheus                                   3s

==> v1beta1/Role
kube-prometheus-exporter-kube-state  4s

==> v1/Prometheus
kube-prometheus  3s

==> v1/Pod(related)
NAME                                                  READY  STATUS             RESTARTS  AGE
kube-prometheus-exporter-node-4b78h                   0/1    ContainerCreating  0         3s
kube-prometheus-exporter-node-cnnxr                   0/1    ContainerCreating  0         3s
kube-prometheus-exporter-node-dlj8s                   0/1    ContainerCreating  0         3s
kube-prometheus-exporter-node-gs7n6                   0/1    ContainerCreating  0         3s
kube-prometheus-exporter-node-ktmvf                   0/1    ContainerCreating  0         3s
kube-prometheus-exporter-node-m4j2d                   0/1    ContainerCreating  0         3s
kube-prometheus-exporter-kube-state-658f46b8dd-2xll6  0/2    ContainerCreating  0         3s
kube-prometheus-grafana-f869c754-nm4x9                0/2    ContainerCreating  0         3s


NOTES:
DEPRECATION NOTICE:

- alertmanager.ingress.fqdn is not used anymore, use alertmanager.ingress.hosts []
- prometheus.ingress.fqdn is not used anymore, use prometheus.ingress.hosts []
- grafana.ingress.fqdn is not used anymore, use prometheus.grafana.hosts []

- additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- prometheus.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- alertmanager.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- exporter-kube-controller-manager.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- exporter-kube-etcd.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- exporter-kube-scheduler.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- exporter-kubelets.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
- exporter-kubernetes.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
```

The pods are up and running:

```
$ kubectl get pods -n monitoring
NAME                                                   READY     STATUS    RESTARTS   AGE
alertmanager-kube-prometheus-0                         2/2       Running   0          58s
kube-prometheus-exporter-kube-state-54959bf8d-w584s    1/2       Running   0          30s
kube-prometheus-exporter-kube-state-658f46b8dd-2xll6   2/2       Running   0          58s
kube-prometheus-exporter-node-4b78h                    1/1       Running   0          58s
kube-prometheus-exporter-node-cnnxr                    1/1       Running   0          58s
kube-prometheus-exporter-node-dlj8s                    1/1       Running   0          58s
kube-prometheus-exporter-node-gs7n6                    1/1       Running   0          58s
kube-prometheus-exporter-node-ktmvf                    1/1       Running   0          58s
kube-prometheus-exporter-node-m4j2d                    1/1       Running   0          58s
kube-prometheus-grafana-f869c754-nm4x9                 2/2       Running   0          58s
prometheus-kube-prometheus-0                           3/3       Running   1          58s
prometheus-operator-858c485-7z9dm                      1/1       Running   0          4m
```

Let's create the services to use grafana and alertmanager from a node port:

```
$ kubectl apply -f monitoring/prometheus-services.yaml -n monitoring
service/kube-prometheus-grafana-nodeport configured
```

You can now access the dashboard from the [port 32004 of any node of the cluster](http://159.69.177.125:32004).
