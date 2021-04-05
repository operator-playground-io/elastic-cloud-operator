---
title: Elastic Cloud Operator Status Check
description: This tutorial explains check deployment status of elastic cloud operator
---

### Check the Elastic Cloud Operator creation status 

Check the status of Elastic search Operator created in your cluster.

Use the command below to get the status of Elasticsearch Operator post creation process.

```execute
kubectl get csv -n operators | egrep -i "name|elastic-cloud"
```

```
NAME                             DISPLAY                       VERSION       REPLACES                   PHASE
elastic-cloud-eck.v1.0.0-beta1   Elastic Cloud on Kubernetes   1.0.0-beta1   elastic-cloud-eck.v0.9.0   Succeeded
```

**Note - Please wait untill the `PHASE` status is `Succeeded` and then proceed.**
