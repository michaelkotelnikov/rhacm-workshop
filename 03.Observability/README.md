# Exercise 3 - Observability

In this exercise you enable and use the `Observability` function in Red Hat Advanced Cluster Management. You will configure observability, explore the build in dashboards, enable custom alerts using Thanos Ruler and design custom dashboards for your own organizational needs.

### 3.1 - Deploying Observability

This part focuses on the `Observability` addon deployment. In order to deploy the functionality, you have to obtain an object storage provider. For this deployment you will use [minio](https://min.io/). A PVC will be assigned to a minio pod which will create a bucket and export an S3 object storage endpoint. The endpoint's information and credentials are exported in a secret with the `thanos-object-storage` name.

To create a namespace for the `MCO operator` to run in, run the next command on the hub cluster -

```
<hub> $ oc new-project open-cluster-management-observability
```

To create the `minio` deployment run the next commands on the hub cluster -

```
<hub> $ git clone https://github.com/open-cluster-management/multicluster-observability-operator.git

<hub> $ oc apply -k multicluster-observability-operator/examples/minio/ -n open-cluster-management-observability
```

After running the command, a `minio` deployment will be available. The S3 endpoint is now exported in the `thanos-object-storage` secret.

```
<hub> $ oc extract secret/thanos-object-storage --to=- -n open-cluster-management-observability
# thanos.yaml
type: s3
config:
  bucket: "thanos"
  endpoint: "minio:9000"
  insecure: true
  access_key: "minio"
  secret_key: "minio123"
```

To create an instance of `Multi Cluster Obervability`, apply the next object to the  `open-cluster-management-observability` namespace on the hub cluster.

**NOTE** If you're not using an OpenShift cluster that's deployed on AWS, make sure to modify the StorageClass definition in the below YAML.

```
<hub> $ oc apply -f https://raw.githubusercontent.com/michaelkotelnikov/rhacm-workshop/single-cluster/03.Observability/exercise/multiclusterobservability.yaml -n open-cluster-management-observability
```

Make sure that both `multicluster-observability-operator` and `endpoint-operator` are deployed (all pods must be in `Running` state).

```
<hub> $ oc get pods -n open-cluster-management-observability
NAME                                                       READY   STATUS    RESTARTS   AGE
grafana-dev-5f9585d797-qnpms                               2/2     Running   0          27h
minio-79c7ff488d-wbzrm                                     1/1     Running   0          2d1h
observability-alertmanager-0                               3/3     Running   0          2d1h
observability-alertmanager-1                               3/3     Running   0          2d1h
observability-alertmanager-2                               3/3     Running   0          2d1h
observability-grafana-6556b6d979-n72hv                     2/2     Running   0          2d1h
observability-grafana-6556b6d979-rjf27                     2/2     Running   0          2d1h
observability-observatorium-api-84fd8849b7-f9bkv           1/1     Running   0          2d1h
observability-observatorium-api-84fd8849b7-z5b9r           1/1     Running   0          2d1h
observability-observatorium-operator-74975fc6fb-gl4pr      1/1     Running   0          2d1h
observability-rbac-query-proxy-66c4944d4d-9ptzp            2/2     Running   0          2d1h
observability-rbac-query-proxy-66c4944d4d-rkl9g            2/2     Running   0          2d1h
observability-thanos-compact-0                             1/1     Running   0          2d1h
observability-thanos-query-5bf8459f67-bgpzw                1/1     Running   0          2d1h
observability-thanos-query-5bf8459f67-rzwss                1/1     Running   0          2d1h
observability-thanos-query-frontend-d6bd84889-9n9kw        1/1     Running   0          2d1h
observability-thanos-query-frontend-d6bd84889-bvn9f        1/1     Running   0          2d1h
observability-thanos-query-frontend-memcached-0            2/2     Running   0          2d1h
observability-thanos-query-frontend-memcached-1            2/2     Running   0          2d1h
observability-thanos-query-frontend-memcached-2            2/2     Running   0          2d1h
observability-thanos-receive-controller-7c775bcdff-7b5gb   1/1     Running   0          2d1h
observability-thanos-receive-default-0                     1/1     Running   0          2d1h
observability-thanos-receive-default-1                     1/1     Running   0          2d1h
observability-thanos-receive-default-2                     1/1     Running   0          2d1h
observability-thanos-rule-0                                2/2     Running   0          27h
observability-thanos-rule-1                                2/2     Running   0          27h
observability-thanos-rule-2                                2/2     Running   0          27h
observability-thanos-store-memcached-0                     2/2     Running   0          2d1h
observability-thanos-store-memcached-1                     2/2     Running   0          2d1h
observability-thanos-store-memcached-2                     2/2     Running   0          2d1h
observability-thanos-store-shard-0-0                       1/1     Running   2          2d1h
observability-thanos-store-shard-1-0                       1/1     Running   2          2d1h
observability-thanos-store-shard-2-0                       1/1     Running   2          2d1h

<hub> $ oc get pods -n open-cluster-management-addon-observability
NAME                                              READY   STATUS    RESTARTS   AGE
endpoint-observability-operator-764b6c666-9s7nz   1/1     Running   0          2d1h
metrics-collector-deployment-765946868-hmk5d      1/1     Running   0          2d1h
```

Now, that all pods are running, in RHACM's dashboard, navigate to **Cluster Lifecycle** -> **Grafana (top right side)**. Make sure that the dashboards are available and graphs are present.

### 3.2 - Exploe the default Grafana dashboards

This part focuses on the default Grafana dashboards that come with RHACM. Each dashboard has its own characteristics and provides valueable information to the system administrator in the organization. This section contains multiple tasks that require you to look for certain values in the default dashboards that come with `MCO`.

- Find the maximum latency value for the `local-cluster` API server.
- Find out how much % of `local-cluster`'s memory is utilized.
- Find what is the size of the etcd database in `local-cluster`.
- Find the namespace that consumes the most CPU in `local-cluster`.
- Find what's the node in `local-cluster` that consumes the most % memory.
- Find what's the `apiserver` (openshift-apiserver namespace) pod CPU utulization and quota.