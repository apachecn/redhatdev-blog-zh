# Red Hat Integration Service Registry 1.1 GA 中的新功能和存储选项

> 原文：<https://developers.redhat.com/blog/2020/12/09/new-features-and-storage-options-in-red-hat-integration-service-registry-1-1-ga>

本文介绍了[Red Hat Integration](https://www.redhat.com/en/products/integration)service registry 中新的存储安装选项和特性。服务注册组件基于 [Apicurio](https://www.apicur.io/) 。您可以使用它来存储和检索服务构件，如 OpenAPI 规范和 AsyncAPI 定义，以及 Apache Avro、JSON 和 Google Protobuf 等模式。我们已经在 [Red Hat Integration 2020-Q4](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q4/html/release_notes_for_red_hat_integration_2020-q4/registry-relnotes) 中提供了 Red Hat Integration 的 Service Registry 1.1 组件作为正式发布(GA)版本。

## 什么是服务注册中心？

Red Hat Integration service registry 是标准事件模式和 API 设计的数据存储库。作为开发人员，您可以使用它将数据结构从应用程序中分离出来。您还可以使用 REST 接口来共享和管理您的数据结构。Red Hat 的服务注册中心是建立在开源社区项目 Apicurio Registry 的基础上的。

服务注册中心处理以下数据格式:

*   Apache Avro 模式
*   Apache Kafka 连接模式
*   JSON 模式
*   Google 协议缓冲区模式
*   谷歌协议缓冲文件描述符
*   OpenAPI 规范
*   AsyncAPI 规范
*   GraphQL 模式
*   Web 服务定义语言
*   XML 模式定义

为了帮助防止无效内容被添加到注册表中，您可以为每个添加的工件配置规则。在您上传新版本之前，工件的所有规则必须通过。

## 两个新的存储安装选项

最初的服务注册中心版本关注于 [Apache Kafka](https://developers.redhat.com/topics/kafka-kubernetes) 模式管理的用例。因此，我们使用 Kafka Streams API 进行存储。随着更广泛的使用案例的出现，我们看到了对额外存储机制的需求。Service Registry 1.1 组件提供了两个新的存储安装选项，目前正在[进行技术预览](https://access.redhat.com/support/offerings/techpreview/)。

该版本使用嵌入式 [Infinispan](https://infinispan.org/) 10 提供基于高速缓存的存储。我们还提供基于 [Jakarta 持久性 API](https://jakarta.ee/specifications/persistence/2.2/apidocs/) 的 PostgreSQL 12 存储。

## Service Registry 1.1 的新功能

除了新的存储选项，此版本还包括一些有趣的功能:

*   REST 客户机的依赖性更少，打包更容易。
*   改进的序列化和反序列化类，支持 [Apache Avro](https://avro.apache.org/) 中的 JSON 编码。
*   新的环境变量可以更灵活地控制 URL、Kafka 主题和全局规则。
*   web 控制台中的新格式按钮，可读性更好。
*   改进的操作员标签和指标。

**注**:参见 [Apicurio GitHub 库](https://github.com/Apicurio/apicurio-registry-examples)中演示如何使用 Kafka 客户端、JSON schema、Protobuf 和 Apache Avro 序列化器和反序列化器的示例应用程序。

## 使用 Red Hat 集成服务注册表

Red Hat Integration service registry 是一个工件存储库，它统一了 REST 和异步 API，形成一个完整的[事件驱动架构](https://developers.redhat.com/topics/event-driven)。您可以将它用作 Kafka 的简单[模式注册表，具体化](https://developers.redhat.com/blog/2019/12/16/getting-started-with-red-hat-integration-service-registry/) [Debezium](https://developers.redhat.com/blog/2020/04/14/capture-database-changes-with-debezium-apache-kafka-connectors/) 模式，或者作为您的 OpenAPI 文档的目录。序列化器和反序列化器类支持使用 Red Hat Integration service registry 作为 Apache Kafka 客户端的汇合模式注册表的[插件替换。](https://developers.redhat.com/blog/2019/12/17/replacing-confluent-schema-registry-with-red-hat-integration-service-registry/)

## 结论

新的 Red Hat Integration Service Registry 1.1 组件提供了两个新的存储安装选项和一些附加功能，以改善您的开发体验。该组件在 [Red Hat Integration 2020-Q4](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q4/html/release_notes_for_red_hat_integration_2020-q4/registry-relnotes) 中正式发布。

*Last updated: December 4, 2020*