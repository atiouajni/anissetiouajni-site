---
title: "Accessing a secure Apache Kafka from Spring Boot"
date: 2022-05-25T12:12:24+02:00
type: 'post'
categories: ['messaging', 'streaming']
tags: ['strimzi', 'kafka', 'kubernetes', 'openshift', 'springboot']
---
Now that we are able to deploy a Kafka cluster using Strimzi (cf. [Simplify Kafka Operational Tasks With Strimzi and Kubernetes]( {{< ref "simplify-kafka-operational-tasks-with-strimzi-and-kubernetes.md" >}})), it will be interesting to see how we can retrieve configuration from the secure cluster to be able to produce and consume events.
In this blog, I will use the Kafka cluster previously created and Spring Boot as the development framework.

But before we start coding, let's try to understand how the authentication and authorization mechanism of our Kafka cluster works. When we deploy a cluster using Strimzi crds, we need to mention which authn and authz mechanisms to use. 
In my last blog, I configured a cluster to use mutual Transport Layer Security (mTLS) for authentication and Simple authorizer (native solution provided by Kafka) for authorization. I will reuse it.

Mutual TLS is one of the most significant practices which uses client TLS certificate to cryptographically ensure the client information while providing an additional layer of security. mTLS works as follows :
- Each Kafka cluster needs its own private-key/certificate pair, and the client uses the certificate to authenticate the cluster.
- Each logical client needs its private-key/certificate pair, and the broker uses the certificate to authenticate the client.

![Mutual TLS handshake](/img/2022-05-25/mtls-process-kafka.png)

I will create a producer and consumer separatly to demonstrate how we can easily connect to the Kafka cluster via applications developed in Spring Boot.

![Kafka producer and consumer diagram](/img/2022-05-25/kafka-scenario.png)


## Prepare your environment

```shell
git clone https://github.com/atiouajni/strimzi-kafka-demo && cd strimzi-kafka-demo
oc new-project <project_name>
```

## Create a new topic

Strimzi provides crds to create topics easily.

```shell
oc create -f manifests/strimzi/kafka-topic.yml
```

## Create a producer and consumer user

On this step, I will create a producer user with write rights on the topic and a second consumer user who can only read from this topic.

```shell
oc create -f manifests/strimzi/kafka-user-producer.yml
oc create -f manifests/strimzi/kafka-user-consumer.yml
```

During the creation of each user, Strimzi takes care of generating the privatekeys, public keys and java keystores and configuring everything for optimal operation. Everything is stored in `secret` with the same name used when creating the user.

```shell
oc describe secret kafka-producer
Name:         kafka-producer
...
Type:  Opaque

Data
====
user.crt:       1472 bytes
user.key:       1708 bytes
user.p12:       2738 bytes
user.password:  12 bytes
ca.crt:         1854 bytes
```

```shell
oc describe secret kafka-consumer
Name:         kafka-consumer
...
Type:  Opaque

Data
====
ca.crt:         1854 bytes
user.crt:       1472 bytes
user.key:       1704 bytes
user.p12:       2738 bytes
user.password:  12 bytes
```

## Retrieve keystores/truststore and passwords

As explained above, to be able to connect to a secure Kafka broker, I will need the private and public keys. And as I'm going to develop JAVA clients, I will have to use keystores and trustores to provide/validate the keys.
Thanks to Strimzi, the keystores/truststores are already created, I just have to retrieve and use them.

Firstly, I retrieve the keystores with their respective passwords:

```shell
oc get secret kafka-producer -o jsonpath='{.data.user\.p12}' | base64 -d > /tmp/kafka-producer.p12
oc get secret kafka-producer -o jsonpath='{.data.user\.password}' | base64 -d > /tmp/kafka-producer.password 

oc get secret kafka-consumer -o jsonpath='{.data.user\.p12}' | base64 -d > /tmp/kafka-consumer.p12 
oc get secret kafka-consumer -o jsonpath='{.data.user\.password}' | base64 -d > /tmp/kafka-consumer.password 
```

Then, I retrieve the truststore and its password. The truststore is linked to the cluster created previously and is contained in a `secret` named `<cluster_name>-cluster-ca-cert`.

> **NOTE** : When you deploy Kafka using the Strimzi Kafka resource, a Secret with the cluster CA certificate is automatically created based on the Kafka cluster name (<cluster_name>-cluster-ca-cert)

```shell
oc get secret my-cluster-cluster-ca-cert -o jsonpath='{.data.ca\.p12}' | base64 -d > /tmp/ca.p12
oc get secret my-cluster-cluster-ca-cert -o jsonpath='{.data.ca\.password}' | base64 -d > /tmp/ca.password 
```

## Build and deploy producer app

