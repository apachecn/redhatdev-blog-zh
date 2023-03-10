# 欢迎阿帕奇卡夫卡来到库伯内特时代！

> 原文：<https://developers.redhat.com/blog/2018/10/25/welcome-apache-kafka-to-the-kubernetes-era>

本周我们有非常令人兴奋的消息，Red Hat 宣布他们的 Apache Kafka Kubernetes 操作系统正式上市。Red Hat AMQ 流在 OpenShift 之上提供了管理 Apache Kafka 的机制，open shift 是我们为 Kubernetes 提供的企业发行版。

一切始于去年 2018 年 5 月，当时大卫英厄姆(@哈丁) [公布了开发者预览](https://developers.redhat.com/blog/2018/05/07/announcing-amq-streams-apache-kafka-on-openshift/) 作为 红帽 AMQ 的新成员。[红帽 AMQ 流](https://access.redhat.com/products/red-hat-amq-streams)专注于在 OpenShift 上运行 Apache Kafka。在[微服务](https://developers.redhat.com/topics/microservices/)世界中，一些组件需要依赖于高吞吐量的通信机制，Apache Kafka 因其作为构建数据管道和流应用程序的领先实时分布式消息传递平台而闻名。

作为传统基础设施部署的领导者，Apache Kafka 在新的 Kubernetes 时代缺少一些易于使用的容器——本地公民。于是，一个团队在 2017 年分组打造了上游 [Strimzi](http://strimzi.io/) 项目。这个团队致力于应用新的 [操作符模式](https://coreos.com/blog/introducing-operators.html) 来解决这个问题。随着传统 Apache Kafka broker 上部署的新组件的开发，这些新的 Kubernetes 操作员现在能够管理集群范围的资源以及作为主题和身份验证用户的实体。

更重要的是，这些 Kubernetes 操作符使用起来非常简单，涵盖了集群的大多数常见配置管理。在 OpenShift 中安装了一些 Kubernetes 自定义资源定义之后，任何用户都可以通过创建一个新的 Kafka 资源定义来创建一个 Apache Kafka 集群。然后，集群操作者将采用该定义并提供所需的组件，以便在您的 OpenShift 基础设施上拥有一个完全部署的集群。这同样适用于为同一个 Kafka 集群创建主题和用户。

最后，最有趣的部分是配置集群的简单配置，可以通过节点端口、负载平衡器或路由从 OpenShift 集群外部轻松获得，后者是开始工作的最简单方式。根据您的应用程序需求，您可以选择使用简单的基于 TLS 的安全方法或传统的节点端口设置。因此，AMQ 流运营商嵌入式逻辑创建动态服务、路由和证书，以从外部客户端访问 Apache Kafka 集群。

是的，听起来很神奇。就是这么好！下载适用于 OpenShift 开发环境的 Red Hat Container 开发工具包，并遵循我们的 [入门指南](https://access.redhat.com/documentation/en-us/red_hat_amq_streams/1.0/html/using_amq_streams/) ，尝试一下 Red Hat AMQ 流。另外，请注意我的[如何在 OpenShift 上运行 Kafka 的指南，企业 Kubernetes](https://developers.redhat.com/blog/2018/10/29/how-to-run-kafka-on-openshift-the-enterprise-kubernetes-with-amq-streams/) ！

* * *

以下是一些关于阿帕奇卡夫卡和红帽 AMQ 溪流的其他文章:

*   [宣布 AMQ 流:OpenShift 上的阿帕奇卡夫卡](https://developers.redhat.com/blog/2018/05/07/announcing-amq-streams-apache-kafka-on-openshift/)
*   [在 OpenShift 上使用 Apache Kafka 进行智能电表数据处理](https://developers.redhat.com/blog/2018/07/16/smart-meter-streams-kafka-openshift/)
*   [Event flow:open shift 上的事件驱动微服务(第一部分)](https://developers.redhat.com/blog/2018/10/15/eventflow-event-driven-microservices-on-openshift-part-1/)
*   [使用 Red Hat Decision Manager 7 检测信用卡欺诈](https://developers.redhat.com/blog/2018/07/26/detecting-credit-card-fraud-with-red-hat-decision-manager-7/)
*   [介绍 Kafka-CDI 库](https://developers.redhat.com/blog/2018/05/31/introducing-the-kafka-cdi-library/)

*Last updated: September 3, 2019*