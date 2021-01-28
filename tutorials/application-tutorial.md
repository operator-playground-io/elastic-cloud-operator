---
title: Elastic Search Operator Sample Application Tutorial
description: This tutorial explains how to use an Elastic Search cluster created by the operator in an application.
---

### Introduction

Shopping List application is a Node.js application which is deployed as a microservice.
The example also uses Skaffold which handles the workflow for building, pushing and deploying your application, allowing you to focus on what matters most: writing code.

### Code Structure

![codestructure](_images/shopping-app-structure.png)

It follows a simple modular and MVC pattern. There are 2 folders that are of our interest:
- k8s :  This contains all the deployment and service yaml for the application. This defines the deployment and exposure of our application.
- k8s_elastic :  This contains all the deployment and service yaml for creating an Elastic Search cluster (for example to run the aplication locally).


### Try the example

**step 1:** Create an Elastic Search cluster executing these commands. If you already installed the Elastic Search operator and followed the steps to create an Elastic Search cluster you can skip this step.

*  Create PV before creating the instance 

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

*  Execute below command to create the PV

```execute
kubectl create -f elastic_pv.yaml -n operators
```

*  Execute below command to create yaml file

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

*  Execute below command to create cluster

```execute
kubectl create -f elasticsearch_cluster.yaml -n operators
```

The operator automatically creates and manages Kubernetes resources to achieve the desired state of the Elasticsearch cluster. 

*  Execute below command to get status of pods

```execute
kubectl get pods -n operators | egrep -i "name|elasticsearch"
```


It may take up to a few minutes until all the resources are created and the cluster is ready for use. Status should be `running` and READY should be `1/1` .

*  Execute below command to check the health of current ElasticSearch Cluster

```execute
kubectl get elasticsearch -n operators
```

When you create the cluster, there is no HEALTH status and the PHASE is empty. After a while, the PHASE turns into Ready, and HEALTH becomes green. Health will turn into green only when all pods of that cluster are in `READY` state.

You can see that one Pod is in the process of being started:

```execute
kubectl get pods -n operators --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch'
```

**Note - Please wait till `Status` should be `Running` and `READY` should be 1/1 , and then proceed further.**

**step 2:** Install the application sample

Get sample code:
```execute
cd /home/student/projects && git clone https://github.com/Andi-Cirjaliu/edge-elasticsearch-songs.git
```

Navigate to the example:
```execute
cd edge-elasticsearch-songs
```

Copy the secrets in the defaule namespace to be able to access the Elastic Search cluster:
```execute
kubectl get secret elasticsearch-es-elastic-user --namespace=operators --export -o yaml | kubectl apply --namespace=default -f -
```
```execute
kubectl get secret elasticsearch-es-http-ca-internal --namespace=operators --export -o yaml | kubectl apply --namespace=default -f -
```

Setup skaffold default repository to the local one:
```execute
skaffold config set default-repo localhost:5000
```

Install and start the sample. To stop and remove the application you will need to follow the steps from **Clean up the Kubernetes resources**.
```execute
skaffold run
```
Alternatively you can use this command to install the sample, watch for code changes and re-deploy the application automatically.
On exiting the command, Skaffold will automatically stop and delete the sample application. 
```execute
skaffold dev
```

### Access the application

Click on the Key icon on the dashboard and copy the value under the `DNS` section and `IP` field

URL :  http://##DNS.ip##:30200

### Deploy changes to Kubernetes in Dev Mode

Go to Developer Dashboard tab, it will provide you with the IDE along with the integrated terminal.  Click on the bottom status bar and select `TERMINAL`. 

k8s folder contains all the manifest files and defines the deployment stategy for the application.
One can execute them using :

```execute
kubectl apply -f k8s/
```

In this example , we use `Skaffold` which simplifies local devlopment. You can deploy the application is DEV mode which keeps watching for the files changes and on any change, triggers the entire deployment process automatically without the user having to run and manage it manually.

```execute
skaffold dev
```

On exiting the command, Skaffold will automatically destroy all the resources it created with above command.


Also, you can use the `skaffold run` to deploy the changes onto kubernetes as a normal mode. In this mode, the resources created remains unless the user deletes them.

### Clean up the Kubernetes resources

You can delete all the application resources created by executing the following command:

```execute
skaffold delete
```
