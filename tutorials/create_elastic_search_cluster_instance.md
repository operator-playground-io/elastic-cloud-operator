---
title: Tutorial to create an instance of Elastic Search cluster
description: This tutorial explains how to create an instance of Elastic Search cluster
---

### Create Instance of Elastic Search Cluster

#Execute below command to create yaml file

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

This small specification causes the operator to deploy a single node Elasticsearch cluster named ElasticSearch . The cluster will automatically have all its communications secured using Transport Layer Security (TLS)

#Execute below command to create cluster

```execute
kubectl create -f elasticsearch_cluster.yaml -n operators
```

```
elasticsearch.elasticsearch.k8s.elastic.co/elasticsearch created
```

The operator automatically creates and manages Kubernetes resources to achieve the desired state of the Elasticsearch cluster. 

#Execute below command to get status of pods

```execute
kubectl get pods -n operators | egrep -i "name|elasticsearch"
```

Sample output:

```
NAME                         READY   STATUS     RESTARTS   AGE
elasticsearch-es-default-0   0/1     Init:0/1   0          27s
```

It may take up to a few minutes until all the resources are created and the cluster is ready for use. Status should be `running` and READY should be `1/1` .

```
NAME                         READY   STATUS    RESTARTS   AGE
elasticsearch-es-default-0   1/1     Running   0          3m23s
```

#Execute below command to check the helath of current ElasticSearch Cluster

```execute
kubectl get elasticsearch -n operators
```

```
NAME            HEALTH   NODES   VERSION   PHASE   AGE
elasticsearch   green    1       7.4.2     Ready   45s
```

When you create the cluster, there is no HEALTH status and the PHASE is empty. After a while, the PHASE turns into Ready, and HEALTH becomes green. Health will turn into green only when all pods of that cluster are in `READY` state.

You can see that one Pod is in the process of being started:

```execute
kubectl get pods -n operators --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch'
```

You will find output similar below:

```
NAME                         READY   STATUS    RESTARTS   AGE
elasticsearch-es-default-0   1/1     Running   0          65s
```

**Note - Please wait till `Status` should be `Running` and `READY` should be 1/1 , and then proceed further.**

#You can run below command to access logs of pods

```
kubectl logs -f elasticsearch-es-default-0 -n operators | more
```

You will find output simlar below :

```
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
{"type": "server", "timestamp": "2019-11-27T09:34:44,928Z", "level": "INFO", "component": "o.e.e.NodeEnvironment", "cluster.name": "elasticsearch", "node.name": "elasticsearch-es-default-0", "message": "using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/xvda2)]], net usable_space [76.5gb], net total_space [98gb], types [ext3]" }
{"type": "server", "timestamp": "2019-11-27T09:34:44,935Z", "level": "INFO", "component": "o.e.e.NodeEnvironment", "cluster.name": "elasticsearch", "node.name": "elasticsearch-es-default-0", "message": "heap size [1015.6mb], compressed ordinary object pointers [true]" }
{"type": "server", "timestamp": "2019-11-27T09:34:44,939Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "elasticsearch", "node.name": "elasticsearch-es-default-0", "message": "node name [elasticsearch-es-default-0], node ID [tavAWXP7Qe63rhMJHfYaZQ], cluster name [elasticsearch]" }
```

#Get the credentials

A default user named elastic is automatically created with the password stored in a Kubernetes secret:

```execute
PASSWORD=$(kubectl get secret elasticsearch-es-elastic-user -n operators -o=jsonpath='{.data.elastic}' | base64 --decode)
echo $PASSWORD
```
You will find output similar below:

```
fdqxb7j68pfvj....
```

#Request the Elasticsearch endpoint

From inside the Kubernetes cluster:

```execute
ClusterIP=`kubectl get service -n operators | grep -i  elasticsearch | awk 'NR==2 {print $3}'`
echo $ClusterIP
curl -u "elastic:$PASSWORD" -k "https://$ClusterIP:9200"
```

You will find output similar below:

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
