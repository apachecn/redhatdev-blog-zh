# 基于 HTTP 的 Kafka 消息与红帽 AMQ 流

> 原文：<https://developers.redhat.com/blog/2020/08/04/http-based-kafka-messaging-with-red-hat-amq-streams>

Apache Kafka 是一个坚如磐石、速度超快的事件流主干，不仅仅是针对[微服务](https://developers.redhat.com/topics/microservices)。它是许多用例的推动者，包括活动跟踪、日志聚合、流处理、变化数据捕获、[物联网](https://developers.redhat.com/blog/category/iot/) (IoT)遥测等。

[红帽 AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet)使得在[红帽 OpenShift](https://developers.redhat.com/products/openshift/overview) 上运行和管理 Kafka 变得简单。AMQ 溪流公司的上游项目 [Strimzi](https://strimzi.io/) ，为 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 做同样的事情。

在开发人员的笔记本电脑上设置 Kafka 集群既快速又简单，但是在某些环境中，客户端设置会比较困难。Kafka 使用一种基于 TCP/IP 的专有协议，并拥有可用于许多不同编程语言的[客户端](https://cwiki.apache.org/confluence/display/KAFKA/Clients)。然而，Kafka 的主要代码库中只有 JVM 客户端。

在许多情况下，这是困难的，不可能的，或者我们只是不想投入精力来手动安装和设置 Kafka 客户端。AMQ 流中隐藏的宝石对于那些想访问 Kafka 客户端，但不想设置客户端的开发人员来说是一个很大的帮助。在本文中，您将从 [Red Hat AMQ 流 Kafka 桥](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html/using_amq_streams_on_openshift/kafka-bridge-concepts-str)开始，这是一个使用 HTTP/1.1 生成和消费 Kafka 主题的 RESTful 接口。

**注**:Kafka HTTP 桥从  和  向前可用。

图 1 显示了典型的 Apache Kafka 消息系统中的 AMQ 流 Kafka 桥。

[![A diagram showing AMQ Streams Kafka Bridge in an Apache Kafka messaging system on Red Hat OpenShift.](img/e3eb3e468449e8801dc270fec6f6de6c.png "kafka-bridge-1-1024x683")](/sites/default/files/blog/2020/07/kafka-bridge-1-1024x683.png)Figure 1: AMQ Streams Kafka Bridge in a typical Apache Kafka messaging system.Figure 1: AMQ Streams Kafka Bridge in a typical Apache Kafka messaging system.">

## AMQ 溪流卡夫卡桥入门

要使用 AMQ 流，您需要一个 3.11 或更高版本的 OpenShift 集群，以及一个具有集群管理员角色的用户。

我在一台开发人员笔记本电脑上测试了本文的代码，该笔记本电脑在 OpenShift 4.3.1 上安装了(RHEL)7.6 和[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers)(CRC)1.9。我建议用至少 16GB 的内存和八个核心运行 CRC，但这取决于你。(只是不要太小气；否则，您可能会在启动 Kafka 集群时遇到问题。)

### 五分钟安装

首先，我们将在一个名为`kafka`的专用项目上安装 Kafka 自定义资源定义(CRD)和基于角色的访问控制(RBAC)。然后，我们将在项目中安装一个 Kafka 集群，我们将其命名为`my-kafka-cluster`。

就是这样！Kafka 集群已经启动并运行。

### 安装 AMQ 溪流卡夫卡桥

为 AMQ 流安装 Kafka HTTP 桥只需要一个 YAML 文件:

```
$ oc apply -f examples/kafka-bridge/kafka-bridge.yaml

```

一旦安装了文件，集群操作者将创建一个部署、一个服务和一个 pod。

### 暴露 OCP 外的桥梁

我们已经安装并配置了网桥，但是我们只能在集群内部访问它。使用以下命令在 OpenShift 外部公开它:

```
$ oc expose service my-bridge-bridge-service
```

网桥本身不提供任何安全性，但我们可以用其他方法来保护它，如网络策略、反向代理(OAuth)和传输层安全性(TLS)终止。如果我们想要一个功能更全面的解决方案，我们可以使用带有 的桥，该网关包括 TLS 认证和授权以及指标、速率限制和计费。

**注意**:当连接到 Kafka 集群时，Kafka HTTP 桥支持 TLS 或基于简单认证和安全层(SASL)的认证，以及 TLS 加密的连接。还可以安装许多桥，选择内部或外部实现，每一个都有不同的认证机制和不同的访问控制列表。

### 验证安装

让我们检查一下桥是否可用:

```
$ BRIDGE=$(oc get routes my-bridge-bridge-service -o=jsonpath='{.status.ingress[0].host}{"\n"}')
curl -v $BRIDGE/healthy

```

注意，桥将 REST API 公开为 OpenAPI 兼容的:

```
$ curl -X GET $BRIDGE/openapi
```

## 利用 AMQ 溪流卡夫卡大桥

此时，一切都准备好了使用 AMQ 流 Kafka 桥来生成和消费消息。我们将一起快速演示一遍。

### 生成和使用系统日志

日志摄取是 Kafka 的常见用例之一。我们将用我们的系统日志填充一个 Kafka 主题，但是它们可以来自任何支持 HTTP 的系统。同样，日志也可以被其他系统使用。

首先创建一个主题，并将其命名为`machine-log-topic`:

```
$ cat << EOF | oc create -f -
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: my-topic
  labels:
    strimzi.io/cluster: "my-cluster"
spec:
  partitions: 3
  replicas: 3
EOF

```

然后，使用`curl`和`jq`用数据填充主题:

```
$ journalctl --since "5 minutes ago"  -p "emerg".."err"  -o json-pretty | \
jq --slurp '{records:[.[]|{"key":.__CURSOR,value: .}]}' - | \
curl -X POST $BRIDGE/topics/machine-log-topic -H 'content-type: application/vnd.kafka.json.v2+json' -d @-

```

通常，内容类型是`application/vnd.kafka.json.v2+json`，但是对于二进制数据格式，它也可以是`application/vnd.kafka.binary.v2+json`。如果使用二进制数据格式，则需要 Base64 值。

### 消费邮件

现在我们有消息要消费。在我们从一个主题消费之前，我们必须将我们的消费者添加到一个消费者组中。然后，我们必须让消费者订阅主题。在本例中，我们将消费者`my-consumer`包括在消费者组`my-group`中:

```
$ CONS_URL=$(curl -s -X POST  $BRIDGE/consumers/my-group -H 'content-type: application/vnd.kafka.v2+json' \
-d '{
"name": "my-consumer",
"format": "json",
"auto.offset.reset": "earliest",
"enable.auto.commit": true
}' | \
jq .base_uri  | \
sed 's/\"//g')

```

接下来，我们订阅题目`my-topic`:

```
$ curl -v $CONS_URL/subscription -H 'content-type: application/vnd.kafka.v2+json'  -d '{"topics": ["my-topic"]}

```

现在我们准备消费:

```
$ curl -X GET $CONS_URL/records -H 'accept: application/vnd.kafka.json.v2+json' | jq

```

## 结论

在尖端的微服务架构中集成旧的但良好的服务或设备可能具有挑战性。但是，如果您可以在没有超高速消息传递的情况下生活(这些较老的服务提供这种功能)，Apache Kafka HTTP 桥允许这些服务——只需要一点点 HTTP/1.1——利用 Apache Kafka 的功能。

Apache Kafka HTTP 桥很容易使用它的 REST API 来设置和集成，并且它允许无限制地使用 HTTP 传输。在本文中，我向您展示了在 OCP 上部署 AMQ 流 Kafka 桥的快速安装过程，然后演示了使用 HTTP 上的日志数据的生产者-消费者消息传递场景。

*Last updated: October 28, 2020*