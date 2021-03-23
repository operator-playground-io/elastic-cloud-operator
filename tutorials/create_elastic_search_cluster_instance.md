---
title: Elasticsearch Cluster Instance Creation
description: How to create an instance of Elasticsearch cluster?
---

### Create Instance of Elasticsearch Cluster

**Step 1:** Create Elasticsearch Cluster Instance

- Before creating an instance, create a persistent volume. To do so, create a file named 'elastic_pv.yaml' in. Edit this file, and enter the below spec in.

```execute
cat <<'EOF' >>elastic_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elastic-pv
  labels:
    type: local
spec:  
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
EOF
```

- Now, execute the command below to create a PersistentVolume (PV).

```execute
kubectl create -f elastic_pv.yaml -n operators
```

- Run the following command to create a yaml file.

```execute
cat <<'EOF' >>elasticsearch_cluster.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1beta1
kind: Elasticsearch
metadata:
  name: elasticsearch
spec:
  version: 7.4.2
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```

The spec induces the operator to deploy a single node Elasticsearch cluster named `ElasticSearch`. This cluster will consequently have all its communications secured by virtue of Transport Layer Security (TLS).

- Create the Elasticsearch Cluster.

```execute
kubectl create -f elasticsearch_cluster.yaml -n operators
```

The operator automatically creates and manages Kubernetes resources to achieve the desired state of the Elasticsearch cluster. 

-  Execute command below to get the status of pods.

```execute
kubectl get pods -n operators | egrep -i "name|elasticsearch"
```

Please wait for the resources to be created and the cluster to be ready for use. You should see the STATUS as `Running` and `READY` should be `1/1`.

-  Execute below command to check the health of current ElasticSearch Cluster

```execute
kubectl get elasticsearch -n operators
```

Once a cluster is created, there is no `HEALTH` status displayed and the PHASE is empty. After some time, the `PHASE` state changes to Ready, and the `HEALTH` status becomes green. The `HEALTH` status turns green only when all the pods of that cluster are in `READY` state.

- You can check that the pod is in the process of being started, as below. 

```execute
kubectl get pods -n operators --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch'
```

**Note - Please wait until the STATUS is `Running` and `READY` value is `1/1`, then proceed.**

- Access the pod logs by running the following command. 
 
```execute
kubectl logs -f elasticsearch-es-default-0 -n operators | more
```


* Get the user credentials using the syntax below

A default username `elastic` is automatically along with a password that is stored in a Kubernetes secret.

```execute
PASSWORD=$(kubectl get secret elasticsearch-es-elastic-user -n operators -o=jsonpath='{.data.elastic}' | base64 --decode)
echo $PASSWORD
```
This should result in something like:

```
fdqxb7j68pfvj....
```

- Request access to Elasticsearch Endpoint from within the Kubernetes cluster.

From inside the Kubernetes cluster:

```execute
ClusterIP=`kubectl get service -n operators | grep -i  elasticsearch | awk 'NR==2 {print $3}'`
echo $ClusterIP
curl -u "elastic:$PASSWORD" -k "https://$ClusterIP:9200"
```

You should see something like:

```
{
  "name" : "elasticsearch-es-default-0",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "9cJ5aKa_R92jXw1A8tWznA",
  "version" : {
    "number" : "7.4.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
    "build_date" : "2019-10-28T20:40:44.881551Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```

You have successfully setup the Elasticsearch Cluster Instance.