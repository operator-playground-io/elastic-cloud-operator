---
title: etcd Operator Tutorial
description: This tutorial explains basics about etcd Operator
---

### Introduction

Built on the Kubernetes Operator pattern, Elastic Cloud on Kubernetes (ECK) extends the basic Kubernetes orchestration capabilities to support the setup and management of Elasticsearch on Kubernetes.

With Elastic Cloud on Kubernetes you can streamline all those critical operations, such as:

- Managing and monitoring multiple clusters
- Scaling cluster capacity up and down
- Changing cluster configuration
- Scheduling backups
- Securing clusters with TLS certificates
- Setting up hot-warm-cold architectures with availability zone awareness

Elastic Cloud on Kubernetes automates the deployment, provisioning, management, and orchestration of Elasticsearch, Kibana and the APM Server on Kubernetes.

**Current features:**

- Elasticsearch, Kibana and APM Server deployments
- TLS Certificates management
- Safe Elasticsearch cluster configuration & topology changes
- Persistent volumes usage
- Custom node configuration and attributes
- Secure settings keystore updates


**Elastic Cloud Operator Architectural Flow**

To create elastic-cloud user need to install olm and deploy elastic cloud operator on kubernetes. elastic cloud operator is used to create instances and with help of kibana we can access elastic cloud. This instances will be used by external applications

![](_attachments/eck_arch.png)

### Check the Elastic Cloud Operator creation status 

```execute
kubectl get csv -n operators | egrep -i "name|elastic-cloud"
```

```
NAME                             DISPLAY                       VERSION       REPLACES                   PHASE
elastic-cloud-eck.v1.0.0-beta1   Elastic Cloud on Kubernetes   1.0.0-beta1   elastic-cloud-eck.v0.9.0   Succeeded
```

**Note - Please wait till `PHASE` status will be `Succeeded` and then proceed further.**
