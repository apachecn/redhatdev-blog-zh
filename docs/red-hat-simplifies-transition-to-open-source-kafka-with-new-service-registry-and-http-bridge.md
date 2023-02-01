# Red Hat 通过新的服务注册中心和 HTTP 桥简化了向开源 Kafka 的过渡

> 原文：<https://developers.redhat.com/blog/2019/11/26/red-hat-simplifies-transition-to-open-source-kafka-with-new-service-registry-and-http-bridge>

通过在 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 和 [Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux/) 上运行 [Apache Kafka](https://developers.redhat.com/videos/youtube/QYbXDp4Vu-8/) ，Red Hat 继续为寻求实现 100%开源、[事件驱动架构](https://developers.redhat.com/topics/event-driven/) (EDA)的用户增加可用功能。 [Red Hat Integration](https://www.redhat.com/en/products/integration) Q4 版本提供了新的特性和功能，包括旨在简化 Apache Kafka 的 [AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet)发行版的使用和部署。

这个新版本中可用的功能包括:

*   服务注册作为技术预览
*   HTTP-Kafka bridge 现在普遍可用
*   具有 3 级 API 管理的安全 HTTP-Kafka 桥

服务注册中心和 HTTP Kafka 桥的加入改善了 Red Hat 作为云原生 Kafka 工作负载的 100%开源平台的定位。这些新特性成为基于 [CNCF 沙盒项目 Strimzi](https://strimzi.io/2019/09/06/cncf.html) 的 OpenShift 的 Kafka 操作符用法的适当补充。

## 事件驱动架构的模式服务注册

Red Hat Integration 的 service registry 基于 [Apicurio 项目](https://www.apicur.io/) registry，提供了一种将用于序列化和反序列化 Kafka 消息的模式与发送/接收它们的应用程序分离的方法。注册中心是模式(和 API 设计)工件的存储，提供了用于访问管理的 REST API 和一组用于实施内容有效性和演化的可选规则。

Apicurio 服务注册表处理以下数据格式:

*   阿帕奇 Avro
*   JSON 模式
*   协议缓冲区
*   OpenAPI
*   AsyncAPI

除了注册表本身，用户还可以利用包含的定制 Kafka 序列化器和反序列化器(SerDes)。这些 SerDes Java 类允许 Kafka 应用程序从服务注册中心获取相关的模式，而不需要将模式与应用程序捆绑在一起。

相应地，注册中心有自己的 REST API 来创建、更新和删除工件，以及管理全局和每个工件的规则。registry API 与另一个 Kafka 提供商的模式注册表兼容，以方便无缝迁移到 AMQ 流，作为一种替代方案。

对于即将到来的[技术预览版](https://access.redhat.com/support/offerings/techpreview)，只有 Avro 格式将被包含在红帽集成版的服务注册中。

## 通过 HTTP 连接到 Kafka

Apache Kafka 使用 TCP/IP 之上的自定义协议在应用程序和集群之间进行通信。客户机可用于许多不同的编程语言，但在许多情况下，诸如 HTTP/1.1 之类的标准协议更合适。

[红帽 AMQ 流 Kafka 桥](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html-single/using_amq_streams_on_openshift/index#assembly-kafka-bridge-overview-str)提供了一个 API，用于集成基于 HTTP 的客户端和运行在 AMQ 流上的 Kafka 集群。应用程序可以执行典型的操作，例如:

*   向主题发送消息。
*   订阅一个或多个主题。
*   从订阅的主题接收消息。
*   提交与收到的消息相关的偏移量。
*   寻求一个特定的位置。

用户可以通过使用 AMQ 流操作符或类似于 AMQ 流安装的操作符，将 Kafka 桥部署到 OpenShift 集群中，并且用户可以下载 Kafka 桥文件以安装在 Red Hat Enterprise Linux 上。

用户可以使用 Red Hat Integration 的 API 管理功能通过使用 3scale 组件保护 Kafka 桥来提供 TLS 支持、身份验证和授权。与 API 管理的集成也意味着度量、速率限制和计费等附加功能可用。

## 摘要

Red Hat Integration 第 4 季度发布版使 Red Hat 的 AMQ 流分发版成为更好的开源平台，可用于云原生 Kafka 工作负载。Service Registry 技术预览版为事件管理总线不断变化的领域中的数据治理提供了一个公共基础。Red Hat 集成中 HTTP 桥的普遍可用性增强了开发人员在使用 Apache Kafka 构建应用程序时的选择。

*Last updated: July 1, 2020*