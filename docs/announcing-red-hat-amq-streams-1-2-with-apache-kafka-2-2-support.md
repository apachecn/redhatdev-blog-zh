# 宣布推出支持 Apache Kafka 2.2 的红帽 AMQ 流 1.2

> 原文：<https://developers.redhat.com/blog/2019/07/04/announcing-red-hat-amq-streams-1-2-with-apache-kafka-2-2-support>

我们很高兴地宣布我们的消息套件的数据流组件的更新版本，红帽 AMQ 流 1.2，这是[红帽集成](https://www.redhat.com/en/products/integration)的一部分。

基于 [Apache Kafka](https://www.redhat.com/en/topics/integration/what-is-apache-kafka) 项目的红帽 AMQ 流提供了一个分布式主干，允许微服务和其他应用以极高的吞吐量和极低的延迟共享数据。AMQ 流使运行和管理 Apache Kafka 成为 Kubernetes 的本地体验，通过额外提供 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 操作器，一种在 Kubernetes 上部署、管理、升级和配置 Kafka 生态系统安装的简化和自动化方式。

## 特征

这个新版本增加了两个主要特性:

*   支持 Apache Kafka 2.2.1
*   支持 Red Hat OpenShift 容器平台 4.1

AMQ 流现在可以使用 YAML 文件手动安装在 OpenShift 4.x 上，或者通过 OpenShift 容器平台操作中心安装。此外，描述随 AMQ 流提供的 CRD 的 YAML 文件现在支持多个版本。此外，现在可以在 JBOD 存储中添加或删除卷，并且现在可以增加现有 AMQ 流集群中用于存储消息和日志的持久卷的大小。为了简化部署，AMQ 流的容器映像数量已经显著减少。

除了上述新功能，此版本还包括突破性的预览:

*   AMQ 流 HTTP 桥
*   Debezium 变更数据捕获(CDC)连接器

CDC 连接器是基于上游项目 [Debezium](https://debezium.io/) 的，可用于红帽 AMQ 流。当在容器中运行时，Debezium 捕获对数据库表的行级更改，并将相应的更改事件传递给 OpenShift 容器平台上的 AMQ 流，以便进一步处理。

## AMQ 流 HTTP 桥

AMQ 流 HTTP 桥为 AMQ 流提供了一个 RESTful 接口，提供了 web API 的优势，易于使用并与 AMQ 流连接，无需解释 Kafka 协议。

HTTP 桥支持 HTTP 生产者和消费者请求:

*   制作唱片
*   消费记录
*   创造消费者
*   删除消费者
*   从主题和分区中检索数据
*   向分区和主题提交偏移量

这些方法提供 JSON 响应和 HTTP 响应代码错误处理。

## 红帽集成

AMQ 流 1.2 今天作为红帽集成产品的一部分推出。对于采用云原生开发模式的客户，Red Hat 提供了完整的技术组合，使他们能够[构建基于微服务的应用](https://www.redhat.com/en/technologies/cloud-computing/openshift/application-runtimes) [，](https://www.redhat.com/en/topics/integration/what-is-integration) [在它们之间同步或异步交换信息](https://www.redhat.com/en/technologies/jboss-middleware/amq) [，](https://www.redhat.com/en/topics/integration/what-is-integration) [将它们与遗留系统](https://www.redhat.com/en/technologies/jboss-middleware/fuse) [集成，并在](https://www.redhat.com/en/topics/integration/what-is-integration) [行业领先的企业容器和 Kubernetes 应用平台](https://www.openshift.com/) [上运行。](https://www.redhat.com/en/topics/integration/what-is-integration)

*Last updated: September 3, 2019*