# 使用 Debezium Db2 连接器捕获 IBM Db2 数据变更

> 原文：<https://developers.redhat.com/blog/2020/11/05/capture-ibm-db2-data-changes-with-debezium-db2-connector>

本文介绍了用于变更数据捕获的新的 [Debezium Db2 连接器](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q3/html/debezium_user_guide/debezium-connector-for-db2)，现在可以作为来自 [Red Hat Integration](https://developers.redhat.com/integration) 的[技术预览版](https://access.redhat.com/support/offerings/techpreview/)获得。快速了解如何在[红帽 AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet) Kafka 集群中使用 [Debezium](https://debezium.io/) ，然后了解如何使用新的 Db2 连接器来捕获 Db2 数据库表中的行级变化。

**注意** : *变更数据捕获*，或称 CDC，是一种成熟的软件设计模式，用于监控和捕获数据库中的数据变更。CDC 捕获对数据库表的行级更改，并将相应的更改事件传递给数据流总线。应用程序可以读取变更事件流，并按照事件发生的顺序访问变更事件。

## 什么是 Debezium？

[Debezium](https://debezium.io/) 是一组分布式服务，它捕获行级别的数据库更改，以便应用程序可以看到并响应这些更改。Debezium 连接器将所有事件记录到红帽 AMQ 流 Kafka 集群。应用程序使用 AMQ 流来消费变更事件。

AMQ 流是红帽集成组件，提供[红帽的 Apache Kafka](https://developers.redhat.com/topics/kafka-kubernetes) 发行版和流行的[云原生计算基础沙盒](https://www.cncf.io/sandbox-projects/)项目 [Strimzi](https://strimzi.io/) 。AMQ 流使运行和管理卡夫卡成为一种本地体验。AMQ 流[运营商](https://developers.redhat.com/topics/kubernetes/operators)在 OpenShift 上提供了一种部署、管理、升级和配置 Kafka 生态系统的简化和自动化方式。

## Db2 的 Debezium 连接器

Debezium 的 Db2 连接器受到了 Debezium 的 SQL Server 实现的启发。这个连接器基于用于 Db2 中 SQL 复制的[抽象语法符号(ASN)捕获和应用代理](https://www.ibm.com/support/pages/q-replication-and-sql-replication-product-documentation-pdf-format-version-101-linux-unix-and-windows)。Db2 连接器代理为处于捕获模式的表生成变更数据。它们还监视表并存储表更新的更改事件。然后，Debezium 连接器使用一个 SQL 接口来查询变更事件的变更数据表。

### Db2 连接器如何工作

Db2 连接器使用 ASN 库，ASN 库是 Db2 for Linux 的标准部分。要使用 ASN 和 Db2 连接器，您必须拥有 IBM info sphere Data Replication(IIDR)产品的许可证。但是，您不需要安装 IIDR。Db2 连接器已经过 Db2/Linux 11.5.0.0 的测试。

Debezium Db2 连接器为每个行级的`INSERT`、`UPDATE`和`DELETE`操作生成一个数据更改事件。每个事件都包含一个键和值。键和值的结构取决于被更改的表。

在运行 Debezium Db2 连接器来捕获提交给 Db2 数据库的更改之前，数据库管理员必须将表置于捕获模式。Debezium 提供了一组用户定义的函数，您可以使用它们来实现这个目的。

### 安装 Db2 连接器

要安装 Debezium 的 Db2 连接器，请按照文档 [*中的步骤在 OpenShift*](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-Q3/html-single/installing_debezium_on_openshift/) 上安装 Debezium。不幸的是，由于许可要求，IBM Db2 的 JDBC 驱动程序不作为 Debezium 连接器的一部分提供。您可以手动下载并复制 JDBC 驱动程序，作为提供的容器映像的一部分。

当连接器启动时，它会对连接器被配置为监视的 Db2 数据库表拍摄一个一致的快照。然后，连接器为行级操作生成数据更改事件，并将更改事件记录传输到 Kafka 主题。图 1 显示了一个 Db2 连接器配置。

[![](img/c3d53b6e8061d6e58c7cc5a0549a5db5.png "img_5f90a3f337e40")](/sites/default/files/blog/2020/10/img_5f90a3f337e40.png)

Figure 1: A sample Db2 connector configuration for AMQ Streams Kafka Connect.

## Debezium Apache Kafka 连接器入门

[Debezium Apache Kafka 连接器](https://developers.redhat.com/topics/event-driven/connectors)可通过 [Red Hat Integration](https://developers.redhat.com/integration) 获得，它提供了一套全面的集成和[消息传递技术](https://developers.redhat.com/topics/event-driven)，可跨混合基础设施连接应用程序和数据。这种敏捷、分布式、容器化和以 API 为中心的解决方案提供了服务组合和编排、应用程序连接和数据转换、实时消息流、变更数据捕获和 [API 管理](https://developers.redhat.com/topics/api-management)—所有这些都与云原生平台和工具链相结合，以支持现代应用程序开发的所有方面。

从[红帽开发者门户](https://developers.redhat.com/products)下载红帽集成 Debezium 连接器开始。您还可以查看 [Gunnar Morling 在](https://developers.redhat.com/videos/youtube/QYbXDp4Vu-8/) [DevNation Tech Talks](https://developers.redhat.com/devnation/the-show) 系列中关于 Debezium 和 Kafka(2019 年 2 月)的网络研讨会，或者他最近在 QCon(2020 年 1 月)上的[演讲。](https://www.infoq.com/presentations/data-streaming-kafka-debezium/)

*Last updated: February 14, 2022*