Spring Boot brings a set of functionality in its project [spring-kafka](https://spring.io/projects/spring-kafka) to facilitate interaction with a Kafka cluster. I will not go into the details of the development but rather explain what are the essential parameters to be able to connect to my kafka cluster.

You will find the source code of my `producer` application in [strimzi-kafka-demo repository](https://github.com/atiouajni/strimzi-kafka-demo/src/kafka-producer/). The most important files are `ApplicationConstant.java` and properties files under `resources` folder.

To connect to our cluster, Spring Boot framework will rely on common parameters for all kafka clients :

| parameter name                        | description                                                                                                   |
|---------------------------------------|---------------------------------------------------------------------------------------------------------------|
| spring.kafka.security.protocol        | Security protocol used to communicate with brokers.                                                           |
| spring.kafka.bootstrap-servers        | Comma-delimited list of host:port pairs to use for establishing the initial connections to the Kafka cluster. |
| spring.kafka.ssl.trust-store-location | Location of the trust store file.                                                                             |
| spring.kafka.ssl.trust-store-password | Store password for the trust store file.                                                                      |
| spring.kafka.ssl.trust-store-type     | Type of the trust store.                                                                                      |
| spring.kafka.ssl.key-store-location   | Location of the key store file.                                                                               |
| spring.kafka.ssl.key-store-password   | Store password for the key store file.                                                                        |
| spring.kafka.ssl.key-store-type       | Type of the key store.                                                                                        |

And some specific parameters for producer client :

| parameter name                         | description                  |
|----------------------------------------|------------------------------|
| spring.kafka.producer.key-serializer   | Serializer class for keys.   |
| spring.kafka.producer.value-serializer | Serializer class for values. |

> **NOTE** : If you want to add a specific configuration to your environment, you can find the full list [here](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.integration)

Strimzi provide an easy way to expose Kafka cluser inside or outside Kubernetes such as through a route, loadbalancer, nodeport or ingress. In this blog post, I will build an OCI image and deploy a producer inside OpenShift.

**1 - Package a jar**

```shell
mvn clean package spring-boot:repackage -f src/kafka-producer/pom.xml
```

**2 - Build an OCI image**

To create an image from the jar I just built, I will use an [OpenShift Builds](https://docs.openshift.com/container-platform/4.10/cicd/builds/understanding-image-builds.html) feature. You can do the same task with :
 - Docker Build
 - [source-2-image](https://github.com/openshift/source-to-image)
 - [mvn spring-boot:build-image](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#build-image)
 - [odo](https://github.com/redhat-developer/odo)

```shell
oc new-build java:11 --name=kafka-producer --binary
oc start-build kafka-producer --from-file=./target/kafka-producer-0.0.1-SNAPSHOT.jar --follow
```

> **NOTE** : Generated image will be pushed to the internal registry at `image-registry.openshift-image-registry.svc:5000/<project_name>/kafka-producer`

**3 - Update properties file and create a ConfigMap**

To update the properties file, I have to consider several things :

 - I will stay inside OpenShift, so I will use the Kubernetes `service` name for the kafka bootstrap url.
 - I will save sensitive data as a secret (password and p12 store)

Which gives me this configuration :

| parameter name                        | value                                       |
|---------------------------------------|---------------------------------------------|
| spring.kafka.security.protocol        | SSL                                         |
| spring.kafka.bootstrap-servers        | my-cluster-kafka-bootstrap:9093             |
| spring.kafka.ssl.trust-store-location | file:///tmp/secret/truststore.p12                          |
| spring.kafka.ssl.trust-store-password | ${TRUSTSTORE_PASSWORD}                      |
| spring.kafka.ssl.trust-store-type     | PKCS12                                      |
| spring.kafka.ssl.key-store-location   | file:///tmp/secret/keystore.p12              |
| spring.kafka.ssl.key-store-password   | ${KEYSTORE_PASSWORD}                        |
| spring.kafka.ssl.key-store-type       | PKCS12                                      |

> **NOTE** : Update the `src/main/resources/application.yml` file according to your settings.

```shell
oc create configmap kafka-producer-app --from-file=src/kafka-producer/src/main/resources/application.yml
```

**4 - Create Secret**

```shell
oc create secret generic kafka-producer-app --from-file=keystore.p12=/tmp/kafka-producer.p12 --from-file=keystore.password=/tmp/kafka-producer.password --from-file=truststore.p12=/tmp/ca.p12 --from-file=truststore.password=/tmp/ca.password
```

**5 - Deploy producer app**

```shell
oc create -f manifests/kafka-client/kafka-producer-deployment.yml 
```

We can then send a message and check that there is no error :

```shell
oc exec deployment/kafka-producer -- curl -X POST -d 'mon message' -H 'Content-Type: application/json' http://localhost:8080/produce/message
oc logs deployment/kafka-producer
```

## Build and deploy consumer app

```shell
#1 - package a jar
mvn clean package spring-boot:repackage -f src/kafka-consumer/pom.xml

#2 - build an OCI image
oc new-build java:11 --name=kafka-consumer --binary
oc start-build kafka-consumer --from-file=src/kafka-consumer/target/kafka-consumer-0.0.1-SNAPSHOT.jar --follow

#3 - Update properties file and upload a configMap
oc create configmap kafka-consumer-app --from-file=src/kafka-consumer/src/main/resources/application.yml

#4 - Create Sercret with sensitive Data (stores and passwords)
oc create secret generic kafka-consumer-app --from-file=keystore.p12=/tmp/kafka-consumer.p12 --from-file=keystore.password=/tmp/kafka-consumer.password --from-file=truststore.p12=/tmp/ca.p12 --from-file=truststore.password=/tmp/ca.password

#5 - Deploy consumer app
oc create -f manifests/kafka-client/kafka-consumer-deployment.yml 

#6 - check logs
oc logs deployment/kafka-consumer
2022-05-25 07:44:39.819  INFO 1 --- [ntainer#0-0-C-1] io.project.kafka_consumer.KafkaConsumer  : Received record from Topic-Partition 'kafka-demo-topic-0' with Offset '0' -> Key: 'null' - Value 'mon message'
```

Once all these steps have been completed correctly, the consumer application displays in the logs the content of the messages recorded in the `kafka-demo-topic` topic.


