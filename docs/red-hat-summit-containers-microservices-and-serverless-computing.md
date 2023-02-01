# 红帽峰会:容器、微服务和无服务器计算

> 原文：<https://developers.redhat.com/blog/2018/05/09/red-hat-summit-containers-microservices-and-serverless-computing>

你在一个 IT 部门。公司的其他人如何看待你？作为其代码和 API 在市场上有所作为的有价值的资产，还是作为应该尽可能减少的不可避免的祸害？容器、微服务和无服务器计算可以使您的响应更快、更灵活、更有竞争力，从而使您的组织更有效。这让你在资产项上稳稳的。

Burr Sutter 在 2018 年红帽峰会开幕式主题演讲的舞台上冲过旧金山的街道后([此处提供回放](https://www.redhat.com/en/summit/2018))，在莫斯克尼南部的一个挤满人的房子里谈论这些技术。容器被广泛接受(例如参见[Red Hat 和微软](https://www.redhat.com/en/about/press-releases/red-hat-and-microsoft-co-develop-first-red-hat-openshift-jointly-managed-service-public-cloud)的声明)，微服务作为一种使单片应用程序现代化的方法越来越受欢迎，无服务器计算正在成为一种重要的新编程模型。

我们不会在这里讨论容器(如果您还在阅读，我们假设您正在赶容器潮流)。在谈到微服务时，Burr 经历了几个步骤来实现单片应用的现代化:

*   这个整体被分解(至少在概念上)成模块。
*   这些模块中的每一个都是成为微服务的明显候选者。
*   随着这些模块被重写为微服务，微服务变成了代码片段的集合。
*   不可避免地，收藏会失去控制。它可能有几个入口，你不知道集合中的微服务是怎么互相调用的，那些微服务很多都是自己管理自己的数据。

为了解决这些问题，Istio 服务网格应运而生。当微服务被部署到 Red Hat OpenShift 集群中时，Istio 设置代理对象，使您可以管理和控制您的服务如何交互，而无需更改这些服务的任何源代码。(参见唐·申克对 Istio 博客系列的精彩介绍来全面理解这项令人敬畏的技术。)Istio 让微服务对企业安全。

伯尔指出，无服务器功能——也称为 FaaS(功能即服务)——在几个重要方面不同于微服务。在 FaaS，你的代码是:

*   托管在一个你无法控制的环境中
*   短暂的(大多数 FaaS 产品在 60 秒左右后切断你的联系)
*   建立在不同的编程模型上(首先，如果你用 Java 编写，你已经用了 20 多年的`main()`方法不再有效)
*   事件驱动和异步

您可以将服务与 FasS 捆绑在一起，无论这些服务是您内部构建的微服务还是来自云的 *X* aaS 产品。当事件发生时，您的无服务器功能调用适当的服务。例如，每当有人对云托管的数据库进行更改时，你可以使用 FaaS 调用你的一个微服务。一旦定义了函数并将其与事件相关联，FaaS 提供程序就会为您处理一切。无论您的函数是每月调用一次还是每小时调用一百万次，提供者都会确保您的代码拥有所需的资源。

最重要的是，FaaS 的崛起改变了你过去习惯的“建造还是购买”的观念。使用无服务器功能比配置和支付处理传统功能可能需要的任何工作负载所需的基础架构要便宜和简单得多。

查看 [Burr 的幻灯片](http://bit.ly/summit2018_serverless)以了解完整的故事和其他有用内容的链接，包括[的 GitHub repos、Istio 演示](http://bit.ly/istio-tutorial)和[的 FaaS 演示](http://bit.ly/faas-tutorial)。

可以在线观看环节的[视频:](https://www.youtube.com/watch?v=_0vvflSo6HA)

[https://www.youtube.com/embed/_0vvflSo6HA](https://www.youtube.com/embed/_0vvflSo6HA)

﻿*Last updated: September 3, 2019*