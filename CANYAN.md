# Documentation

## Introduction

This is a fork of the [Kubespray project](https://github.com/kubernetes-sigs/kubespray) used to deploy Kubernetes clusters on different cloud providers or on-premises.

In addition this repository contains documentation and `yaml` files for the deployment of:
* docker registry
* hostpath provisioners
* nginx ingress
* helm chartmuseum
* Logging (Elastic, Cerebro, Kibana and Fluentbit)
* Monitoring (Prometheus, Alertmanager and Grafana)


## Cluster Installation

### Prerequisites

One should be familiar with [Kubespray](https://github.com/kubernetes-sigs/kubespray). We should start with creating a Python virtual environment:

```$ virtualenv -p python3 venv --no-site-packages```

After creating the venv we shoud activate it with:

```$ source venv/bin/activate```

and then run the setup to install the dependencies from the `requirements.txt` file:

```$ pip install -r requirements.txt```


The inventory hosts are defined in `inventory/ibm/hosts.yaml`.


### Running the playbooks

Once Kubespray and it's requirements are installed and the hosts file reviewed we can start deploying the cluster.

The first `yml` file to be deployed is the bootstrap one. It does the update of the packages and installs `python-minimal` and `python-setuptools` required for the next steps. The command to run the ansible playbook for the ibm cluster is:

```$ ansible-playbook -i inventory/ibm/hosts.yaml  --become --become-user=root bootstrap.yml --flush-cache -v```

To run it on another cluster just change the `-i` parameter from `inventory/ibm/hosts.yaml` to your own hosts file.

The second step is to deploy the cluster and it's run with:

```$ ansible-playbook -i inventory/ibm/hosts.yaml  --become --become-user=root cluster.yml --flush-cache -v```

This command will take some time to complete. After the success of the procedure we have a running Kubernetes cluster on the hosts defined.


## Kubectl configuration

Once the cluster is up and running we can get the kubernetes conf file used by `kubectl` on our local machine to access the cluster:

```$ ansible-playbook -i inventory/ibm/hosts.yaml  --become --become-user=root kubeconf.yml --flush-cache -v```

The file is available at `secrets/kubeconfig/admin.conf`

Edit your `~/.kube/conf` to include the cluster, context and users from the `admin.conf` retrieved with ansible.
For detailed information on `.kube/conf` files take a look at the [official documentation](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).

Once the configuration is in place we can test it requesting the list of nodes of our cluster like this:

```
$ kubectl get nodes
NAME    STATUS   ROLES    AGE   VERSION
node1   Ready    master   8d    v1.16.6
node2   Ready    master   8d    v1.16.6
node3   Ready    <none>   8d    v1.16.6
node4   Ready    <none>   8d    v1.16.6
node5   Ready    <none>   8d    v1.16.6
```

A useful tool for multi-cluster switching is [kubectx](https://github.com/ahmetb/kubectx) and there is also `kubens` to switch between namespaces.


## GlusterFS installation

[GlusterFS](https://www.gluster.org/) is a free and open source software scalable network filesystem.

In the `hosts.yaml` file there is a section `glusterfs` defining the nodes GlusterFS will use.
Also in the `var` section there is a `glusterfs_brick_dir` variable defining the bricks dir to be used on the nodes.

In order to install it on our cluster we run the following command:

```$ ansible-playbook -i inventory/ibm/hosts.yaml  --become --become-user=root glusterfs.yml --flush-cache -v```


## OpenEBS

[OpenEBS](https://openebs.io/) is the leading storage solution for Kubernetes.
Let's install it with helm:

```
$ helm install --namespace openebs --name openebs stable/openebs

NAME:   openebs
LAST DEPLOYED: Thu Mar 12 14:01:29 2020
NAMESPACE: openebs
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME     AGE
openebs  0s

==> v1/ClusterRoleBinding
NAME     AGE
openebs  0s

==> v1/ConfigMap
NAME                AGE
openebs-ndm-config  0s

==> v1/DaemonSet
NAME         AGE
openebs-ndm  0s

==> v1/Deployment
NAME                         AGE
openebs-admission-server     0s
openebs-apiserver            0s
openebs-localpv-provisioner  0s
openebs-ndm-operator         0s
openebs-provisioner          0s
openebs-snapshot-operator    0s

==> v1/Pod(related)
NAME                                         AGE
openebs-admission-server-7b4859ccd5-xf9v6    0s
openebs-apiserver-757d58577c-dkbdp           0s
openebs-apiserver-757d58577c-kcfx9           35s
openebs-localpv-provisioner-c6bc845bb-h5bnr  35s
openebs-localpv-provisioner-c6bc845bb-qbnvd  0s
openebs-ndm-5b6k5                            0s
openebs-ndm-fs7q6                            35s
openebs-ndm-hz68p                            0s
openebs-ndm-operator-5f6c5497d7-84kk9        0s
openebs-ndm-skt5t                            0s
openebs-ndm-w9lzb                            0s
openebs-ndm-xtpzc                            0s
openebs-provisioner-598c45dd4-ll8xn          0s
openebs-provisioner-598c45dd4-wld9m          35s
openebs-snapshot-operator-5f74599c8c-d96zr   0s

==> v1/Service
NAME                AGE
openebs-apiservice  0s

==> v1/ServiceAccount
NAME     AGE
openebs  0s


NOTES:
The OpenEBS has been installed. Check its status by running:
$ kubectl get pods -n openebs

For dynamically creating OpenEBS Volumes, you can either create a new StorageClass or
use one of the default storage classes provided by OpenEBS.

Use `kubectl get sc` to see the list of installed OpenEBS StorageClasses. A sample
PVC spec using `openebs-jiva-default` StorageClass is given below:"

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-vol-claim
spec:
  storageClassName: openebs-jiva-default
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
---

Please note that, OpenEBS uses iSCSI for connecting applications with the
OpenEBS Volumes and your nodes should have the iSCSI initiator installed.

For more information, visit our Slack at https://openebs.io/community or view the documentation online at http://docs.openebs.io/.
```

If we take a look at the storage classes we will find `openbs-hostpath` which we will use for elasticsearch 
and other applications that are doing replication by themselves (for example MongoDB).

```
$ kubectl get storageclass
```

For all the other applications that are not replicating data by themselves we can use GlusterFS with OpenEBS with this yaml:

```
$ kubectl apply -f k8s-yamls/openebs-glusterfs.yaml
```

Again with ``` $ kubectl get storageclass ``` we will see also the `openebs-glusterfs` storage class.


## Docker Registry

If we need a local docker registry deployed on our cluster we can easily do it with the following command:

```$ kubectl apply -f k8s-yamls/registry/ibm/registry.yaml```

It will run a deployment, service and ingress in the `default` namespace.
There is no ssl in our case so there is a need to edit the `daemon.json` on the local machine that is pushing the images to the repository adding:
```
{
  "insecure-registries" : ["registry.k8.ibm.local:5000"]
}
```


## Helm

We use `helm` package manager to install the components. Please refer to the [Helm documentation](https://docs.helm.sh/using_helm/#installing-helm) for installing the tool on your specific platform.

Set up `RBAC` and install the `helm tiller` (once in the lifecycle of a K8S cluster):

```
$ kubectl apply -f k8s-yamls/helm/rbac-config.yaml
$ helm init --service-account tiller --history-max 10
```

Update the repos:

```
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "incubator" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```


## Nginx ingress controller

Install the nginx ingress controller using the helm chart `stable/nginx-ingress`:

```
$ helm install --name nginx-ingress stable/nginx-ingress -f k8s-yamls/helm/nginx-ingress.yaml
```


## ChartMuseum

[ChartMuseum](https://github.com/helm/chartmuseum) is an open-source Helm Chart Repository written in Go (Golang), with support for cloud storage backends. Works as a valid Helm Chart Repository, and also provides an API for uploading new chart packages to storage.
We use the official Helm chart for ChartMuseum to deploy `ChartMusem` to our k8s cluster.
We are using a `presistent volume` to store the charts as we do it for the registry. 

Now, Install the Helm chart:

```
$ helm install --name chartmuseum --namespace default -f k8s-yamls/helm/chartmuseum/chartmuseum-values.yaml stable/chartmuseum --version 2.7.1

NAME:   chartmuseum
LAST DEPLOYED: Thu Mar 12 15:47:47 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME                     AGE
chartmuseum-chartmuseum  0s

==> v1/PersistentVolumeClaim
NAME                     AGE
chartmuseum-chartmuseum  0s

==> v1/Pod(related)
NAME                                      AGE
chartmuseum-chartmuseum-5585fdb48f-t6927  0s

==> v1/Secret
NAME                     AGE
chartmuseum-chartmuseum  0s

==> v1/Service
NAME                     AGE
chartmuseum-chartmuseum  0s


NOTES:
** Please be patient while the chart is being deployed **

Get the ChartMuseum URL by running:

  export POD_NAME=$(kubectl get pods --namespace default -l "app=chartmuseum" -l "release=chartmuseum" -o jsonpath="{.items[0].metadata.name}")
  echo http://127.0.0.1:8080/
  kubectl port-forward $POD_NAME 8080:8080 --namespace default

```

The deployment is up and running:

```
$ kubectl get pods -n default
chartmuseum-chartmuseum-5585fdb48f-t6927         1/1     Running     0          40s
```

We are using the chart repository from a public IP with a DNS entry, thus we need to declare an ingress for `ChartMuseum`:

```
$ kubectl apply -f k8s-yamls/helm/chartmuseum/ibm/chartmuseum-ingress.yaml -n default
ingress.extensions/chartmuseum-ingress created
```

We can now use our chart server: [http://charts.k8.ibm.local/index.yaml](http://charts.k8.ibm.local/index.yaml)


## Elasticsearch cluster

Elasticsearch is the storage database for logs collected among the components of our infrastructure. For development purposes we deploy a single-master, single-data, single-client Elasticsearch cluster setting the minimum number of masters in the cluster to 1.

**PLEASE NOTE: this is a testing configuration not suitable for production environments!**

### Deployment of the Elasticsearch cluster

Let's create a namespace for the logging tools we will deploy:

```$ kubectl create namespace efk```

Let's add the helm repository named `elastic`:
```
$ helm repo add elastic https://helm.elastic.co
```

Now we can use the `helm` package manager to deploy the elasticsearch cluster into the *efk* namespace:

```
$ helm install elastic/elasticsearch --name elastic -f k8s-yamls/logging/elastic-values.yaml --namespace efk

NAME:   elastic
LAST DEPLOYED: Thu Mar 12 14:37:34 2020

NAMESPACE: efk
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                    AGE
elasticsearch-master-0  0s
elasticsearch-master-1  0s
elasticsearch-master-2  0s

==> v1/Service
NAME                           AGE
elasticsearch-master           0s
elasticsearch-master-headless  0s

==> v1/StatefulSet
NAME                  AGE
elasticsearch-master  0s

==> v1beta1/PodDisruptionBudget
NAME                      AGE
elasticsearch-master-pdb  0s


NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=efk -l app=elasticsearch-master -w
2. Test cluster health using Helm test.
  $ helm test elastic
```

Our *elastic* release is deployed, according to `helm`:

```
$ helm list
NAME         	REVISION	UPDATED                 	STATUS  	CHART               	APP VERSION	NAMESPACE
chartmuseum  	1       	Mon Mar 16 20:16:54 2020	DEPLOYED	chartmuseum-2.7.1   	0.11.0     	default
elastic      	1       	Mon Mar 16 20:17:43 2020	DEPLOYED	elasticsearch-7.6.1 	7.6.1      	efk
nginx-ingress	1       	Fri Feb 28 11:31:56 2020	DEPLOYED	nginx-ingress-1.33.0	0.30.0     	default
openebs      	1       	Mon Mar 16 19:50:37 2020	DEPLOYED	openebs-1.8.0       	1.8.0      	openebs
```

We can check elasticserach cluster pods are up and running:

```
$ kubectl get pods -n efk
NAME                                            READY     STATUS    RESTARTS   AGE
elastic-elasticsearch-client-674b6f8dd6-fj9tz   1/1       Running   0          3m
elastic-elasticsearch-data-0                    1/1       Running   0          3m
elastic-elasticsearch-master-0                  1/1       Running   0          3m
```

Let's try to use our cluster, first of all forwarding the 9200 TCP port to localhost:

```
$ kubectl port-forward --namespace efk elastic-elasticsearch-client-674b6f8dd6-s2qnv 9200:9200
Forwarding from 127.0.0.1:9200 -> 9200
Forwarding from [::1]:9200 -> 9200
```

From another shell we can call Elasticsearch:

```
$ curl http://localhost:9200
{
  "name" : "elastic-elasticsearch-client-674b6f8dd6-s2qnv",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "6.4.0",
    "build_flavor" : "oss",
    "build_type" : "tar",
    "build_hash" : "595516e",
    "build_date" : "2018-08-17T23:18:47.308994Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### Upgrade of the elasticsearch cluster

If we need to fine-tune the configuration of our cluster we can update the settings in ` k8s-yamls/logging/elastic-values.yaml` and upgrade the release:

```
$ helm upgrade elastic elastic/elasticsearch -f k8s-yamls/logging/elastic-values.yaml --namespace efk
```


### Tear down of the elasticsearch cluster

We ask `helm` to delete (and optionally purge) the release:

```
$ helm delete --purge elastic
```


## Cerebro

Cerebro is an elasticsearch web admin tool. We use `helm` to install it into our infrastructure:

```
$ helm install stable/cerebro --name cerebro -f k8s-yamls/logging/ibm/cerebro-values.yaml --namespace efk

NAME:   cerebro
LAST DEPLOYED: Wed Sep 12 10:37:05 2018
NAMESPACE: efk
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME     DATA  AGE
cerebro  1     0s

==> v1/Service
NAME     TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)  AGE
cerebro  ClusterIP  10.110.75.87  <none>       80/TCP   0s

==> v1/Deployment
NAME     DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
cerebro  1        1        1           0          0s

==> v1/Pod(related)
NAME                      READY  STATUS    RESTARTS  AGE
cerebro-64864d7d9f-cbp7s  0/1    Init:0/1  0         0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace efk -l "app=cerebro,release=cerebro" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

**NOTE: the output provided by helm is wrong, the TCP PORT cerebro is listening to is 9000!**

Let's try to use our cerebro instance, first of all forwarding the 80 TCP port to localhost:

```
$ kubectl port-forward --namespace efk cerebro-64864d7d9f-cbp7s 9000:9000
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

You can now open your browser and point it to `http://localhost:9000`.


## Kibana

Kibana is a data visualization, exploration and analysis tool for the Elastic stack. We use `helm` to install it into our infrastructure:


```
helm install --name kibana elastic/kibana -f k8s-yamls/logging/ibm/kibana-values.yaml --namespace efk

NAME:   kibana
LAST DEPLOYED: Thu Mar 12 16:18:02 2020
NAMESPACE: efk
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME           AGE
kibana-kibana  0s

==> v1/Pod(related)
NAME                            AGE
kibana-kibana-7d858b77f6-s8bql  0s

==> v1/Service
NAME           AGE
kibana-kibana  0s

==> v1beta1/Ingress
NAME           AGE
kibana-kibana  0s

```

Let's try to use our Kibana instance, first of all forwarding the 5601 TCP port to localhost:

```
$ kubectl port-forward --namespace efk kibana-5f8fbdd7c8-86l94 5601:5601
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

You can now open your browser and point it to `http://localhost:5601`.

We also expose cerebro as a nodePort service.
You can now connect to kibana [from your browser](http://k8sdev-node1.cloud.evox.it:32001).


## Collection of logs

In order to collect the logs from the components of our infrastructure we use the following technologies:

* **fluentd** - log forwarder installed on the k8s nodes

### fluentd

```
$ helm repo add kiwigrid https://kiwigrid.github.io
$ helm repo update
```

We can now use `helm` to install it into our infrastructure:

```
$ helm install kiwigrid/fluentd-elasticsearch -f k8s-yamls/logging/fluentd-values.yaml --name fluentd --namespace efk
```

`fluentd` is a log forwarder with native Kubernetes support.
`fluentd` will automatically start to send log data to `elasticsearch`.


## Kubernetes Dashboard

We create a k8s user `kube-admin` authorized to access the dashboard:

```
$ kubectl apply -f k8s-yamls/dashboard/dashboard-adminuser.yml
```
and:

```
$ kubectl apply -f k8s-yamls/dashboard/admin-role-binding.yml

serviceaccount/admin-user unchanged
clusterrolebinding.rbac.authorization.k8s.io/admin-user configured
```

We can now get the the bearer token to authenticate in the dashboard:

```
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

Name:         admin-user-token-c8kws
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
              kubernetes.io/service-account.uid=8aef614e-bd80-11e8-b69d-96000010ad8f

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGc.....
ca.crt:     1025 bytes
```

You can now access the dashboard from the [port 32003 of any node of the cluster](https://kubem01.ibm.local:32003).

Use the token you previously obtained to log in to the dashboard.


## Monitoring

We are installing `Prometheus`, `Alertmanager` and `Grafana` using the `Prometheus operator`.

We'll install the monitoring infrastructure in the `monitoring` namespace so let's create it:

```
$ kubectl create namespace monitoring
namespace/monitoring create
```

We use Prometheus to get:

- Proactive monitoring
- Cluster visibility and capacity planning
- Trigger alerts and notification
- Metrics dashboards

We can now install the operator:

```
$ helm install stable/prometheus-operator --name prometheus-operator -f k8s-yamls/monitoring/ibm/prometheus-values.yaml --namespace monitoring

```
