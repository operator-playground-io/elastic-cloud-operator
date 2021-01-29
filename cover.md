<h1 align="center">Elastic Cloud on Kubernetes</h1>

![Logo](_images/logo.png)

Elastic Cloud on Kubernetes automates the deployment, provisioning, management, and orchestration of Elasticsearch, Kibana, APM Server, Enterprise Search, and Beats on Kubernetes based on the operator pattern.

### Current features

*  Elasticsearch, Kibana, APM Server, Enterprise Search, and Beats deployments
*  TLS Certificates management
*  Safe Elasticsearch cluster configuration & topology changes
*  Persistent volumes usage
*  Custom node configuration and attributes
*  Secure settings keystore updates

### Elastic Cloud Operator Architectural Flow

Elastic cloud operator is used to create instances and with help of Kibana we can access elastic cloud. These instances will be used by external applications

![](_images/eck_arch.png)


### Supported versions

*  Kubernetes 1.12+ or OpenShift 3.11+
*  Elasticsearch, Kibana, APM Server: 6.8+, 7.1+
*  Enterprise Search: 7.7+
*  Beats: 7.0+

### Objective of tutorial

This course provides the hands-on with Elastic Cloud Operator, including:

- Deploy Elastic Cloud Operator on Kubernetes
- Deploy an Elasticsearch Cluster
- Deploy Kibana Cluster
- Monitor Cluster Health and Creation Progress
- Access Elasticsearch and upgrade deployment

For general questions, please see the Elastic [forums](https://discuss.elastic.co/c/eck).
