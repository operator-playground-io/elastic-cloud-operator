---
title: Elasticsearch Operator Sample Application Tutorial
description: This tutorial explains how to use an Elasticsearch cluster created by the operator in an application.
---

### Introduction

In this section, we will discuss Shopping List application which is a Node.js application which is deployed as a microservice. The following example also uses Skaffold, a tool that simplifies operational tasks by providing continuous development for Kubernetes applications. Skaffold handles the workflow for building, pushing and deploying your application, allowing you to save time and focus on what matters most: writing code.

### Code Structure

The procedure follows a simple modular and MVC pattern.

![codestructure](_images/shopping-app-structure.png)

The two folders of our interest are:

-	k8s: This folder contains all the deployment and service yaml files for the application. This also defines the deployment and exposure of our application.
- k8s_elastic: This folder contains all the deployment and service yaml files for creating an Elasticsearch cluster to run the application locally.



### Try the example

**Step-by-step Example**

**Step 1:** Create an Elasticsearch cluster.

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

**Note:** If you have Elasticsearch operator already installed and have followed the steps to create an Elasticsearch cluster, you can skip this step.

-	Before creating the instance, we first create a PersitentVolume (PV) and define the spec in.
-	Use the command below to create the PV.

```execute
kubectl create -f elastic_pv.yaml -n operators
```

-	Execute the command below to create a yaml file.

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

This small specification causes the operator to deploy a single node Elasticsearch cluster named ElasticSearch. The cluster will automatically have all its communications secured using Transport Layer Security (TLS)

This specification lets the operator to deploy a single node Elasticsearch cluster `ElasticSearch`. This cluster will automatically have all its communications secured using Transport Layer Security (TLS).

-	Execute the command below to create your cluster.

```execute
kubectl create -f elasticsearch_cluster.yaml -n operators
```

It may take up to a few minutes for the resources to be created and the cluster to be ready for use. Please verify that Status is Running and READY value is 1/1.

The content for step 1 is repeating and matches with previous md file description. PLEASE REFER ABOVE REMARKS FOR THE SAME.

**Step 2:** Install the Sample Application 

- Get the sample code:

```execute
cd /home/student/projects && git clone https://github.com/operator-playground-io/elasticsearch-sample.git
```

- Navigate to the example:
```execute
cd elasticsearch-sample
```

- Copy the secrets in the default namespace to be able to access the Elasticsearch cluster:
```execute
kubectl get secret elasticsearch-es-elastic-user --namespace=operators --export -o yaml | kubectl apply --namespace=default -f -
```
```execute
kubectl get secret elasticsearch-es-http-ca-internal --namespace=operators --export -o yaml | kubectl apply --namespace=default -f -
```

- Setup skaffold default repository to the local one:
```execute
skaffold config set default-repo localhost:5000
```

Install and start the sample application. To stop and remove the application you will need to follow the steps from Clean up the Kubernetes resources.

**Clean up the Kubernetes resources**.
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

In this example , we use `Skaffold` which simplifies local development. You can deploy the application is DEV mode which keeps watching for the files changes and on any change, triggers the entire deployment process automatically without the user having to run and manage it manually.

```execute
skaffold dev
```

On exiting the command, Skaffold will automatically destroy all the resources it created with above command.


Also, you can use the `skaffold run` to deploy the changes onto Kubernetes as a normal mode. In this mode, the resources created remains unless the user deletes them.

### Clean up the Kubernetes resources

You can delete all the application resources created by executing the following command:

```execute
skaffold delete
```
