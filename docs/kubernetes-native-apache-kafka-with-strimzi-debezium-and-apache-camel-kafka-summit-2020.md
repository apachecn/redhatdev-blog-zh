# kubernetes-土著阿帕奇卡夫卡与 Strimzi，Debezium，和阿帕奇骆驼(卡夫卡峰会 2020)

> 原文：<https://developers.redhat.com/blog/2020/08/21/kubernetes-native-apache-kafka-with-strimzi-debezium-and-apache-camel-kafka-summit-2020>

Apache Kafka 已经成为构建实时数据管道的领先平台。今天，Kafka 被大量用于开发[事件驱动的应用](https://developers.redhat.com/topics/event-driven/)，它让服务通过事件相互通信。使用 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 处理这种类型的工作负载需要添加专门的组件，如 Kubernetes 操作器和连接器，以将您的其余系统和应用程序连接到 Kafka 生态系统。

在本文中，我们将了解开源项目 Strimzi、Debezium 和 Apache Camel 如何与 Kafka 集成，以加速 Kubernetes 的关键领域——原生开发。

**注**:红帽正在赞助 2020 年 8 月 24-25 日的卡夫卡峰会 2020 虚拟会议。详见本文末尾。

## 卡夫卡与斯特里兹谈库伯内特斯

[Strimzi](https://strimzi.io/) 是一个开源项目，是[云本地计算基金会](https://www.cncf.io/) (CNCF)的一部分，它使得将 Apache Kafka 工作负载迁移到云变得更加容易。Strimzi 依赖于 Kubernetes 提供的抽象层和 [Kubernetes 操作符](https://developers.redhat.com/topics/kubernetes/operators/)模式。它的主要工作是在 Kubernetes 上运行 Apache Kafka，同时为 Kafka、Zookeeper 和 Strimzi 生态系统中的其他组件提供容器图像。

Strimzi 用 Kafka 相关的定制资源定义(CRD)扩展了 Kubernetes API。主要的卡夫卡 CRD 描述了要部署的卡夫卡集群，以及需要的动物园管理员合奏。但是 Strimzi 不仅仅是为了经纪人；您还可以使用它来创建和配置主题，并创建访问这些主题的用户。Strimzi 还支持使用 [Kafka MirrorMaker 2.0](https://cwiki.apache.org/confluence/display/KAFKA/KIP-382%3A+MirrorMaker+2.0) 自定义资源在集群之间镜像数据的配置，以及为 HTTP 客户端部署和管理 [Strimzi Kafka Bridge](https://strimzi.io/docs/bridge/latest/) 。

****了解更多**:**[strim zi 简介:库伯内特斯上的阿帕奇卡夫卡(KubeCon Europe 2020)](https://developers.redhat.com/blog/2020/08/14/introduction-to-strimzi-apache-kafka-on-kubernetes-kubecon-europe-2020/) 。

## 用 Debezium 改变数据捕获

[Debezium](https://debezium.io/) 是一组分布式服务，它捕获数据库中的行级变化，以便您的应用程序可以看到并响应这些变化。Debezium 在一个事务日志中记录提交给每个数据库表的所有行级更改。应用程序只需读取它们感兴趣的事务日志，并按照事件发生的顺序查看所有事件。Debezium 持久而快速，因此应用程序可以快速响应，即使出现问题也不会错过任何事件。

Debezium 提供了用于监控以下数据库的连接器:

*   MySQL 连接器
*   PostgreSQL 连接器
*   MongoDB 连接器
*   SQL Server 连接器

Debezium 连接器将所有事件记录到一个[红帽 AMQ 流](https://developers.redhat.com/blog/2018/10/29/how-to-run-kafka-on-openshift-the-enterprise-kubernetes-with-amq-streams/) Kafka 集群。然后，应用程序通过 AMQ 流使用这些事件。Debezium 使用 [Apache Kafka Connect](https://developers.redhat.com/blog/2020/02/14/using-secrets-in-apache-kafka-connect-configuration/) 框架，该框架将 Debezium 的所有连接器都变成 Kafka 连接器源连接器。因此，可以使用 AMQ 流的 Kafka Connect 自定义 Kubernetes 资源来部署和管理它们。

**了解更多信息** : [使用 Debezium Apache Kafka 连接器捕获数据库变更](https://developers.redhat.com/blog/2020/04/14/capture-database-changes-with-debezium-apache-kafka-connectors/)。

## 与 Apache Camel Kafka Connect 的 Kafka 连接

Apache Camel 社区已经构建了 Apache 基金会生态系统中最繁忙的开源集成框架之一。Camel 让您快速轻松地集成数据消费者和生产者系统。它还实现了最常用的[企业集成模式](https://www.enterpriseintegrationpatterns.com/)，并在流行的接口和协议出现时将其合并。

[Camel Kafka Connector](https://camel.apache.org/camel-kafka-connector/latest/index.html) 子项目着重于使用 Camel 组件作为 [Kafka Connect 连接器](https://developers.redhat.com/blog/2020/05/19/extending-kafka-connectivity-with-apache-camel-kafka-connectors)。为此，开发团队在 Camel 和 Kafka 框架之间构建了一个微小的层，它允许您轻松地将每个 Camel 组件用作 Kafka 生态系统中的 Kafka 连接器。超过 340 个 Camel Kafka 连接器支持从 AWS S3 到 Telegram 和 Slack 的各种集成。所有这些连接器都可以与 Kafka 一起使用，而不用抛出一行代码。

**了解更多信息** : [使用 Apache Camel Kafka 连接器扩展 Kafka 连接](https://developers.redhat.com/blog/2020/05/19/extending-kafka-connectivity-with-apache-camel-kafka-connectors/)。

## 结论

每天都有新的组织采用 Apache Kafka 作为活动骨干。像 Apache Camel 这样的社区正在研究如何加快集成等关键领域的开发。Debezium 社区提供了专门的连接器，简化了将数据库生成的事件从微服务或遗留应用程序集成到现代的事件驱动架构中。最后，像 Strimzi 这样的 CNCF 项目可以更容易地获得 Kubernetes 的优势，并以云原生方式部署 Apache Kafka 工作负载。

对于那些想要一个有企业支持的开源开发模型的人来说， [Red Hat Integration](https://www.redhat.com/en/products/integration) 让你在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) (企业 Kubernetes)上部署你的基于 Kafka 的事件驱动架构。[红帽 AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet)、Debezium 和 Apache Camel Kafka Connect 连接器都可以通过红帽集成订阅获得。

## 卡夫卡峰会 2020

如果你想了解更多关于在 Kubernetes 上运行 Apache Kafka 的信息，红帽正在赞助 2020 年 8 月 24 日至 25 日举行的 [Kafka Summit 2020](https://kafka-summit.org/) 虚拟会议。您可以参加以下任何一个会议(请注意，您必须注册才能访问这些链接):

*   **2020 年 8 月 25 日星期二上午 10:00 PDT**:[用 Debezium 和 Kafka 流改变数据捕获管道](https://kafkasummit.io/session-virtual/?v26dd132ae80017cdaf764437c30ebe6f10c1b1eeaab01165e44366654b368dfaeab6baf7e386a642ecb238989334530e=29357BABE872174F33FC8355B5D7F6CBA10F9416968BEE4F161E83F2847328787AEE868E65ECEAE3D43713051B9D2B3C)Debezium 项目负责人 Gunnar Morling。
*   **2020 年 8 月 25 日星期二上午 10:30 PDT**:[Camel Kafka Connectors:将 Kafka 调成(几乎)可以和一切东西“说话](https://kafkasummit.io/session-virtual/?v26dd132ae80017cdaf764437c30ebe6f10c1b1eeaab01165e44366654b368dfaeab6baf7e386a642ecb238989334530e=21BEFF118082C7BD226CEA8B405E4A27DAC3D4A0A525C2F58BD5774B496BD7899376E76179D6BCF1CA17BC373DD0BE4C)Apache Camel 工程师 Andrea Cosentino 和 Andrea Tarocchi，

如果您想跟踪对话并与演示者交流，我将在以下时间主持与工程主管的小组讨论:

*   太平洋时间 2020 年 8 月 24 日星期一上午 10:00-11:00
*   太平洋时间 2020 年 8 月 24 日星期一下午 1:00-2:00
*   太平洋时间 2020 年 8 月 25 日星期二上午 11:00-下午 12:00
*   太平洋时间 2020 年 8 月 25 日星期二下午 1:00-2:00

最后，在整个活动期间，我们将有更多的红帽子在[赞助商展位](https://kafkasummit.io/virtual-exhibitor/?v0326b739525aaf6a5900c153ea6485e67109462e8db159b156161fc07c7e3d8016769932b4c0398e64b5ea52edb3d1c5=56CC3380CBA86BDA1DB77B0F6C902F3EE409DC5CA6F71077702CE1C9452986BBE14B10CA443311D61A730309F78FE22B)解决您关于在 Kubernetes 上运行 Kafka 的问题。

*Last updated: August 20, 2020*