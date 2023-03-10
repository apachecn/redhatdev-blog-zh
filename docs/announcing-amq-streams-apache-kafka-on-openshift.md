# 宣布 AMQ 流:在 OpenShift 上的阿帕奇卡夫卡

> 原文：<https://developers.redhat.com/blog/2018/05/07/announcing-amq-streams-apache-kafka-on-openshift>

大家好，

我们很高兴地宣布红帽 AMQ 流的开发者预览，这是红帽 AMQ 的新成员，专注于在 OpenShift 上运行 Apache Kafka。

Apache Kafka 是一个领先的实时分布式消息平台，用于构建数据管道和流应用程序。

使用 Kafka，应用程序可以:

*   发布和订阅记录流。
*   存储记录流。
*   在记录出现时进行处理。

Kafka 让这一切成为可能，同时具有快速、横向可伸缩性和容错性。这使得 Kafka 适用于大范围的用例，包括网站活动跟踪、指标和日志聚合、流处理、事件源和物联网遥测。即将推出的 AMQ 流产品将为 Red Hat 客户提供在 Red Hat Enterprise Linux 和 Red Hat OpenShift 容器平台上运行 Apache Kafka 的支持产品。

随着越来越多的应用转向 Kubernetes 和 OpenShift，能够在同一平台上运行通信基础设施变得越来越重要。OpenShift 作为一个高度可扩展的平台，非常适合 Kafka 等消息传递技术。但是对于 AMQ 流，我们的目标不仅仅是*在 OpenShift 上运行* Apache Kafka，而是 AMQ 流让运行和管理 Apache Kafka 成为“OpenShift 原生的”

在 OpenShift 这样的弹性平台上整合 Kafka 的巨大可伸缩性需要解决许多技术挑战:

*   Kafka 代理本质上是有状态的，因为每个代理都有自己的身份和数据日志，在重启时必须保留。
*   更新和扩展 Kafka 集群需要仔细的编排，以确保消息客户端不受影响，并且不会丢失任何记录。
*   按照设计，Kafka 客户端连接到集群中的所有代理。这是 Kafka 水平扩展和高可用性的一部分，但当在 OpenShift 上运行时，这意味着 Kafka 集群不能像其他服务一样简单地放在负载平衡服务之后。相反，服务必须与集群扩展并行协调。
*   运行 Kafka 还需要运行 Zookeeper 集群，这与运行 Kafka 集群有许多相同的挑战。

AMQ 流使用运算符概念简化了 Apache Kafka 在 OpenShift 上的部署、配置、管理和使用，从而实现了 OpenShift 的固有优势，如弹性伸缩。操作者是特定于应用程序的控制器，它扩展了 Kubernetes APIs，并将它们与特定于领域的知识结合起来，使运行和管理复杂的应用程序变得容易。习惯于 OpenShift 的声明式资源供应方法的开发人员和管理员现在可以在使用 Kafka、Kafka Connect 和 Kafka 主题时享受同样的好处。

AMQ 溪流让人很容易:

*   只需点击一个按钮或一个`oc create`命令，就可以部署一个完整的 Kafka 集群。
*   将 Kafka 主题部署在使用它的微服务旁边。
*   扩大该主题的范围。
*   根据负载来上下缩放 Kafka 集群。

![Strimzi Logo](img/e0d09b20edc1000481a23407df92159d.png)

AMQ 流针对在 OpenShift 上运行进行了优化(相对于普通的 Kubernetes)。它不仅受益于 Red Hat 多年的经验和从开发和运行 OpenShift 中获得的深入知识，而且还特别支持使用用户自己的 Kafka Connect 插件构建 Kafka Connect 集群。此外，OpenShift 特定的功能和 Red Hat 产品集成也在预期之中，总体目标是在 OpenShift fabric up 的全面支持下实现无缝体验。

不出所料，因为 Red hat 是世界领先的企业开源技术提供商，AMQ 流是完全开源的，并且基于 [Strimzi 项目](http://strimzi.io)。

开发者预览版将于本周向感兴趣的用户开放，它为在 OpenShift 上运行 Kafka 提供了基础。感兴趣的客户和其他相关方被邀请[尝试一下，给我们他们的反馈，如果需要，在开源 Strimzi 项目上进行合作，以帮助塑造 OpenShift 上 AMQ 流的未来方向。](https://docs.google.com/forms/d/e/1FAIpQLSfWfg1iW0OSSAhhtqjy-qyJYIHJowqESsSrIySw3Uu8rFqn5g/viewform?usp=sf_link)

如果您有幸参加本周在三藩市举行的红帽峰会，那么您可以在以下会议中听到更多关于 AMQ 流(以及更广泛的红帽 AMQ 产品)的信息:

*   5 月 8 日星期二下午 1:00-3:00，Moscone South 156
    Marius Bogoevici，Paolo Patierno，Gunnar Morling [ [L1099](https://agenda.summit.redhat.com/SessionDetail.aspx?id=154665) ]使用 Kafka 在 OpenShift
    上运行数据流应用程序
*   红帽 AMQ 概述和路线图【2011 年 5 月 9 日星期三上午 11:45-下午 12:30，莫斯康西
    大卫·英厄姆，杰克·布里顿 [S2802](https://agenda.summit.redhat.com/SessionDetail.aspx?id=163898)
*   介绍 AMQ 流 Apache Kafka 的数据流【2014 年 5 月 10 日星期四上午 11:15 至下午 12:00，莫斯康西
    保罗·帕蒂尔诺，大卫·英厄姆 [S1775](https://agenda.summit.redhat.com/SessionDetail.aspx?id=154757)
*   红帽 AMQ 在线—消息即服务
    5 月 10 日星期四下午 1:45-3:45，莫斯康南 214
    Ulf Lilleengen，Paolo Patierno [ [W1098](https://agenda.summit.redhat.com/SessionDetail.aspx?id=154662) ]

我们预计将在今年晚些时候发布正式版时发布更多预览版。

请尝试一下，让我们知道你的想法。

*Last updated: November 15, 2018*