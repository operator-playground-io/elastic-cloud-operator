---
title: Kibana Instance deployment
description: Tutorial to create an instance of Kibana and associate it with Elasticsearch Cluster
---

### Create a Kibana Instance

- Declare a Kibana instance and associate it with your Elasticsearch cluster.

```execute
cat <<'EOF' > kibana_instance.yaml
apiVersion: kibana.k8s.elastic.co/v1beta1
kind: Kibana
metadata:
  name: kibanainstance
spec:
  version: 7.4.2
  count: 1
  elasticsearchRef:
    name: elasticsearch
EOF
```

- Execute the command below to create a Kibana instance.

```execute
kubectl create -f kibana_instance.yaml -n operators
```

- You should find see the output like below:

```
kibana.kibana.k8s.elastic.co/kibanainstance created
```

### Monitor Kibana Instance health and progress of create operation

- Like Elasticsearch, you can also retrieve details about Kibana instances. See the command below.

```execute
kubectl get kibana -n operators
```

You will see the output like below:

```
NAME             HEALTH   NODES   VERSION   AGE
kibanainstance   green    1       7.4.2     62s
```

- Get details about the associated pods using the below command.

```execute
kubectl get pod -n operators --selector='kibana.k8s.elastic.co/name=kibanainstance'
```

Sample output is given below.

```
NAME                                 READY   STATUS    RESTARTS   AGE
kibanainstance-kb-66f574459f-vvbx8   1/1     Running   0          91s
```
**Note - Please wait until the `STATUS` is `Running` and `READY` value is `1/1`, then proceed.**

### Access Kibana Service

- To access the Kibana service externally, update it to use NodePort.

```execute
kubectl get service kibanainstance-kb-http -n operators --output yaml > /tmp/kb_test.yaml
sed -i "s/type: .*/type: NodePort/g" /tmp/kb_test.yaml
kubectl patch svc kibanainstance-kb-http -n operators -p "$(cat /tmp/kb_test.yaml)"
```

- Now, update NodePort to 30449.

```execute
kubectl get service kibanainstance-kb-http -n operators --output yaml > /tmp/kb_test1.yaml
sed -i "s/    nodePort: .*/    nodePort: 30449/g" /tmp/kb_test1.yaml
kubectl patch svc kibanainstance-kb-http -n operators -p "$(cat /tmp/kb_test1.yaml)"
```

- Now you can get the details of Kibana service by executing the following command.

```execute
kubectl get service kibanainstance-kb-http -n operators
```

Sample output is given below:

```
NAME                     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kibanainstance-kb-http   NodePort   10.96.67.171   <none>        5601:30449/TCP   7m42s
```

Make sure that to you access by logging in with username **elastic** via a web browser.

Use the following commands to view the username and password:

```execute
export password=`kubectl get secret elasticsearch-es-elastic-user -n operators -o=jsonpath='{.data.elastic}' | base64 --decode; echo`
echo "User: elastic"
echo "Password: $password"
```

Click on <a href="https://##DNS.ip##:30449" target="_blank">https://##DNS.ip##:30449</a> to access Kibana Dashboard

This will redirect you to the login page as shown below:

![](_images/kibana_localhost.png)

Once you're logged in , you will see the following  page:

![](_images/kibana_get_started.png)

Click on `Explore on my own` option.

You will see the dashboard like below:

![](_images/kibana_login.png)

You’ve successfully deployed Kibana instance to Elasticsearch Cluster.
