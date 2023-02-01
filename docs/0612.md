# Red Hat 将 Apache Kafka 支持的 Debezium CDC 连接器提升到技术预览版

> 原文：<https://developers.redhat.com/blog/2019/11/22/red-hat-advances-debezium-cdc-connectors-for-apache-kafka-support-to-technical-preview>

在几个月的开发者预览版之后，用于变更数据捕获(CDC)的 Debezium[Apache Kafka](https://www.redhat.com/en/topics/integration/what-is-apache-kafka)连接器现在可以作为 [技术预览版](https://access.redhat.com/support/offerings/techpreview) 的一部分，作为第四季度发布的 [红帽集成](https://www.redhat.com/en/products/integration) 。技术预览功能提供了对即将推出的产品创新的早期访问，使您能够在开发过程中测试功能并提供反馈。

Red Hat Integration 提供了 Debezium 连接器，用于从以下数据库中捕获变更:

*   MySQL 连接器
*   PostgreSQL 连接器
*   MongoDB 连接器
*   SQL Server 连接器

Debezium 连接器基于流行的[Apache Kafka Connect API](https://kafka.apache.org/documentation.html#connect)，适合沿着 [红帽 AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet) Kafka 集群部署。

AMQ 流是红帽集成组件，提供红帽发行的阿帕奇卡夫卡和流行的 [CNCF 沙盒](https://www.cncf.io/sandbox-projects/) 项目[strim zi](https://strimzi.io/)。AMQ 流通过提供 OpenShift 操作符，使运行和管理 Kafka 成为 OpenShift 原生体验 ，OpenShift 操作符提供了一种在 open shift 上部署、管理、升级和配置 Kafka 生态系统的简化和自动化方法。

有了这一补充，Red Hat Integration 现在可以提供更多组件来连接整个企业生态系统中的系统。随着 [阿帕奇骆驼的 200 个连接器](https://developers.redhat.com/products/fuse/connectors) ，用户几乎可以连接到任何东西——从传统系统到软件即服务(SaaS)应用，以及应用编程接口(API)到物联网(IoT)设备。

### 什么是变更数据捕获(CDC)？

变更数据捕获，简称 CDC，是一种为系统建立的软件设计模式，它监控和捕获数据的变更，以便其他软件能够响应这些变更。CDC 捕获对数据库表的行级更改，并将相应的更改事件传递给数据流总线。应用程序可以读取这些变更事件流，并按照事件发生的顺序访问这些变更事件。

### 什么是 Debezium？

[Debezium](https://debezium.io/) 是一组分布式服务，它捕获数据库中行级别的变化，以便应用程序可以看到并响应这些变化。Debezium 连接器将所有事件记录到红帽 AMQ 流 Kafka 集群，应用程序通过 AMQ 流消费这些事件。

### CDC 与 Debezium 合作

你可以查看 Sadhana Nandakumar 的 [博文](https://developers.redhat.com/blog/2019/09/03/cdc-pipeline-with-red-hat-amq-streams-and-red-hat-fuse/) ，她在博文中解释了如何利用 Red Hat 集成创建一个完整的 CDC 管道。在这个例子中，她使用 Debezium 捕捉发生的变化，并使用 Red Hat AMQ 流对其进行流处理。然后，她使用 Red Hat Fuse 对数据进行过滤和转换，并将其发送到 Elasticsearch，在那里数据可以被下游系统进一步分析或使用。

你可以从 [红帽开发者网站](https://developers.redhat.com/products/amq/download) 下载红帽集成 Debezium CDC 技术预览连接器。如果您有关于运行 Debezium 技术预览版的请求或问题，请发送电子邮件至[Debezium-CDC-Preview 邮件列表](mailto:debezium-cdc-preview@redhat.com) ，让我们知道。

*Last updated: July 1, 2020*