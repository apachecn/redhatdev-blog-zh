# 用于变更数据捕获(CDC)的 Debezium Apache Kafka 连接器的开发者预览版

> 原文：<https://developers.redhat.com/blog/2019/07/10/developer-preview-of-debezium-apache-kafka-connectors-for-change-data-capture-cdc>

随着 [Red Hat AMQ 流 1.2](https://developers.redhat.com/blog/2019/07/04/announcing-red-hat-amq-streams-1-2-with-apache-kafka-2-2-support/) 的发布， [Red Hat Integration](https://www.redhat.com/en/topics/integration/what-is-integration) 现在包括变更数据捕获(CDC)功能的开发者预览版，以支持基于现代云原生微服务的应用的数据集成。CDC 特性基于上游项目 [Debezium](https://debezium.io/) ，并与 Apache Kafka 和 Strimzi 进行了本机集成，以运行在 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview)、企业 Kubernetes 之上，作为 AMQ 流版本的一部分。

## 什么是变更数据捕获(CDC)？

变更数据捕获(或 CDC)是一种旧的软件设计模式，用于监控和捕获数据中的变更，以便其他软件可以响应这些变更。CDC 经常在数据仓库解决方案中用于对 OLTP 数据变化做出反应，作为分布式数据集成方法的一部分，它在微服务架构中获得了新生。

当在容器环境中运行时，CDC 捕获对数据库表的行级更改，并将相应的更改事件传递给数据流总线。应用程序可以读取这些*变更事件流*，并按照事件发生的顺序访问变更事件。

CDC 的主要用途是使应用程序能够在数据库中的数据发生变化时几乎立即做出响应。应用程序可以对插入、更新和删除事件做任何事情，包括:

*   简化数据集成和数据复制
*   缓存失效和提供全文搜索索引
*   从简化的整体应用中提取微服务

## 什么是 Debezium？

Debezium 是一组分布式服务，它捕获数据库中行级别的变化，以便应用程序可以看到并响应这些变化。Debezium 在一个事务日志中记录提交给每个数据库表的所有行级更改。应用程序只需读取它们感兴趣的事务日志，并按照事件发生的顺序查看所有事件。Debezium 持久而快速，因此应用程序可以快速响应，即使出现问题也不会错过任何事件。

Debezium 提供了用于监控以下数据库的连接器:

*   [MySQL 连接器](https://developers.redhat.com/download-manager/file/debezium-connector-mysql-0.10.0.Beta2-redhat-00001-plugin.zip)
*   [PostgreSQL 连接器](https://developers.redhat.com/download-manager/file/debezium-connector-postgres-0.10.0.Beta2-redhat-00001-plugin.zip)
*   [MongoDB 连接器](https://developers.redhat.com/download-manager/file/debezium-connector-mongodb-0.10.0.Beta2-redhat-00001-plugin.zip)
*   [SQL Server 连接器](https://developers.redhat.com/download-manager/file/debezium-connector-sqlserver-0.10.0.Beta2-redhat-00001-plugin.zip)

Debezium 连接器将所有事件记录到红帽 AMQ 流 Kafka 集群，应用程序通过 AMQ 流消费这些事件。Debezium 使用 Apache Kafka Connect 框架制作所有的 Debezium 连接器、Kafka 连接器源连接器，因此，它们可以使用 AMQ 流 Kafka Connect 资源进行部署和管理。

## 红帽集成

Red Hat integration 的 Debezium CDC Apache Kafka 连接器今天作为 Red Hat Integration 的一部分作为开发者预览版提供。对于采用云原生开发模式的客户，Red Hat 提供了完整的技术组合，使他们能够[构建基于微服务的应用](https://www.redhat.com/en/technologies/cloud-computing/openshift/application-runtimes)，[在它们之间同步或异步交换信息](https://www.redhat.com/en/technologies/jboss-middleware/amq)，[将它们与遗留系统](https://www.redhat.com/en/technologies/jboss-middleware/fuse)集成，并在[行业领先的企业容器和 Kubernetes 应用平台](https://www.openshift.com/)上运行。

如果您有任何关于在 OpenShift 上使用 Red Hat AMQ 流运行 Debezium 开发者预览版的请求或问题，请发送电子邮件至 [debezium-cdc-preview](mailto:debezium-cdc-preview@redhat.com) 邮件列表告知我们。欲了解更多关于 Debezium 和 AMQ 溪流的信息，请参见 https://debezium.io/docs/amq-streams/

*Last updated: September 3, 2019*