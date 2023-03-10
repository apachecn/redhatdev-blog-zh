# 使用 Red Hat 集成轻松创建完整 API 生命周期的 API(第 1 部分)

> 原文：<https://developers.redhat.com/blog/2019/02/11/red-hat-integration-effortless-api-creation>

如今，具有适当生命周期管理的 API 开发通常需要几天甚至几周的时间来启动和运行一个简单的 API 服务。这背后的一个主要原因是这个过程中总是有太多的参与方。另外还有几个小时的开发和配置时间。

首先，系统分析师与 API 消费者协商 API 接口；然后开发人员编写实际的 API 来实现接口。然后，他们将 API 传递给负责部署 API 的 DevOps 团队。这还没有完成；然后，需要将部署信息传递给负责在管理系统中设置 API 端点以及应用访问策略的运营团队。

提供托管 API 服务的速度可能是公司业务成功的主要因素之一。

本文是三篇系列文章中的第一篇，描述了新的 Red Hat Integration bundle 如何允许 citizen integrators 通过工具快速提供一个 API，使得在五个简单的步骤中创建一个 API 变得毫不费力。

 **Red Hat Integration bundle 允许您通过五个简单的步骤快速提供 API:

1.直观地设计 OpenAPI 标准文档。

2.在 API 定义之后立即实现它。

3.立即在云上部署 API。

4.自动发现服务并将 API 加载到管理系统中。

5.应用策略来保护和管理 API，您就可以开始了！

## 演示

在这个演示中，一家葡萄酒经销商通过 API 向其合作伙伴提供标签和排名信息。分销商已经有了一个系统，但这个新的合作伙伴——该地区最大的移动超市——希望自己的应用程序有自己的数据和格式。它想尽快得到这项服务。让我们看看葡萄酒经销商如何从使用 Red Hat 集成中获益。

[![Diagram of architecture](img/b7f28d1e5558e59934b27c09785f9b53.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/imageedit_1_6138086535.png)

快速浏览下面的视频，在视频中，我展示了如何通过五个简单的步骤快速开发和应用完整的 API 生命周期

[https://www.youtube.com/embed/4BfxZDIjeTo?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/4BfxZDIjeTo?autoplay=0&start=0&rel=0)

## 幕后是怎么回事？

让这一切发生看起来很容易，但让我们看看幕后发生了什么。

[红帽 Fuse Online](https://developers.redhat.com/products/fuse/overview/) 、低代码集成平台以及其他资源都需要在 [OpenShift](http://openshift.com/) 之上运行，以实现标准化。数据源来自独立名称空间中的数据库。为了将来自内部数据源的数据公开给 API，citizen 开发人员使用 Apicurio 定义 API，API curio 生成 OpenAPI 标准文档，以后在实现阶段会引用和使用这些文档。

实现本身相当快；它只是将输入参数映射到 select SQL，并将输出映射到导出的结果。Fuse online 将获取您的集成配置，并将其存储在内部数据库中。

与此同时，源到图像(S2I)过程将启动，将应用程序构建到一个[容器](https://developers.redhat.com/blog/category/containers/)中，然后作为一个 [Kubernetes](https://developers.redhat.com/blog/category/kubernetes/) pod 部署在云平台上。构建之后是部署。在部署过程中，会创建一个 Kubernetes 服务，该服务充当运行 pod 的后端的内部代理/负载平衡器。

为了保护 API 免受外部攻击、不必要的访问和任何其他安全问题，[Red Hat 3 scale API Management](https://www.redhat.com/en/technologies/jboss-middleware/3scale)用于通过选择保护方法来保护 API 端点:通过简单 API 密钥、通过 Auth 2.0 或通过其他 SSO 方法进行身份验证。这将设置访问 API 的路径和策略，并确定允许谁访问 API。但是 3scale API 管理如何知道在哪里寻找内部端点呢？

*   聪明的发现。3scale API 管理自动检测标记为“真”的 Kubernetes 服务请注意，您需要允许 3scale API 管理拥有集群范围可见性等权限，或者授权 view 查看集群上的服务。
*   要允许 Red Hat Fuse Online 为您标记服务，以便它可以被 3scale 发现，您需要将*CONTROLLER EXPOSE VIA 3 scale*设置为 true。
*   此后，API 将对所有客户端可用、受到保护并受到保护。

这就是在整个 API 生命周期中幕后发生的事情。看看这个视频，它带你了解幕后的故事。

[https://www.youtube.com/embed/ejJIOKF_jTY?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/ejJIOKF_jTY?autoplay=0&start=0&rel=0)

希望你喜欢使用红帽集成！**