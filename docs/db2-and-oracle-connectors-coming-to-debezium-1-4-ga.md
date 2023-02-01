# Db2 和 Oracle 连接器即将进入 Debezium 1.4 GA

> 原文：<https://developers.redhat.com/blog/2021/03/25/db2-and-oracle-connectors-coming-to-debezium-1-4-ga>

这篇文章概述了新的 Red Hat 集成 [Debezium 连接器](/topics/event-driven/connectors)以及 Debezium 1.4 的正式发布(GA)中包含的特性。开发人员现在有了两种选择，可以将数据从他们的数据存储区流式传输到 [Apache Kafka](/topics/kafka-kubernetes) ，并支持处理数据模式的集成。

Db2 连接器的 GA 允许开发人员从 Db2 传输数据。现在处于技术预览版的 Oracle connector 提供了一种从最流行的数据库之一捕获变化的简单方法。最后，开发人员可以通过与 Red Hat Integration 的服务注册中心的完全支持的集成来委托 Debezium 模式。

## 什么是 Debezium？

Debezium 是一组分布式服务，它捕获行级数据库的变化，以便应用程序可以查看和响应这些变化。Debezium 连接器记录由[红帽 AMQ 流](https://developers.redhat.com/blog/2018/10/29/how-to-run-kafka-on-openshift-the-enterprise-kubernetes-with-amq-streams/)管理的 Kafka 集群的所有事件。应用程序使用 [AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet)来消费变更事件。

Debezium 使用 [Apache Kafka Connect](/blog/2020/02/14/using-secrets-in-apache-kafka-connect-configuration/) 框架，该框架将 Debezium 的连接器转换为 Kafka 连接器源连接器。可以使用 AMQ 流提供的 Kafka Connect Kubernetes 自定义资源来部署和管理它们。

最新的 1.4 版本引入了新的连接器，改进了其他连接器，并增加了有效处理模式的特性。

## Db2 的 Debezium 连接器正式上市

Debezium 对 SQL Server 的实现启发了它的 Db2 连接器。SQL Server 连接器基于用于 Db2 中 SQL 复制的[抽象语法符号(ASN)捕获和应用代理](https://www.ibm.com/support/pages/q-replication-and-sql-replication-product-documentation-pdf-format-version-101-linux-unix-and-windows)。Db2 连接器代理为处于捕获模式的表生成变更数据。它们还监视表并存储表更新的更改事件。然后，Debezium 连接器使用一个 SQL 接口来查询变更事件的变更数据表。

在技术预览版中[可供开发人员尝试并提供反馈，以及在 Red Hat 团队的广泛测试之后，Db2 连接器现已正式发布。Db2 连接器为 Db2 for Linux 用户提供了一种受支持的机制来传输他们的数据库。您可以在](https://developers.redhat.com/blog/2020/11/05/capture-ibm-db2-data-changes-with-debezium-db2-connector/)[文档](https://access.redhat.com/documentation/en-us/red_hat_integration/2021.q1/html/debezium_user_guide/debezium-connector-for-db2)中了解更多关于连接器及其配置的信息。

## 开发人员预览版中用于 Oracle 数据库的 Debezium 连接器

请求最多的连接器插件之一是 Red Hat 集成。现在，您可以使用 Debezium connector for Oracle 从 Oracle 数据库中流式传输数据，现在是在开发人员预览版中。

Oracle 数据库是许多组织架构的重要组成部分，因为开发人员创建了使用数据库作为其应用程序核心的完整系统。存储在 Oracle 数据库中的数据对于这些组织来说仍然至关重要，并且在新应用展开的同时能够访问这些数据对于成功迁移到现代系统架构至关重要。

用于 Oracle 数据库的 Debezium 连接器可以监视和记录 Oracle server 版本 12R2 和更高版本的数据库中的行级更改。Debezium 使用作为 Oracle 数据库实用程序一部分的 Oracle 本地 LogMiner 数据库包。LogMiner 提供了一种查询在线和归档重做日志文件的方法。

借助 Debezium connector for Oracle，开发人员可以将数据库中的数据更改直接传输到 [AMQ 流 Apache Kafka 集群](https://developers.redhat.com/topics/kafka-kubernetes)。流媒体活动允许任何人使用现代混合云技术充分利用他们的数据。实现[“为你的数据解放”](https://twitter.com/gunnarmorling/status/1123191912800845825)

## 与服务注册中心集成

Red Hat Integration[service registry](https://developers.redhat.com/blog/2020/12/09/new-features-and-storage-options-in-red-hat-integration-service-registry-1-1-ga/)是标准事件模式和 API 设计的数据存储。作为开发人员，您可以使用它将数据结构从应用程序中分离出来。您还可以使用 REST 接口来共享和管理您的数据结构。Red Hat 的服务注册中心建立在一个开源社区项目 [Apicurio Registry](https://www.apicur.io/registry/docs/apicurio-registry/1.3.3.Final/getting-started/assembly-intro-to-the-registry.html) 之上。

Debezium 使用 JSON 转换器将记录键和值序列化到 JSON 文档中。默认情况下，JSON 转换器包含一个记录的消息模式，所以每个记录都很长。另一种选择是使用其他序列化格式，如 [Apache Avro](http://avro.apache.org/) 或 [Google Protocol Buffers](https://developers.google.com/protocol-buffers) ，来序列化和反序列化每个记录的键和值。如果要使用这些格式中的任何一种进行序列化，Service Registry 会管理数据格式的消息模式和版本。您可以在 Debezium 连接器配置中指定所需的转换器。然后，转换器将 Kafka Connect 模式映射到该数据格式的模式。

以前作为技术预览版提供，现在完全支持 Debezium 和 Service Registry 之间的集成。你可以在我的[示例视频](https://youtu.be/NtGF-kwvcFI)中查看它是如何工作的。

## 开始阅读 Debezium 和 Kafka

您可以从 [Red Hat 开发者门户](https://developers.redhat.com/products)下载 Red Hat Integration Debezium 连接器。您还可以查看 [Gunnar Morling 在](https://developers.redhat.com/videos/youtube/QYbXDp4Vu-8/) [DevNation Tech Talks](https://developers.redhat.com/devnation/the-show) 系列中关于 Debezium 和 Kafka(2019 年 2 月)的网络研讨会，或者他在 QCon(2020 年 1 月)上的 [Kafka 和 Debezium 演示](https://www.infoq.com/presentations/data-streaming-kafka-debezium/)。最后，您可以在 Red Hat Developer 上了解有关您的活动可用的各种[连接选项](https://developers.redhat.com/topics/event-driven/connectors)的更多信息。

*Last updated: March 24, 2021*