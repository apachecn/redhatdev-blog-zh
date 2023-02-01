# 首先看看新的 Apicurio 注册表 UI 和操作符

> 原文：<https://developers.redhat.com/blog/2020/06/11/first-look-at-the-new-apicurio-registry-ui-and-operator>

去年， [Apicurio](https://www.apicur.io/) 开发者社区启动了新的 [Apicurio Registry](https://github.com/Apicurio/apicurio-registry) 项目，这是一个用于微服务的 API 和模式注册表。您可以使用 Apicurio Registry 来存储和检索服务构件，如 OpenAPI 规范和 AsyncAPI 定义，以及模式，如 Apache Avro、JSON 和 Google Protocol Buffers。

因为注册表也可以作为一个目录，您可以在其中浏览工件，所以添加一个新的基于 web 的用户界面(UI)是当前 Apicurio Registry 1.2.2 版本的一个优先事项。在这个版本中，Apicurio 社区已经将 Apicurio 注册表作为二进制文件下载或从容器映像中获得。为了更容易地设置和管理您的 Apicurio 注册表部署，他们还为 Apicurio 注册表创建了一个新的 [Kubernetes 操作器。](https://github.com/Apicurio/apicurio-registry-operator)

本文快速介绍了新的 Apicurio 注册用户界面和 Apicurio 注册操作符。我将向您展示如何在 Apicurio 1.2.2 中访问这些新特性，并描述一些使用它们的亮点。更详细的演示，请查看我的视频教程，介绍新的 UI 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) [操作符](https://developers.redhat.com/topics/kubernetes/operators/)。

[https://www.youtube.com/embed/6v4PS5vaiZc?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/6v4PS5vaiZc?autoplay=0&start=0&rel=0)

## Apicurio 注册表用户界面

Apicurio 新的基于 web 的用户界面允许您浏览存储在注册表中的工件。您可以使用 UI 通过标签、名称或描述来搜索工件。您还可以预览一个工件的内容，并且可以选择在本地下载和存储一个工件。在注册表中，您可以检查任何存储的工件的所有可用版本。

除了浏览工件之外，Apicurio UI 还允许您配置和管理注册表的内容规则，既可以是全局的，也可以是针对每个工件的。当然，也可以上传新的工件，并更新任何现有工件的内容或版本。

要访问 Apicurio UI，只需导航到当前 Apicurio 注册中心部署的主要端点，比如`http://localhost:8080/`。在终点，您将被重定向到`/ui`路径。

## Apicurio 注册管理执行机构

您可以在任何工作的 Kubernetes 或 OpenShift 集群中部署 Apicurio Registry Operator，并使用它来快速安装和配置 Apicurio Registry 部署。有关说明，请参见 [Apicurio 注册管理执行机构部署快速入门](https://github.com/Apicurio/apicurio-registry-operator#quickstart)。未来，该运营商将可从 Kubernetes[Operator hub . io](https://operatorhub.io)目录中获得。

安装后，您可以创建一个 ApicurioRegistry 资源(请参见部署快速入门的内存示例)并将其配置为部署在集群中。如果您想要定制已部署的 ApicurioRegistry 资源，您可以定义持久性类型和入口路由配置之类的东西，以便在外部公开它。

## 结论

Apicurio Registry 和 Apicurio Registry Operator 使得发现、管理和使用服务规范和工件变得更加容易——不仅对于请求-响应同步 API，而且对于异步和事件驱动的架构也是如此。要获得 Apicurio 注册中心的技术预览和支持版本，请查看 T2 红帽集成服务注册中心。这个服务注册中心是融合模式注册中心的[替代，旨在帮助团队管理他们的服务模式。](https://developers.redhat.com/blog/2019/12/17/replacing-confluent-schema-registry-with-red-hat-integration-service-registry/)

与此同时，Apicurio 社区继续增强 Apicurio 的组件生态系统。新的 Apicurio 注册管理执行机构仍在开发中，但我们邀请您试用它，[向团队](https://github.com/Apicurio/apicurio-registry-operator)提供反馈。

*Last updated: January 17, 2022*