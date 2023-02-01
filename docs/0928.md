# 宣布 Kubernetes-Red Hat AMQ 在线本地自助信息

> 原文：<https://developers.redhat.com/blog/2019/02/11/red-hat-amq-online-self-service-messaging-kubernetes>

[微服务](https://developers.redhat.com/topics/microservices/)架构正在接管各地的软件开发讨论。越来越多的公司正在适应开发微服务作为他们新系统的核心。然而，当超越“微服务 101”谷歌教程时，所需的服务通信变得越来越复杂。可伸缩的分布式系统、容器本地微服务和无服务器功能受益于去耦合通信来访问其他相关服务。异步(非阻塞)直接或代理交互通常被称为*消息传递*。

管理和设置用于开发的消息传递基础设施组件通常是一项长期的先决任务，需要项目日历上的几天时间。需要队列或主题？至少等几个星期。与您的基础架构运营团队一起筹集资金，喝一大杯咖啡，祈祷他们有时间调配。当您的开发团队采用敏捷方法时，等待基础设施的时间是不可接受的。

那么，走向公共云是答案吗？也许吧，但是你需要回答这样的问题:

*   我们将有哪些可用性？
*   谁会买单？
*   谁将管理用户？
*   我们可以转向另一家提供商吗？
*   进入生产阶段后，我们还能回到数据中心吗？

在贵组织自己的基础架构中拥有公共云自助消息传递基础架构的优势，这难道不是很棒吗？好吧，我们有好消息，红帽[于 2019 年 1 月 30 日](https://access.redhat.com/announcements/3869211)宣布，其最新[红帽 AMQ](https://www.redhat.com/en/technologies/jboss-middleware/amq) 消息套件组件:红帽 AMQ 在线全面上市。

## 红帽 AMQ 在线

红帽 AMQ 在线将红帽坚如磐石的 AMQ 产品的最佳功能与[红帽 OpenShift](https://www.openshift.com/) 的云可访问性相结合。Red Hat Integration 解决方案的这一新特性允许服务管理员部署和管理消息传递基础设施，同时用户团队(租户)可以请求消息传递资源，两者都使用[Kubernetes](https://developers.redhat.com/topics/kubernetes/)——本地 API 和工具。

Red Hat AMQ 在线让管理员能够在云中或内部配置云原生多租户消息服务。它允许开发人员随时随地提供消息，从易于使用的浏览器控制台为自己服务。

他们可以创建不同的消息代理和路由器配置(称为“计划”),供租户使用，而不是等待运营团队调配环境。基础设施配置可以被命名为——例如，*小型*、*中型*和*大型*——反映为每个系综提供的资源量。小型计划可能使用一个只有 2GB Java 堆和几 GB 存储的小型代理，而超大型配置涉及部署一个大型可伸缩集群，该集群拥有大量防护资源。

从租户的角度来看，所有之前的复杂性都隐藏在计划的背后。这使得租户只需关注其消息传递目的地的设计，在一对一或一对多通信以及是否需要持久性之间进行选择。有经验的用户可以直接在 OpenShift 上创建 Kubernetes 定制资源定义来请求消息传递地址，或者他们可以访问 web 控制台并拥有一个图形用户界面来创建、管理和监控他们的地址。

Red Hat AMQ 在线通过与 Red Hat 单点登录(SSO)的集成，包括内置的客户端认证和授权以及身份管理。当使用 Red Hat SSO 时，组织可以在本地提供他们的用户，或者他们可以连接到他们当前的 LDAP 或 Active Directory 服务以使用现有的提供者。

### 下一步是什么？

对红帽 AMQ 在线感兴趣？观看我的同事克里斯蒂娜·林的视频，或者关注我们的[“你好，世界！”指导](https://developers.redhat.com/products/amq/hello-world/#fndtn-amq-online)开始使用真正的自助信息服务。

[https://player.vimeo.com/video/315481089?autoplay=0](https://player.vimeo.com/video/315481089?autoplay=0)

## 资源

*   要了解更多关于 Red Hat 集成的信息，请参阅 Christina Lin 的文章:[使用 Red Hat 集成轻松创建完整的 API 生命周期(第 1 部分)](https://developers.redhat.com/blog/2019/02/11/red-hat-integration-effortless-api-creation/)
*   [试试红帽 AMQ 简单的“你好，世界”](https://developers.redhat.com/products/amq/hello-world/#fndtn-amq-online)

*Last updated: September 3, 2019*