---
title: "Simplify Kafka Operational Tasks With Strimzi and Kubernetes"
date: 2022-03-01T16:49:36+01:00
type: 'post'
categories: ['messaging', 'streaming']
tags: ['strimzi', 'kafka', 'kubernetes', 'openshift']
---
More and more companies are integrating the Kafka distributed event streaming platform into their information system. The reasons can be multiple, it can range from setting up event-oriented architectures, doing change data capture ( CDC ), or setting up a data-centric strategy (Kafka as a message bus). And as we all know, each new technology introduces challenges. The idea of this post is not to list the advantages and disadvantages of Kafka but rather to present the [Strimzi](https://strimzi.io/) project and how it responds to certain challenges that can be encountered.
 
Strimzi is an Open Source project and is integrated into an enterprise product called Red Hat AMQ Streams. It includes some very interesting features to simplify the process of running Apache Kafka in a Kubernetes cluster. It takes advantage of [Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) to automate all operational tasks that are usually tedious and complicated to set up. It brings benefits in day 1 like deploying and running Kafka cluster and also in day 2 like cluster upgrade.
 
![Strimzi Operators](/img/2022-03-01/strimzi-operators.png)
 
As I had the opportunity to recently work on this project, it is interesting that I share my progress with you. I used Strimzi on an OpenShift cluster, however the steps should be the same for a vanilla Kubernetes cluster.
 
To introduce the Strimzi features, I will simulate a production release of a Kafka cluster:
 
1. A cluster of 3 highly available brokers
2. Communication is always encrypted
3. Access is restricted to authenticated and authorized users
4. A complete monitoring solution for the Kafka platform
 
![Strimzi Kafka Cluster](/img/2022-03-01/architecture-kafka-3-nodes.png)
  
## Deploy Strimzi Operator
 
Once I checked the [prerequisites](https://strimzi.io/docs/operators/in-development/full/deploying.html#deploy-prereqs-str) and created a dedicated project called (`kafka`), the first step is to install Strimzi Operator Lifecycle Manager ([OLM](https://github.com/operator-framework/operator-lifecycle-manager)) :
 
```shell
git clone https://github.com/atiouajni/strimzi-kafka-demo && cd strimzi-kafka-demo
oc new-project kafka
oc create -f manifests/strimzi/strimzi-operator-subscription.yaml
```
 
Strimzi will create necessary kubernetes resources like ClusterRole, RoleBinding and ServiceAccount and also will extend the Kubernetes API with CustomResourceDefinitions. The list of created resources can be found in the [Strimzi GitHub](https://github.com/strimzi/strimzi-kafka-operator/tree/main/install/cluster-operator) and is represented by the following figure.
 
![Strimzi deployed resources](/img/2022-03-01/strimzi-deployed-resources.png)
 
We can verify that the installation is complete :
 
```shell
oc get events -w
LAST SEEN   TYPE      REASON                OBJECT                                                   MESSAGE
80m         Normal    Scheduled             pod/strimzi-cluster-operator-v0.28.0-6b56cbd7cc-b9gss    Successfully assigned blog/strimzi-cluster-operator-v0.28.0-6b56cbd7cc-b9gss to master-1
...
0s          Normal    InstallSucceeded      clusterserviceversion/strimzi-cluster-operator.v0.28.0   install strategy completed with no errors
```
 
> **NOTE** : Only cluster administrators can install Operators to an OpenShift cluster.
 
## Deploy Prometheus and Grafana Operators
 
To provide metrics information, Strimzi uses Prometheus and [JMX Exporter](https://github.com/prometheus/jmx_exporter) Java agent. This feature helps us to monitor all the components (Kafka, Zookeeper, Strimzi operators...) easily using Prometheus to store the metrics and Grafana Dashboards to expose them. Strimzi provides [docs and manifests](https://strimzi.io/docs/operators/latest/deploying.html#assembly-metrics-config-files-str) to facilitate implementation.
 
```shell
oc create -f manifests/prometheus/prometheus-operator-subscription.yaml
oc create -f manifests/grafana/grafana-operator-subscription.yaml
```
 
## Create a secure Kafka cluster
 
Strimzi makes the broker setup part a lot easier. Through a single `kind: Kafka` manifest, we are able to deploy multiple Zookeeper and Kafka instances. Cluster Operator automatically sets up TLS certificates for data encryption and authentication within the cluster. It only remains to say that we want to expose an internal and external kafka listener with TLS encryption and enable authentication and Kafka simple authorization. External clients can access the Kafka through an OpenShift Route.
 
> **NOTE** : If you do not have access to the Quay.io Registry (Disconnected platform) or want to use your own container repository, you can push container images to your own registry by following [this guide](https://strimzi.io/docs/operators/in-development/full/deploying.html#container-images-str).
 
```shell
oc create -f manifests/strimzi/kafka-cluster.yml
```
 
Here is an image to illustrate my configuration :
 
![Kafka Architecture](/img/2022-03-01/architecture-kafka-details.png)
 
We can verify that the deployment is done without issue, it should take a few minutes :

```shell
oc get events -w
LAST SEEN   TYPE      REASON                  OBJECT                                                      MESSAGE
11s         Normal    Provisioning            persistentvolumeclaim/data-my-cluster-zookeeper-0           External provisioner is provisioning volume for claim "kafka/data-my-cluster-zookeeper-0"
...
8s          Normal    SuccessfulCreate        statefulset/my-cluster-zookeeper                            create Pod my-cluster-zookeeper-0 in StatefulSet my-cluster-zookeeper successful
8s          Normal    SuccessfulCreate        statefulset/my-cluster-zookeeper                            create Pod my-cluster-zookeeper-1 in StatefulSet my-cluster-zookeeper successful
8s          Normal    SuccessfulCreate        statefulset/my-cluster-zookeeper                            create Pod my-cluster-zookeeper-2 in StatefulSet my-cluster-zookeeper successful
...
0s          Normal    SuccessfulCreate        statefulset/my-cluster-kafka                                create Pod my-cluster-kafka-0 in StatefulSet my-cluster-kafka successful
0s          Normal    SuccessfulCreate        statefulset/my-cluster-kafka                                create Pod my-cluster-kafka-1 in StatefulSet my-cluster-kafka successful
0s          Normal    SuccessfulCreate        statefulset/my-cluster-kafka                                create Pod my-cluster-kafka-2 in StatefulSet my-cluster-kafka successful
...
```
 
```shell
oc get pods -w
NAME                                                   READY   STATUS    RESTARTS   AGE
grafana-operator-controller-manager-6f456b8f58-vvqd8   2/2     Running   0          2m
my-cluster-entity-operator-6766d76694-fwv2j            3/3     Running   0          82s
my-cluster-kafka-0                                     1/1     Running   0          82s
my-cluster-kafka-1                                     1/1     Running   0          82s
my-cluster-kafka-2                                     1/1     Running   0          82s
my-cluster-kafka-exporter-57f58dd7fc-lh2jb             1/1     Running   0          82s
my-cluster-zookeeper-0                                 1/1     Running   0          82s
my-cluster-zookeeper-1                                 1/1     Running   0          82s
my-cluster-zookeeper-2                                 1/1     Running   0          82s
prometheus-operator-5b9b44b48f-jngfx                   1/1     Running   0          2m
strimzi-cluster-operator-v0.28.0-6b56cbd7cc-ntlf6      1/1     Running   0          82s
```
By default, Strimzi automatically creates a NetworkPolicy resource for every listener that is enabled on a Kafka broker. This NetworkPolicy allows applications to connect to listeners in all namespaces.

```shell
oc get networkpolicy
NAME                                  POD-SELECTOR                           AGE
my-cluster-network-policy-kafka       strimzi.io/name=my-cluster-kafka       1h
my-cluster-network-policy-zookeeper   strimzi.io/name=my-cluster-zookeeper   1h
```

```shell
oc describe networkpolicy my-cluster-network-policy-kafka
Name:         my-cluster-network-policy-kafka
...
Spec:
  PodSelector:     strimzi.io/name=my-cluster-kafka
  Allowing ingress traffic:
    To Port: 9090/TCP
    From:
      PodSelector: strimzi.io/name=my-cluster-kafka
    ----------
    To Port: 9091/TCP
    From:
      PodSelector: strimzi.io/kind=cluster-operator
    From:
      PodSelector: strimzi.io/name=my-cluster-kafka
    From:
      PodSelector: strimzi.io/name=my-cluster-entity-operator
    From:
      PodSelector: strimzi.io/name=my-cluster-kafka-exporter
    From:
      PodSelector: strimzi.io/name=my-cluster-cruise-control
    ----------
    To Port: 9093/TCP
    From: <any> (traffic not restricted by source)
    ----------
    To Port: 9094/TCP
    From: <any> (traffic not restricted by source)
    ----------
    To Port: 9404/TCP
    From: <any> (traffic not restricted by source)
  Not affecting egress traffic
  Policy Types: Ingress
```

My configuration does not show all Strimzi capabilities. One of the interesting features is that it allows brokers to be distributed over several availability zones, data centers or in several machine room to maintain resilience. This can be done through the [rack awareness](https://strimzi.io/docs/operators/in-development/configuring.html#type-Rack-reference) and [pod affinity](https://strimzi.io/docs/operators/in-development/configuring.html#assembly-scheduling-str) features.

Even though I used Mutual TLS and simple authz, Strimzi supports more authentication and authorization mechanisms:
 
|       Authentication                 |        Authorization                   |
|:------------------------------------:|:--------------------------------------:|
|   Mutual TLS client authentication   |          Simple authorization          |
| SASL SCRAM-SHA-512                   | OAuth 2.0 authorization (Keycloak only)|
| OAuth 2.0 token based authentication | Open Policy Agent (OPA) authorization  |
| Custom authentication                | Custom authorization                   |
 
## Deploy Metrics Platform
 
Prometheus and Grafana Operator help us to deploy local Prometheus, Alertmanager and Grafana instances where we could visualize and monitor Apache Kafka cluster metrics.
 
I'm using [yq](https://github.com/mikefarah/yq) command to edit files before creating resources. If you don't have it, you can edit the manifest manually. 
You can skip `yq` commands if you deploy on a namespace called `kafka`.
 
```shell
yq -i '.spec.namespaceSelector.matchNames=["<project-name>"]' manifests/prometheus/PodMonitor-strimzi.yaml
yq -i '.subjects[].namespace="<project-name>"' manifests/prometheus/Prometheus-instance.yaml
yq -i '.spec.alerting.alertmanagers[].namespace="<project-name>"' manifests/prometheus/Prometheus-instance.yaml
 
oc create -f manifests/grafana/Grafana-instance.yaml
oc create -f manifests/prometheus/Prometheus-instance.yaml
oc create -f manifests/prometheus/Alertmanager-instance.yaml
```
 
We check that everything starts correctly:
 
```shell
oc get pods
NAME                                                   READY   STATUS    RESTARTS   AGE
grafana-deployment-76548b566d-9ldqs                    1/1     Running   0          1m
prometheus-prometheus-0                                2/2     Running   0          1m
alertmanager-alertmanager-0                            2/2     Running   0          1m
...
```
 
We specify Prometheus to monitor the pods and collect the specified metric endpoints:
 
```shell
oc create -f manifests/prometheus/PodMonitor-strimzi.yaml
oc create -f manifests/prometheus/PrometheusRule-alert-rules.yaml
```
 
And to finish, we import Grafana datasource and dashboards:
 
```shell
oc create -f manifests/grafana/GrafanaDataSource-prometheus.yaml \
-f manifests/grafana/GrafanaDashboard-strimzi-kafka.yaml \
-f manifests/grafana/GrafanaDashboard-strimzi-zookeeper.yaml \
-f manifests/grafana/GrafanaDashboard-strimzi-kafka-exporter.yaml \
-f manifests/grafana/GrafanaDashboard-strimzi-operators.yaml
```
 
We can now visualize our new dashboards by retrieving the Route to access to Grafana :
 
```shell
oc get route grafana-route -o jsonpath='{.spec.host}'
```
 
From a web browser, the list of dashboards is accessible at the url `<hostname>/dashboards`
 
![Grafana dashboards](/img/2022-03-01/grafana-dashboards.png)

|       Kafka dashboard                                            | Zookeeper dashboard                   |
|:----------------------------------------------------------------:|:-----------------------------------------------------------------------:|
| ![Kafka dashboard](/img/2022-03-01/strimzi-kafka-dashboard.png)  | ![Zookeeper dashboard](/img/2022-03-01/strimzi-zookeeper-dashboard.png) | 

## Upgrading Kafka
 
The [upgrade](https://strimzi.io/docs/operators/in-development/full/deploying.html#assembly-upgrade-str) part is very well documented by Strimzi. However, to do the tests and demonstrate that there is no loss of connection to the cluster or loss of data, I must first look at the implementation of producers and consumers clients. I will do this in another blog post. So stay tuned !
 
## Encountered issues
 
When I deployed the first time my Grafana Dashboard, I didn't get any metrics from Prometheus. Every Grafana panel was empty or N/A.
I noticed that when Grafana made a request to Prometheus, it returned an empty response `https://<server-hostname>/api/datasources/proxy/1/api/v1/query?query=XXX`
So I made the Prometheus url accessible via a Route and I checked that the metrics were indeed collected by Prometheus.
 
```shell
oc create -f manifests/prometheus/prometheus-route.yaml
```
 
As soon as I accessed the UI, I saw this message in Prometheus UI :
 
```
Warning: Error fetching server time: Detected ***** seconds time difference between your browser and the server. Prometheus relies on accurate time and time drift might cause unexpected query results.
```
 
![Prometheus Warning time difference](/img/2022-03-01/prometheus-warning.png)
 
And indeed, my Openshift server was not at the right time. I updated the time on my OCP nodes and everything started working !
 
```shell
date +%T -s "10:13:13"
```
 
## Summary
 
We are able to deploy a Kafka cluster easily without the configuration complexity that we can have during traditional installation. Strimzi provides an abstraction layer to simplify operational tasks. Deploying, scaling and monitoring become tasks much more mastered and easy to achieve.