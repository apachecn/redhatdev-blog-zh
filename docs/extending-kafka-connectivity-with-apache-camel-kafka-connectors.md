# 使用 Apache Camel Kafka 连接器扩展 Kafka 连接

> 原文：<https://developers.redhat.com/blog/2020/05/19/extending-kafka-connectivity-with-apache-camel-kafka-connectors>

Apache Kafka 是现代应用程序开发中使用最多的软件之一，因为它具有分布式、高吞吐量和水平可伸缩性的特点。每天都有越来越多的组织采用 Kafka 作为他们的[事件驱动架构](https://developers.redhat.com/topics/event-driven/)的中心事件总线。因此，越来越多的数据通过集群流动，使得连接性需求对于任何积压都变得更加重要。出于这个原因， [Apache Camel](https://camel.apache.org/) 社区发布了第一个迭代 [Kafka Connect](https://kafka.apache.org/documentation.html#connect) 连接器，目的是减轻开发团队的负担。

## 什么是阿帕奇骆驼？

Camel 社区已经在 [Apache 基金会](https://www.apache.org/)中构建了[最繁忙的](https://camel.apache.org/blog/ASF-Report-2019/)开源集成框架之一。Camel 框架让您能够快速轻松地集成数据消费者和生产者系统。它实现了最常用的[企业集成模式](https://www.enterpriseintegrationpatterns.com/) (EIPs)，加上到处都在使用的接口和协议。让所有东西都处于相同的组件基础配置下，可以让您创建所需的构建块来解决几乎所有的集成需求。

## 什么是骆驼卡夫卡连接器项目？

在成熟了近十年之后，Camel 社区推出了几个[子项目](https://camel.apache.org/projects/)，以促进运行时支持和容器就绪等领域的创新。具体来说，Camel Kafka 连接器子项目侧重于将 Camel 组件用作 Kafka Connect 连接器。考虑到这一点，他们在 Camel 和 Kafka 框架之间构建了一个微小的层，以便轻松地将每个 Camel 组件用作 Kafka 连接器，可以毫不费力地在 Kafka 生态系统中使用。

这个项目的[第一次发布](https://camel.apache.org/blog/Camel-Kafka-connector-release-0.1.0/)允许社区尝试和分享关于自动生成的连接器的反馈。尽管是这个子项目的第一个版本，Camel 的大部分底层组件都经过了实战测试，并在世界各地的生产场景中使用。

## Camel Kafka 连接器入门

有超过 [340+ Camel Kafka 连接器](https://camel.apache.org/camel-kafka-connector/latest/connectors.html)可供入门，从 AWS S3 集成到 Telegram 或 Slack。所以应该很容易找到一个用例来实现一个小项目。

为了简化 Kafka Connect 集群部署，您可以使用来自 [Strimzi](https://landscape.cncf.io/selected=strimzi) 项目的 [Kubernetes 操作符](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)在您的笔记本电脑上运行 Minikube 或 Kind。然后，从 [Maven central repository](https://repo1.maven.org/maven2/org/apache/camel/kafkaconnector/) 中搜索并下载连接器包 zip 版本。zip 文件应该包含插件路径中运行连接器所需的所有库 jar。接下来，在 Strimzi 基础映像的顶部创建一个包含插件的容器映像，这样您就可以将该映像用作 Kafka Connect 基础。最后，使用 Kafka Connect REST API 或新的 [Strimzi Kafka 连接器](https://strimzi.io/docs/latest/#proc-deploying-kafkaconnector-str)自定义资源定义(CRD)来配置连接器。

你也可以在本地的笔记本电脑上尝试连接器，而不需要 Kubernetes。您只需要一个本地运行的 Kafka 实例并遵循这些说明。

想看看跑步时的样子吗？观看我关于将 Slack 整合到 Kafka 的视频:

[https://www.youtube.com/embed/tkkPO7hu848?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/tkkPO7hu848?autoplay=0&start=0&rel=0)

## 摘要

阿帕奇卡夫卡每天都在新组织里当事件骨干。像 Apache Camel 这样的社区正在研究如何加速现代应用程序关键领域的开发，比如集成。Apache 基金会的 Camel Kafka Connect 项目使他们的大量连接器能够与 Kafka Connect 进行本地交互，这样开发人员就可以在他们首选的系统上发送和接收来自 Kafka 的数据。

*精选图片来源:《让卡夫卡为骑骆驼做好准备》是由 [g p](https://www.flickr.com/photos/malaqa/) 创作的[卡夫卡](https://flic.kr/p/5Hntiw)和由 [Ziad Fhema](https://www.flickr.com/photos/162485676@N06/) 创作的[骆驼](https://flic.kr/p/2hNq3ba)的衍生作品，在 2.0CC 下使用。*

*Last updated: January 18, 2022*