# 安全地连接红帽集成服务注册与红帽 AMQ 流

> 原文：<https://developers.redhat.com/blog/2021/04/07/securely-connect-red-hat-integration-service-registry-with-red-hat-amq-streams>

[Red Hat Integration Service Registry](/blog/2019/12/16/getting-started-with-red-hat-integration-service-registry/)是一个基于 [Apicurio](https://www.apicur.io/) 开源项目的数据存储。在我之前的文章中，我向您展示了[如何将 Spring Boot 与服务注册中心](/blog/2021/02/15/integrating-spring-boot-with-red-hat-integration-service-registry/)集成。在本文中，您将了解如何将 Service Registry 连接到安全的 [Red Hat AMQ 流](/products/amq/overview)集群。

## 将服务注册表与 AMQ 流连接

Service Registry 包括一组用于存储 API、规则和验证的可插拔存储选项。由[红帽 AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet)提供的基于 [Kafka](/topics/kafka-kubernetes) 的存储选项，适用于为运行在[红帽 OpenShift](/products/openshift/overview) 上的 Kafka 集群配置持久存储的生产环境。

在生产环境中，安全性不是可选的，AMQ 流必须为您连接的每个组件提供安全性。安全性由*身份验证*和*授权*定义，前者确保客户端安全连接到 Kafka 集群，后者指定哪些用户可以访问哪些资源。我将向您展示如何使用 AMQ 流和服务注册来设置身份验证和授权。

## AMQ 河运营商

AMQ 流和服务注册中心提供了一组 [OpenShift 操作符](/topics/kubernetes/operators)，可从 [OpenShift 操作符中心](https://operatorhub.io/)获得。开发人员使用这些操作符来打包、部署和管理 OpenShift 应用程序组件。

[AMQ 流操作符](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/amq_streams_on_openshift_overview/index#overview-components_str)提供了一组定制资源定义(CRD)来描述 Kafka 部署的组件。这些对象——即 Zookeeper、Brokers、Users 和 Connect——提供了我们用来管理 Kafka 集群的 API。AMQ 流运营商管理认证、授权和用户的生命周期。

AMQ 流[集群操作符](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/amq_streams_on_openshift_overview/index#overview-components-cluster-operator-str)管理 [Kafka 模式引用](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/using_amq_streams_on_openshift/index#type-Kafka-reference)资源，该资源声明要使用的 Kafka 拓扑和特性。

AMQ 流[用户操作符](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/amq_streams_on_openshift_overview/index#overview-concepts-user-operator-str)管理 [KafkaUser 模式引用](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/using_amq_streams_on_openshift/index#type-KafkaUser-reference)资源。该资源为 AMQ 流的实例声明用户，包括用户的身份验证、授权和配额定义。

## 服务注册运营商

[服务注册操作符](https://www.apicur.io/registry/docs/apicurio-registry/1.3.3.Final/getting-started/assembly-installing-registry-openshift.html#installing-registry-operatorhub)提供了一组 CRD 来描述服务注册部署组件，如存储、安全性和副本。这些对象共同提供了管理服务注册中心实例的 API。

服务注册中心操作者使用 [ApicurioRegistry 模式引用](https://github.com/Apicurio/apicurio-registry-operator/blob/master/deploy/crds/apicur.io_apicurioregistries_crd.yaml)来管理服务注册中心的生命周期。`ApicurioRegistry`声明了服务注册中心的拓扑和主要特性。Apicurio 操作符管理`ApicurioRegistry`对象。

## 使用 AMQ 流进行身份验证

红帽 AMQ 流支持以下认证机制:

*   SASL 急停-SHA-512
*   传输层安全性(TLS)客户端身份验证
*   OAuth 2.0 基于令牌的身份验证

这些机制在每个监听器的`Kafka`定义中的`authentication`块中声明。每个侦听器实现定义的身份验证机制，因此客户端应用程序必须使用确定的机制进行身份验证。

### 认证 AMQ 流集群的两种方法

首先，让我们看看如何激活 AMQ 流集群中的每个机制。我们需要在服务注册中心识别在 AMQ 流集群中激活的认证机制。服务注册中心只允许[紧急停堆-SHA-512](https://tools.ietf.org/id/draft-melnikov-scram-sha-512-01.html) 和 TLS 作为认证机制。我将向您展示如何使用这些机制来配置身份验证。

#### 用 SCRAM-SHA-512 认证 Kafka 集群

下面的`Kafka`定义为安全监听器声明了一个使用 SCRAM-SHA-512 认证保护的 Kafka 集群(使用 TLS 协议保护):

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
 name: my-kafka
spec:
 kafka:
   listeners:
     tls:
       authentication:
         type: scram-sha-512

```

应用此配置会创建一组存储 TLS 证书的机密。我们需要知道的允许安全连接的秘密被声明为`my-kafka-cluster-ca-cert`。请注意，我们稍后将需要这个值。

下面的`KafkaUser`定义声明了一个具有 SCRAM-SHA-512 认证的 Kafka 用户:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
 name: service-registry-scram
 labels:
   strimzi.io/cluster: my-kafka
spec:
 authentication:
   type: scram-sha-512

```

应用此配置会创建一个密码(用户名),并将其存储在存储用户凭据的位置。该密码包含生成的用于向 Kafka 集群进行身份验证的密码:

```
$ oc get secrets
NAME                    TYPE      DATA   AGE
service-registry-scram  Opaque    1      4s

```

#### 使用 TLS 认证 Kafka 集群

下面的`Kafka`定义为安全监听器声明了一个使用 TLS 认证保护的 Kafka 集群(使用 TLS 协议保护):

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
 name: my-kafka
spec:
 kafka:
   listeners:
     tls:
       authentication:
         type: tls

```

应用此配置会创建一组存储 TLS 证书的机密。我们需要知道的允许安全连接的秘密被声明为`my-kafka-cluster-ca-cert`。请注意，我们稍后将需要这个值。

下面的`KafkaUser`定义声明了一个具有 TLS 认证的 Kafka 用户:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
 name: service-registry-tls
 labels:
   strimzi.io/cluster: my-kafka
spec:
 authentication:
   type: tls

```

应用此配置会创建一个密码(用户名),并将其存储在存储用户凭据的位置。此密码包含向 Kafka 集群进行身份验证的有效客户端证书:

```
$ oc get secrets
NAME                    TYPE      DATA   AGE
Service-registry-tls    Opaque    1      4s

```

### 服务注册中心认证

为了识别 AMQ 流集群中激活的身份验证机制，我们需要使用`ApicurioRegistry`定义相应地部署服务注册中心:

```
apiVersion: apicur.io/v1alpha1
kind: ApicurioRegistry
metadata:
  name: service-registry
spec:
  configuration:
    persistence: "streams"
    streams:
      bootstrapServers: "my-kafka-kafka-bootstrap:9093"

```

**注意**:在撰写本文时，只有在 AMQ 流 TLS 侦听器中激活了身份验证机制时，Service Registry 才能连接到该侦听器(通常在端口 9093)。`ApicurioRegistry`定义的`boostrapServers`属性必须指向那个监听器端口。

#### 使用 SCRAM-SHA-512 的服务注册认证

下面的`ApicurioRegistry`定义通过 SCRAM-SHA-512 认证声明了与用户的安全连接:

```
apiVersion: apicur.io/v1alpha1
kind: ApicurioRegistry
metadata:
  name: service-registry
spec:
  configuration:
    persistence: "streams"
    streams:
      bootstrapServers: "my-kafka-kafka-bootstrap:9093"
      security:
        scram:
          user: service-registry-scram
          passwordSecretName: service-registry-scram
          truststoreSecretName: my-kafka-cluster-ca-cert
```

我们需要确定该对象中的以下值:

*   **用户**:安全连接的用户名。
*   **PasswordSecretName** :保存密码的秘密名称。
*   **truststorescreetname**:部署的 Kafka 集群的带有证书颁发机构(CA)证书的秘密名称。

#### 使用 TLS 的服务注册表身份验证

下面的`ApicurioRegistry`定义声明了使用 TLS 认证的用户的安全连接:

```
apiVersion: apicur.io/v1alpha1
kind: ApicurioRegistry
metadata:
  name: service-registry
spec:
  configuration:
    persistence: "streams"
    streams:
      bootstrapServers: "my-kafka-kafka-bootstrap:9093"
      security:
        tls:
          keystoreSecretName: service-registry-tls
          truststoreSecretName: my-kafka-cluster-ca-cert
```

我们需要在该对象中识别的值是:

*   **KeystoreSecretName** :带有客户端证书的用户机密的名称。
*   **truststorescreetname**:带有已部署 Kafka 集群的 CA 证书的秘密名称。

## AMQ 流的授权

AMQ 流支持对用于客户端连接的所有监听器全局使用`SimpleACLAuthorizer`进行授权。这种机制使用访问控制列表(ACL)来定义哪些用户可以访问哪些资源。

如果在 Kafka 集群中应用授权，则拒绝是默认的访问控制。侦听器必须为希望在 Kafka 集群中操作的每个用户声明不同的规则。

### 卡夫卡的定义

下面的`Kafka`定义激活 Kafka 集群中的授权:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
 name: my-kafka
spec:
 kafka:
   authorization:
     type: simple
```

### KafkaUser 定义

在`KafkaUser`定义中为每个用户声明了一个 ACL。`acls`部分(见下文)包括一个资源列表，其中每个资源都被声明为一个新规则:

*   **资源类型**:标识 Kafka 中管理的对象的类型；对象包括主题、使用者组、集群、事务 id 和委托令牌。
*   **资源名称**:标识应用规则的资源。资源名可以定义为一个*文字*，以标识一个资源，或者定义为一个*前缀模式*，以标识一系列资源。
*   **操作**:声明允许的操作种类。每个资源类型可用操作的完整列表可在[这里](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/using_amq_streams_on_openshift/index#simple-acl-str)获得。

为了让服务注册用户成功地使用我们的安全 AMQ 流集群，我们必须声明以下规则，指定允许用户做什么:

*   读懂自己的消费群体。
*   创建、读取、写入和描述一个全局 ID 主题(`global-id-topic`)。
*   创建、读取、写入和描述存储主题(`storage-topic`)。
*   创建、读取、写入和描述自己的本地 changelog 主题。
*   在其自己的本地组上描述和编写事务 id。
*   阅读消费者补偿主题(`__consumer_offsets`)。
*   阅读事务状态主题(`__transaction_state`)。
*   在群集上以幂等方式写入。

### ACL 定义

以下是 ACL 定义的示例:

```
    acls:
      # Group Id to consume information for the different topics used by the Service Registry.
      # Name equals to metadata.name property in ApicurioRegistry object
      - resource:
          type: group
          name: service-registry
        operation: Read
      # Rules for the Global global-id-topic
      - resource:
          type: topic
          name: global-id-topic
        operation: Read
      - resource:
          type: topic
          name: global-id-topic
        operation: Describe
      - resource:
          type: topic
          name: global-id-topic
        operation: Write
      - resource:
          type: topic
          name: global-id-topic
        operation: Create
      # Rules for the Global storage-topic
      - resource:
          type: topic
          name: storage-topic
        operation: Read
      - resource:
          type: topic
          name: storage-topic
        operation: Describe
      - resource:
          type: topic
          name: storage-topic
        operation: Write
      - resource:
          type: topic
          name: storage-topic
        operation: Create
      # Rules for the local topics created by our Service Registry instance
      # Prefix value equals to metadata.name property in ApicurioRegistry object
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Read
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Describe
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Write
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Create
      # Rules for the local transactionalsIds created by our Service Registry instance
      # Prefix equals to metadata.name property in ApicurioRegistry object
      - resource:
          type: transactionalId
          name: service-registry-
          patternType: prefix
        operation: Describe
      - resource:
          type: transactionalId
          name: service-registry-
          patternType: prefix
        operation: Write
      # Rules for internal Apache Kafka topics
      - resource:
          type: topic
          name: __consumer_offsets
        operation: Read
      - resource:
          type: topic
          name: __transaction_state
        operation: Read
      # Rules for Cluster objects
      - resource:
          type: cluster
        operation: IdempotentWrite
```

注意，在 AMQ 流中激活授权不会影响`ApicurioRegistry`的定义。它只与`KafkaUser`对象中的正确 ACL 定义相关。

## 摘要

将 Service Registry 的安全功能连接到安全的 AMQ 流群集使您的生产环境能够提示有关您的安全要求的警告。本文介绍了与安全需求相关的服务注册中心和 AMQ 流组件，并向您展示了如何成功地应用它们。

为了更深入的理解和分析，请参考以下参考文献:

*   [在 OpenShift 上使用 AMQ 流，第 12 章:安全性](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/using_amq_streams_on_openshift/index#security-str)
*   [使用 AMQ 流的用户操作符](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/using_amq_streams_on_openshift/index#assembly-using-the-user-operator-str)(红帽集成 2020-Q2 文档)
*   [服务注册中心入门](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q2/html/getting_started_with_service_registry/index)(红帽集成 2020-Q2 文档)

*Last updated: April 5, 2021*