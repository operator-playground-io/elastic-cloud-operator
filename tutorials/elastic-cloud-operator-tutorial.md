---
title: Elastic Cloud Operator Status Check

description: This tutorial explains check deployment status of elastic cloud operator
---

### Check the Elastic Cloud Operator creation status 

```execute
kubectl get csv -n operators | egrep -i "name|elastic-cloud"
```

```
NAME                             DISPLAY                       VERSION       REPLACES                   PHASE
elastic-cloud-eck.v1.0.0-beta1   Elastic Cloud on Kubernetes   1.0.0-beta1   elastic-cloud-eck.v0.9.0   Succeeded
```

**Note - Please wait till `PHASE` status will be `Succeeded` and then proceed further.**
