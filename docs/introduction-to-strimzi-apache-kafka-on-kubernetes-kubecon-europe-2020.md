# 《斯特里姆齐简介:库伯内特斯上的阿帕奇卡夫卡》(KubeCon Europe 2020)

> 原文：<https://developers.redhat.com/blog/2020/08/14/introduction-to-strimzi-apache-kafka-on-kubernetes-kubecon-europe-2020>

Apache Kafka 已经成为构建实时数据管道的领先平台。Kafka 最初是一个消息传递系统，主要用于发布/订阅模式，现在它已经成为一个数据- [流](https://developers.redhat.com/blog/category/stream-processing/)平台，用于实时处理数据。今天，Kafka 还被大量用于开发[事件驱动的应用](https://developers.redhat.com/topics/event-driven/)，使基础设施中的服务能够通过使用 Apache Kafka 作为主干的事件相互通信。与此同时，由于 Kubernetes ，云原生应用程序开发正在获得更多的关注。

由于该平台提供的抽象层，可以很容易地将您的应用程序从裸机上迁移到任何支持混合云场景的云提供商(AWS、Azure、GCP、IBM 等)。但是，如何将 Apache Kafka 工作负载转移到云上呢？有可能，但是不简单。您可以学习 Apache Kafka 处理集群的所有工具，以便将 Kafka 工作负载转移到 Kubernetes，或者您可以使用 [Strimzi](https://strimzi.io/) 利用您已经掌握的 Kubernetes 知识。

**注** : Strimzi 将出席 2020 年 8 月 17 日至 20 日举行的虚拟 KubeCon Europe 2020 大会。详见文末。

## 欢迎来到斯特里兹

Strimzi 是一个在 Apache License 2.0 下授权的开源项目，它是去年开始作为沙盒项目的[云本地计算基金会](https://www.cncf.io/) (CNCF)的一部分。它的主要工作是在 Kubernetes 上运行 Apache Kafka，同时为 Apache Kafka 本身、Zookeeper 和 Strimzi 生态系统中的其他组件提供容器映像。

利用 [Kubernetes Operator](https://developers.redhat.com/topics/kubernetes/operators/) 模式，它解决了从创建、管理和监控 Kafka 集群到管理所有相关实体(如主题和用户)的整个生命周期。您可以获得真正的 Kubernetes-native 体验，处理 Apache Kafka 生态系统中的所有组件。

## 扩展 Kubernetes:strim zi 定制资源定义

Strimzi 用新的 Kafka 相关的定制资源定义(CRD)扩展了 Kubernetes API。这意味着除了拥有常见的 Kubernetes——原生资源和对象，如`Pod`、`Deployment`等等，您还可以获得一堆描述 Kafka 相关组件的定制资源。主 CRD 是`Kafka`，它描述了要部署的 Kafka 集群:

*   您想要的副本(代理)数量。
*   相关的配置。
*   侦听器，用于使代理可以从 Kafka 运行的 Kubernetes 集群内部或外部访问。
*   还有很多。

它还描述了 Kafka 工作所需的 ZooKeeper ensemble 和 Kubernetes 操作员的配置；最后，由于有了这个资源，还可以为集群重新平衡操作部署巡航控制。

使用众所周知的`kubectl`工具，您可以获得在 Kubernetes 集群上运行的所有 Kafka 实例:

```
$ kubectl get kafka
NAME         DESIRED KAFKA REPLICAS   DESIRED ZK REPLICAS
my-cluster   3                        3
```

清单 Kubernetes 集群中的 Kafka 资源。

但 Strimzi 不仅仅是卡夫卡经纪人，它是关于它的整个生态系统。由于有了`KafkaTopic`和`KafkaUser`资源，您可以创建具有相关配置(分区、副本等)的主题。)和用户，使用相关的访问控制列表(ACL)来访问主题，而无需使用任何特定的 Kafka 工具。`KafkaConnect`和`KafkaConnector`资源允许您部署 Kafka Connect 并配置连接器，以便使用 Kafka 在不同系统之间移动数据(即，将数据从一个数据库迁移到另一个数据库)。

Strimzi 还支持在位于不同数据中心的两个不同集群之间镜像数据——多亏了 Mirror Maker——可以使用`KafkaMirrorMaker`和`KafkaMirrorMaker2`资源进行部署。HTTP 客户端可以使用 Strimzi 桥连接到您的 Kafka 集群，这要感谢`KafkaBridge`资源。

最后，由于随着时间的推移，集群可能会变得不平衡，一些代理处理的流量比其他代理多，因此可以使用`KafkaRebalance`资源来请求 Cruise Control 实例重新平衡集群，以满足 CPU、网络、内存利用率等方面的目标。

## Strimzi 是如何工作的？

在您的 Kubernetes 集群上安装 Strimzi 有不同的方法:使用每个版本提供的 YAML 文件，通过 [OperatorHub.io](https://operatorhub.io/) ，或者直接从官方网站应用单个 YAML 文件，就像我们在本例中将要做的那样。

键入以下命令以部署 Strimzi 集群操作符:

```
$ kubectl apply -f https://strimzi.io/install/latest?namespace=default
```

*清单 2:安装 Strimzi 操作符。*

此时，您需要的是一个 YAML 文件，其中包含一个描述您想要部署的 Kafka 集群的`Kafka`资源:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 2.5.0
    replicas: 3
    listeners:
      plain: {}
      tls: {}
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      log.message.format.version: "2.5"
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 100Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

清单 3:描述集群的 Kafka 资源。

清单 3 中的`Kafka`资源描述了一个简单的 Kafka 集群，其中有三个代理可以通过“普通的”(在端口 9092 上)和 TLS 加密的监听器以及特定的配置参数进行访问。它还使用持久存储。

当应用这个资源时，集群 Strimzi 集群操作符负责处理它，以便部署 Kafka 集群，启动 ZooKeeper 和 Kafka pods，最后启动 Strimzi 操作符来处理主题和用户。最后，您应该看到清单 3 中的所有 pods 都在运行，集群已经准备好供您的 Kafka 客户端用于交换消息:

```
NAME                                         READY   STATUS    RESTARTS   AGE
my-cluster-entity-operator-f977bf457-l2rjf   3/3     Running   0          51s
my-cluster-kafka-0                           2/2     Running   0          90s
my-cluster-kafka-1                           2/2     Running   0          90s
my-cluster-kafka-2                           2/2     Running   0          90s
my-cluster-zookeeper-0                       1/1     Running   0          2m58s
my-cluster-zookeeper-1                       1/1     Running   0          2m58s
my-cluster-zookeeper-2                       1/1     Running   0          2m58s
strimzi-cluster-operator-7d6cd6bdf7-8xxvx    1/1     Running   0          17m
```

*清单 4: Strimzi 操作符和 Kafka 集群相关的 pod 正在运行。*

几分钟后，一个 Apache Kafka 集群就在 Kubernetes 上启动并运行了。在那里，您可以通过更改副本的数量或更新集群配置来轻松扩展或缩减集群。Strimzi 集群操作器监视`Kafka`资源的变化，并将这些变化应用到正在运行的集群，启动新的代理或关闭现有的代理，更新它们的配置，并在需要时进行滚动更新。

这只是你旅程的开始。看看 Strimzi 的官方网站和博客帖子，了解更多关于其他资源的信息。

## strizzi 社区

作为 CNCF 的一部分，Strimzi 与这样一个生态系统中的其他项目很好地结合在一起。首先，它提供了用于部署 Strimzi 操作符的 [Helm](https://helm.sh/) 图表，以及配置由 [Prometheus](https://prometheus.io/) 服务器抓取并显示在 [Grafana](https://grafana.com/) 仪表板上的指标导出。还可以使用 [Jaeger](https://www.jaegertracing.io/) 和 [OpenTracing](https://opentracing.io/) 来启用*跟踪*，以便跟踪客户端和其他组件(如桥、Kafka Connect 和 Mirror Maker)之间通过 Apache Kafka 集群的消息流。

Strimzi 还很好地集成了用于描述客户端授权策略的[开放策略代理](https://www.openpolicyagent.org/) (OPA)项目，以便让它们生成和消费关于主题的消息。最后， [Kubernetes 事件驱动自动缩放(KEDA)](https://keda.sh/) 项目可用于使用基于 Apache Kafka 的缩放器自动缩放事件驱动的应用程序。

但是 Strimzi 生态系统不仅仅意味着与其他 CNCF 项目的整合。这也意味着与项目本身的工作人员的不断增长的社区进行互动。

参与社区并帮助项目获得牵引力:

*   下载并试用 Strimzi，部署您的 Apache Kafka 集群，并可能通过官方的 [GitHub](https://github.com/strimzi) 库发现 bug 或建议新功能。
*   如果你没有在代码上工作，改进[文档](https://strimzi.io/documentation/)。
*   在会议上传播消息。
*   关于 Strimzi 和你使用它的方式的博客。

## 结论

本文展示了将 Apache Kafka 工作负载迁移到云上并不困难。多亏了 Strimzi，只需几分钟就可以在 Kubernetes 上启动并运行 Kafka 集群，然后您就可以开始进行产品化调整了。令人敬畏的是，Strimzi 不仅仅是关于卡夫卡本身，而是它的整个生态系统。

如果你想了解更多，只需访问[官网](https://strimzi.io/)，加入 [the Slack channel](https://slack.cncf.io/) 的#strimzi room，或者关注[的 Twitter 账号](https://twitter.com/strimziio)。我们真的很高兴收到你的来信！

## KubeCon 欧洲 2020

Strimzi 将出席 2020 年 8 月 17 日至 20 日举行的虚拟 KubeCon Europe 2020 大会:

*   【2020 年 8 月 17 日 13:00 CEST : [“与维护者见面”环节](https://sched.co/djTb)。
*   【2020 年 8 月 19 日 13:00 CEST :参加我们的[介绍会](https://kccnceu20.sched.com/event/Zevl/introduction-to-strimzi-apache-kafka-on-kubernetes-jakub-scholz-paolo-patierno-red-hat?iframe=no&w=100%&sidebar=yes&bg=no)，它将解释 Strimzi 如何工作的基础知识，并展示如何轻松部署 Kafka 集群的演示。这个演示之后将会有一个现场问答环节&
*   【2020 年 8 月 19 日 14:00 CEST : [“与维护者见面”环节](https://sched.co/djUZ)。

在“会见维护者”会议中，您可以会见我们所有的维护者，与我们交谈，并就问题、错误或缺失的功能提出疑问。

*Last updated: August 13, 2020*