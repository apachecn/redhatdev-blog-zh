# 在 Kafka Connect 配置中使用秘密

> 原文：<https://developers.redhat.com/blog/2020/02/14/using-secrets-in-apache-kafka-connect-configuration>

Kafka Connect 是一个集成框架，是 Apache Kafka 项目的一部分。在 Kubernetes 和[红帽 OpenShift](http://developers.redhat.com/openshift/) 上，你可以使用 [Strimzi](https://strimzi.io/) 和[红帽 AMQ 流](https://www.redhat.com/en/products/integration)操作符部署 Kafka Connect。Kafka Connect 允许用户运行接收器和源连接器。源连接器用于将数据从外部系统加载到 Kafka。Sink 连接器的工作方式正好相反，它允许您将数据从 Kafka 加载到另一个外部系统中。在大多数情况下，连接器在连接到其他系统时需要进行身份验证，因此您需要提供凭据作为连接器配置的一部分。本文向您展示了如何使用 Kubernetes secrets 来存储凭证，然后在连接器的配置中使用它们。

在本文中，我将使用一个 S3 源连接器，它是 Apache Camel Kafka 连接器之一。要了解更多关于 Apache Camel Kafka 连接器的信息，可以从[这篇博文](https://camel.apache.org/blog/Camel-Kafka-connector-intro/)开始。这个连接器只是作为如何配置连接器来访问机密的一个例子。您可以对任何连接器使用相同的过程，因为对连接器本身没有任何特殊要求。我们将使用 S3 连接器连接到亚马逊 AWS S3 存储，并将文件从 S3 存储桶加载到 Apache Kafka 主题中。为了连接到 S3 存储，我们需要指定 AWS 凭证:访问密钥和秘密密钥。所以，让我们从准备凭据的秘密开始。

## 用凭证创建一个秘密

首先，我们将创建一个名为`aws-credentials.properties`的简单属性文件，它应该如下所示:

```
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

您在这个属性文件中使用的凭证需要能够访问我们将要读取的 S3 存储桶。一旦我们准备好了带有凭证的属性文件，我们就必须从这个文件中创建秘密。您可以使用以下命令来完成此操作:

```
$ kubectl create secret generic aws-credentials --from-file=./aws-credentials.properties
```

## 用连接器构建新的容器映像

接下来，我们需要为连接器准备一个新的 Docker 映像。使用 Strimzi 时，用于添加连接器的`Dockerfile`应该如下所示:

```
FROM strimzi/kafka:0.16.1-kafka-2.4.0
USER root:root
COPY ./my-plugins/ /opt/kafka/plugins/
USER 1001
```

使用 AMQ 流时，应该是这样的:

```
FROM registry.redhat.io/amq7/amq-streams-kafka-23:1.3.0
USER root:root
COPY ./my-plugins/ /opt/kafka/plugins/
USER jboss:jboss
```

使用`Dockerfile`用您需要的连接器构建一个容器映像，并将它们推入您的注册表。如果你没有自己的私人注册中心，你可以使用公共注册中心，如[码头](https://quay.io/)或[码头枢纽](https://hub.docker.com/)。

## 部署 Apache Kafka Connect

一旦有了容器映像，我们就可以部署 Apache Kafka Connect 了。您可以通过创建以下自定义资源来实现这一点:

```
apiVersion: kafka.strimzi.io/v1alpha1
kind: KafkaConnect
metadata:
  name: my-connect-cluster
spec:
  image: docker.io/scholzj/kafka:camel-kafka-2.4.0
  replicas: 3
  bootstrapServers: my-cluster-kafka-bootstrap:9092
  externalConfiguration:
    volumes:
      - name: aws-credentials
        secret:
          secretName: aws-credentials
  config:
    config.providers: file
    config.providers.file.class: org.apache.kafka.common.config.provider.FileConfigProvider
    key.converter: org.apache.kafka.connect.json.JsonConverter
    value.converter: org.apache.kafka.connect.json.JsonConverter
    key.converter.schemas.enable: false
    value.converter.schemas.enable: false
```

让我们更详细地看看定制资源的几个部分。首先，注意`image`字段，它告诉部署 Apache Kafka Connect 的操作员使用添加了连接器的正确映像。在我的例子中，我将前一部分构建的容器映像作为`scholzj/kafka:camel-kafka-2.4.0`推送到 Docker Hub，所以我的配置如下所示:

```
image: docker.io/scholzj/kafka:camel-kafka-2.4.0
```

接下来，注意`externalConfiguration`部分:

```
externalConfiguration:
  volumes:
    - name: aws-credentials
      secret:
        secretName: aws-credentials
```

在本节中，我们指导操作员将我们在本文开始时创建的 Kubernetes secret `aws-credentials`挂载到 Apache Kafka Connect pods 中。这里列出的秘密将被安装在路径`/opt/kafka/external-configuration/<secretName>`中，其中`<secretName>`是秘密的名称。

最后，在配置部分，我们在 Apache Kafka Connect 中启用`FileConfigProvider`作为配置提供者:

```
config:
  config.providers: file
  config.providers.file.class: org.apache.kafka.common.config.provider.FileConfigProvider
```

配置提供程序是从另一个源加载配置值的一种方式，而不是直接在配置中指定它们。在这种情况下，我们创建名为 f `file`的配置提供者，它将使用`FileConfigProvider`类。这个配置提供程序是 Apache Kafka 的一部分。`FileConfigProvider`可以读取属性文件并从中提取值，我们将使用它来加载我们的亚马逊 AWS 帐户的 API 键。

## 使用 Apache Kafka Connect REST API 创建连接器

通常，我们必须等待一两分钟，让 Apache Kafka Connect 部署就绪。一旦*准备就绪，我们就可以创建连接器实例了。在旧版本的 Strimzi 和 Red Hat AMQ 流中，您必须使用 REST API 来完成这项工作。我们可以通过发布以下 JSON 来创建连接器:*

```
{
 "name": "s3-connector",
 "config": {
   "connector.class": "org.apache.camel.kafkaconnector.CamelSourceConnector",
   "tasks.max": "1",
   "camel.source.kafka.topic": "s3-topic",
   "camel.source.maxPollDuration": "10000",
   "camel.source.url": "aws-s3://camel-connector-test?autocloseBody=false",
   "key.converter": "org.apache.kafka.connect.storage.StringConverter",
   "value.converter": "org.apache.camel.kafkaconnector.converters.S3ObjectConverter",
   "camel.component.aws-s3.configuration.access-key": "${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_access_key_id}",
   "camel.component.aws-s3.configuration.secret-key": "${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_secret_access_key}",
   "camel.component.aws-s3.configuration.region": "US_EAST_1"
   }
}
```

连接器配置包含字段`camel.component.aws-s3.configuration.access-key`和`camel.component.aws-s3.configuration.secret-key`中的 AWS API 键。我们没有直接使用这些值，而是引用文件配置提供者从我们的`aws-credentials.properties`文件中加载字段`aws_access_key_id`和`aws_secret_access_key`。

注意我们如何引用配置提供程序，告诉它应该使用的文件的路径，并包含要提取的密钥的名称:

```
"camel.component.aws-s3.configuration.access-key": "${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_access_key_id}"
```

并且:

```
"camel.component.aws-s3.configuration.secret-key": "${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_secret_access_key}"
```

您可以将结果`POST`到 Apache Kafka Connect REST API，例如，使用`curl`:

```
$ curl -X POST -H "Content-Type: application/json" -d connector-config.json http://my-connect-cluster-connect-api:8083/connectors
```

使用配置提供程序的一个优点是，即使您稍后获得连接器配置，它仍将包含配置提供程序，而不是您想要保密的值:

```
$ curl http://my-connect-cluster-connect-api:8083/connectors/s3-connector
{
  "name": "s3-connector",
  "config": {
    "connector.class": "org.apache.camel.kafkaconnector.CamelSourceConnector",
    "camel.source.maxPollDuration": "10000",
    "camel.source.url": "aws-s3://camel-connector-test?autocloseBody=false",
    "camel.component.aws-s3.configuration.region": "US_EAST_1",
    "camel.component.aws-s3.configuration.secret-key": "${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_secret_access_key}",
    "tasks.max": "1",
    "name": "s3-connector",
    "value.converter": "org.apache.camel.kafkaconnector.converters.S3ObjectConverter",
    "camel.component.aws-s3.configuration.access-key": "${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_access_key_id}",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "camel.source.kafka.topic": "s3-topic"
  },
  "tasks": [
    {
      "connector": "s3-connector",
      "task": 0
    }
  ],
  "type": "source"
}
```

## 使用 Strimzi 连接器运算符创建连接器

当使用 Strimzi 0.16.0 或更新版本时，我们还可以使用新的连接器操作符。它允许我们使用以下定制资源 YAML 创建连接器(您也可以直接在`KafkaConnector`定制资源中使用配置提供者):

```
apiVersion: kafka.strimzi.io/v1alpha1
kind: KafkaConnector
metadata:
  name: s3-connector
  labels:
    strimzi.io/cluster: my-connect-cluster
spec:
  class: org.apache.camel.kafkaconnector.CamelSourceConnector
  tasksMax: 1
  config:
    key.converter: org.apache.kafka.connect.storage.StringConverter
    value.converter: org.apache.camel.kafkaconnector.converters.S3ObjectConverter
    camel.source.kafka.topic: s3-topic
    camel.source.url: aws-s3://camel-connector-test?autocloseBody=false
    camel.source.maxPollDuration: 10000
    camel.component.aws-s3.configuration.access-key: ${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_access_key_id}
    camel.component.aws-s3.configuration.secret-key: ${file:/opt/kafka/external-configuration/aws-credentials/aws-credentials.properties:aws_secret_access_key}
    camel.component.aws-s3.configuration.region: US_EAST_1
```

## 结论

Kubernetes secrets 的安全性有其局限性，任何可以进入容器的用户都可以读取挂载的机密。这个过程至少防止了机密信息，比如凭证或 API 密钥，通过 REST API 或在`KafkaConnector`定制资源中暴露出来。

*Last updated: June 29, 2020*