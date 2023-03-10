# 用 Debezium Apache Kafka 连接器捕获数据库变化

> 原文：<https://developers.redhat.com/blog/2020/04/14/capture-database-changes-with-debezium-apache-kafka-connectors>

变更数据捕获，简称 CDC，是一种为系统建立的软件设计模式，它监控和捕获数据的变更，以便其他软件能够响应这些变更。CDC 捕获对数据库表的行级更改，并将相应的更改事件传递给数据流总线。应用程序可以读取这些变更事件流，并按照事件发生的顺序访问这些变更事件。

因此，变更数据捕获有助于连接传统数据存储和新的云原生事件驱动架构。 同时， [Debezium](https://debezium.io/) 是一组分布式服务，它捕获数据库中行级的变化，以便应用程序可以看到并响应这些变化。这个来自 [Red Hat Integration](https://developers.redhat.com/integration/) 的通用(GA)版本包括以下用于 Apache Kafka 的 Debezium 连接器:MySQL 连接器、PostgreSQL 连接器、MongoDB 连接器和 SQL Server 连接器。

![Diagram showing where Debezium fits in Kafka infrastructure](img/a9dab6225b3c9919e7cac60d6b658273.png)

## 使用 Debezium 的 CDC 和事件驱动型微服务

Debezium 基于日志的变更数据捕获的优点是可以捕获数据库中注册的所有数据变更，提供了一个可靠的来源。与查询数据库或开销相比，没有延迟。Debezium 还为应用程序和数据模型提供了透明的机制，避免了污染当前系统设计的需要。与轮询相比，读取事务日志还提供了较低的开销，没有丢失任何事件的风险。

基于流行的 [Apache Kafka Connect API](https://kafka.apache.org/documentation.html#connect) ，Debezium Apache Kafka 连接器适合与[红帽 AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet) Kafka 集群一起部署。他们将所有事件记录到一个[红帽 AMQ 流](https://developers.redhat.com/topics/event-driven/) Kafka 集群，应用程序通过 AMQ 流消费这些事件。

Debezium 还允许捕获*事件，以及关于 *旧记录状态* 和其他元数据的信息，这些信息可以作为事件的一部分进行共享，以便进一步处理。Debezium change 事件结构包括关于表的键的信息，以及关于具有先前状态的值、当前更改和元数据信息的信息。这些事件可以序列化为熟悉的 JSON 或 Avro 格式，而对 CloudEvents 的支持将在未来的版本中推出。*

 *变更数据捕获特别有用的用例包括:

*   将数据复制到其他数据库，以便将数据提供给其他团队，或者作为流用于分析、数据湖或数据仓库。
*   微服务数据交换，在不同服务之间传播数据，无需耦合，本地保持优化视图，或用于整体到微服务的演进。
*   审计、缓存失效、全文搜索索引、更新 CQRS 读取模型等等。

## CDC Debezium Apache Kafka 连接器入门

Debezium Apache Kafka 连接器可通过 [红帽集成](https://www.redhat.com/en/products/integration)获得，红帽集成提供了一套全面的集成和消息传递技术，可跨混合基础架构连接应用程序和数据。这种敏捷、分布式、容器化和以 API 为中心的解决方案提供服务组合和编排、应用连接和数据转换、实时消息流、变更数据捕获和 API 管理，所有这些都与云原生平台和工具链相结合，以支持现代应用开发的所有方面。

这个 Red Hat Integration 版本为 Debezium 连接器提供了全面的支持，以捕捉 MySQL 连接器和 PostgreSQL 连接器的变化。MongoDB 和 SQL Server 的连接器现在包含在内，并作为[技术预览版](https://access.redhat.com/support/offerings/techpreview)交付。也被称为 Red Hat Integration CDC connectors，Debezium 为企业提供了开源的好处，如社区驱动的上游创新，提供企业级支持，帮助您的组织安全地使用开源技术。 查看完整的 [支持的配置](https://access.redhat.com/articles/4938181) 了解更多信息。

从 [红帽客户门户](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=red.hat.integration&downloadType=distributions) 下载红帽集成 Debezium CDC 连接器开始使用。此外，不要忘记查看 Gunnar Morling 关于 Debezium 和 Kafka 的网络研讨会，或者他在 QCon 的 [演讲。](https://www.infoq.com/presentations/data-streaming-kafka-debezium/)

*Last updated: June 29, 2020**