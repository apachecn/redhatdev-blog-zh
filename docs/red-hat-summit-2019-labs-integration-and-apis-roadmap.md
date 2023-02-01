# 红帽峰会 2019 实验室:集成和 API 路线图

> 原文：<https://developers.redhat.com/blog/2019/04/12/red-hat-summit-2019-labs-integration-and-apis-roadmap>

[红帽峰会 2019](https://www.redhat.com/en/summit/2019/?intcmp=701f20000012i8UAAQ) 将于 5 月 7 日至 9 日在[波士顿会展中心](https://www.signatureboston.com/BCEC)举办。关于开源企业级软件的现状，你需要知道的一切都可以在这个活动中找到。从客户谈论他们在解决方案中利用开源的经历，到您正在使用的开源技术的创造者，一直到关于这些技术的动手实验室体验。

这种亲身实践的吸引力正是这一系列文章的内容。之前，我们查看了云原生应用开发专题讲座中的[实验室，这一次，我们提供了“集成和 API”实验室内容的路线图。](https://developers.redhat.com/blog/2019/03/31/red-hat-summit-2019-labs-cloud-native-app-dev-roadmap/)

通过搜索标题或过滤“讲师指导实验”和“集成&API”，可以在在线的[课程目录中找到以下实验](https://summit.redhat.com/conference/sessions)

### 与 API 和容器的敏捷集成研讨会

与 API 和容器的敏捷集成会议是一个实践研讨会，旨在开发、测试和部署集成的云原生解决方案。

这个 2 小时的实验将从敏捷集成的概述和必要架构的讨论开始。我们还将展示客户如何使用 Red Hat 的敏捷集成方法来保持竞争力的例子。这个实践研讨会是为将领导 API 开发和安全活动的集成商而设计的。这些活动是 UI 驱动的，允许集成者成功地部署、集成(Red Hat Fuse)、保护和管理 API 服务。我们还将涵盖加速云原生应用程序的开发、开发以 API 为中心的服务、提供 API 安全性以及建立运营管理。
*演讲嘉宾:西蒙·格林、约西·科伦、克里斯蒂娜·梅玮·林、维奈·巴勒劳*

### 面向企业的敏捷集成

组织投资于深入的技术组合，以满足不同的业务需求。这些系统的互连是企业成功的基础。为了跟上竞争的步伐，解决方案必须实现可伸缩性，以满足市场需求，以及满足业务利益相关者的要求。敏捷集成是寻求扩展和支持高要求涉众期望的组织取得成功的关键。

在本实验中，您将学习在各种使用案例中使用 Red Hat 的集成产品组合，包括:

*   无需编码即可集成应用程序。
*   实现使用多个后端服务的高级集成场景。
*   熟悉各种部署方法。
*   构建容错微服务应用。
*   通过将这些系统公开为微服务，促进与遗留系统的集成。
*   设计、公开和管理 REST APIs。
*   将服务级别协议应用于微服务监控。
*   使用云原生基础架构托管高可用性集成解决方案。

您将获得 Red Hat Fuse Online、Red Hat Fuse for open shift Container Platform、Red Hat Fuse for Red Hat JBoss Enterprise Application Platform(EAP)、ISTIO、Red Hat 3scale API 管理和 Red Hat OpenShift 应用程序运行时的实践经验。

*演讲者:安德鲁·布洛克、查德·达比、洪华钦*

### 借助 Apache Kafka 和 Debezium 跨越微服务界限

领域驱动设计建议将大型系统分割成有限的上下文。由独立团队作为松散耦合的微服务来实现，这种模式让组织能够快速适应新的业务需求。虽然它们不应该共享诸如公共数据库之类的资源，但是服务也不是孤立存在的:通常一个服务需要来自另一个服务的数据来提供其功能。

在这个实验室里。我们将涵盖:

*   微服务如何使用 Apache Kafka 共享数据，同时保持适当的隔离和独立性。
*   如何使用变更数据捕获(CDC)将数据变更直接从数据库中导出，而无需对应用程序进行任何更改。
*   如何将微服务拥有的数据传播到同步系统，如缓存和全文搜索索引。

基于红帽 OpenShift、红帽 AMQ 流、Kafka 和 Debezium，一个开源的 CDC 解决方案。该动手实验将指导您完成在微服务之间成功实施异步数据交换模式的步骤。

*发言人:伊曼纽尔·伯纳德、贡纳尔·莫林、马里乌斯·博戈维奇、保罗·帕蒂尔诺*

### 学习使用带有 3scale 和 OpenShift 的 Camel Rest DSL

本实验将介绍开源集成框架 Apache Camel、Red Hat Fuse 的上游项目，以及 Red Hat 3scale API Automation 和 Red Hat OpenShift 容器平台。您将学习骆驼的基本知识，并逐步了解 Spring Boot 路线的开发和部署。我们还将介绍如何将 Camel 与 OpenShift 容器平台和 3Scale API 自动化结合使用，以实现 web 级应用程序和完全托管的 API。由于企业应用程序中对 REST APIs 的普遍需求，我们将使用 Camel REST DSL 的例子来介绍如何开始编写 REST Camel route。

来了解如何将 camel routes 部署到 OpenShift 容器平台上，并使用 3scale API 管理设置 API 管理，以管理您的 API 使用、URL 等。
 *演讲者:克劳斯·易卜生、玛丽·科克伦、达斯丁·汉弗莱斯*

### 导航混合云集成—黑客马拉松

勇于创新，利用 Red Hat 集成技术创建您自己的混合集成解决方案，以解决日常集成挑战。你会得到一组集成问题供你选择，并访问红帽 Fuse，红帽的集成平台。您可以单独或分组工作，在此基础上构建您的集成解决方案。常见的挑战包括连接到棕地系统、SaaS 应用程序、处理事件流和提供 API。

*演讲者:加里·高汉、尼古拉·费拉罗、克里斯蒂娜·梅玮·林、埃文·肖特里斯、雨果·格雷罗·奥利瓦雷斯*

### 用 AMQ 流在 OpenShift 上运行 Apache Kafka

在本实验中，您将通过 Red Hat AMQ 流，学习在 Red Hat OpenShift 容器平台上部署、操作 Apache Kafka 集群并与之交互的实际操作。我们将介绍如何:

*   使用 AMQ 流操作器和 Kubernetes 自定义资源(CRDs)管理 OpenShift 容器平台上的 Apache Kafka 集群、主题和用户。
*   通过监控特定项目，自助服务您的 Kafka 集群、主题和用户。
*   使用持久卷创建持久 Kafka 集群。
*   使用一组示例应用程序，从 OpenShift 容器平台实例的内部和外部与 Kafka 集群进行交互。
*   使用 Grafana 和 Prometheus 管理和监控您的 Kafka 集群。
*   使用 AMQ 流集群运营商部署的 MirrorMaker 实施跨数据中心解决方案。
*   处理用于加密和身份验证的群集和客户端 TLS 证书。

演讲者:马里乌斯·博戈维奇、保罗·帕蒂尔诺、伊曼纽尔·伯纳德、贡纳尔·莫林

请立即在 2019 年红帽峰会[上注册其中一个由讲师指导的实验室。我们期待在那里见到您！](https://www.redhat.com/en/summit/2019/?intcmp=701f20000012i8UAAQ)

*Last updated: April 22, 2019*