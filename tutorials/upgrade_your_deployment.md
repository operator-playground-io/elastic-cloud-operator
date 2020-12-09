---
title: Upgrade deployment
description: This tutorial explains how to upgrade deployment
---
### Upgrade deployment
You can add and modify most elements of the original cluster specification provided that they translate to valid transformations of the underlying Kubernetes resources (e.g., existing volume claims cannot be resized). The operator will attempt to apply your changes with minimal disruption to the existing cluster. You should ensure that the Kubernetes cluster has sufficient resources to accommodate the changes (extra storage space, sufficient memory and CPU resources to temporarily spin up new pods etc.).

```execute
cat << EOF > elasticsearch_cluster.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1beta1
kind: Elasticsearch
metadata:
  name: elasticsearch
spec:
  version: 7.4.2
  nodeSets:
  - name: default
    count: 5
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```

#Execute below command to apply changes:

```execute
kubectl apply -f elasticsearch_cluster.yaml -n operators
```

#Now you can check pods status by executing below command:

```execute
kubectl get pods -n operators
```

You will find output similar below:

```
NAME                                 READY   STATUS    RESTARTS   AGE
elasticsearch-es-default-0           1/1     Running   0          30m
elasticsearch-es-default-1           1/1     Running   0          2m39s
elasticsearch-es-default-2           1/1     Running   0          107s
elasticsearch-es-default-3           1/1     Running   0          83s
elasticsearch-es-default-4           1/1     Running   0          55s
kibanainstance-kb-545cb5b6d6-mpvzx   1/1     Running   0          22m
```

**Note - Please wait till `Status` should be `Running` and `READY` should be 1/1**
