# API 管理细节的集成(第 4 部分)

> 原文：<https://developers.redhat.com/blog/2018/12/20/integration-of-api-management-details-part-4>

在本系列的[第 3 部分，我们开始深入探讨决定您的集成如何成为转变客户体验的关键的细节。](https://developers.redhat.com/blog/2018/12/14/integration-of-external-application-details-part-3/)

它从展示我如何通过研究成功的客户组合解决方案作为通用架构蓝图的基础来处理用例的过程开始。现在是时候涵盖各种蓝图细节了。

本文将带您深入了解通用架构概述的特定元素( *API 管理和反向代理)*。

## 建筑细节

如前所述，这里涉及的架构细节基于使用开源技术的真实客户集成解决方案。这里呈现的元素是我在通用架构蓝图中识别和收集的*通用通用架构元素*。我的意图是提供一个蓝图，提供指导，而不是深入的技术细节。

[![Generic common architectural elements](img/2ba89ccfbbb3c5b36a6e5f6dd2976b69.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/12/Screenshot-2018-12-07-at-13.23.05.png)

本节涵盖了所呈现的视觉表示，但是预计它们会随着时间的推移而在视觉上发生变化。在这个建筑蓝图中，有许多方法来表示每个元素，但是我选择了图标、文本和颜色，我希望它们能让你更容易理解。欢迎在这篇文章的底部发表评论，或者[直接联系我](https://www.schabell.org/p/contact.html)提供您的反馈。

现在让我们来看看这个架构中的细节，并概述一下我在研究中发现的元素。

## API 管理

进入组织的网关分为管理 API 访问和隐藏组织中访问服务背后的实际情况。我确定的第一个元素是用于处理 API 网关活动的管理平台。

[![Management platform for handling API gateway activities](img/ad2e3287400c9d2a165c00d5ee9753c1.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/12/Screenshot-2018-12-07-at-14.44.05.png)

*API 管理*指的是如何提供对组织服务的访问。这是内部和外部访问服务的关键路径。

研究客户组合解决方案发现，API 管理提供了对服务接口、应用和其他集成[微服务](https://developers.redhat.com/blog/category/microservices/)的访问。它提供了可扩展性、可靠性和接口使用指标，供客户在运营监控过程中进行评估。

在通用架构蓝图中，它管理以下接口:

*   前端微服务(提供对内部集成微服务的访问)
*   流程门面微服务(提供对自动化集成流程的访问)
*   其他应用(提供对聚合微服务或其他内部应用的访问)

外部方通过接口最终访问内部服务的过程中，有一部分涉及到隐藏特定的网络细节。为此，我们将检查*反向代理*的详细信息。

## 反向代理

这涵盖了研究中发现的各种解决方案，但所有解决方案都提供相同的功能。

[![Reverse proxies](img/fc403d7cfb4e2abf1e2083fe0ba31268.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/12/Screenshot-2018-12-07-at-14.44.14.png)

基本的安全性是通过这些代理实现的，因为它们是根据第三方的请求进行操作的。通过为客户检索请求的资源，所有外部方都无法实际访问内部网络。

代表客户的交互为他们提供了对以下微服务的访问:

*   前端微服务(提供对内部集成微服务的访问)
*   流程门面微服务(提供对自动化集成流程的访问)
*   其他应用(提供对聚合微服务或其他内部应用的访问)

这些细节并不能说明一切，但是应该可以给你一些指导，帮助你在自己的架构环境中开始工作。

## 下一步是什么

本概述涵盖了构成我们全渠道客户体验用例架构蓝图的 API 和代理元素。

全渠道客户体验组合架构蓝图系列概述可在此处找到:

*   [第 1 部分:整合如何成为客户体验的关键](https://developers.redhat.com/blog/2018/11/28/integration-is-key-to-customer-experience/)
*   [第 2 部分:现代集成架构的通用架构元素](https://developers.redhat.com/blog/2018/11/30/common-architectural-elements-for-modern-integration-architectures/)
*   [第三部分:外部应用细节的整合](https://developers.redhat.com/blog/2018/12/14/integration-of-external-application-details-part-3/)
*   [第 4 部分:API 管理细节的集成(本文)](https://developers.redhat.com/blog/2018/12/20/integration-of-api-management-details-part-4/)
*   [第五部分:集装箱平台要素集成](https://developers.redhat.com/blog/2019/01/04/integration-of-container-platform-essentials-part-5/)
*   [第 6 部分:存储服务集成](https://developers.redhat.com/blog/2019/01/18/integration-of-storage-services-part-6/)
*   第 7 部分:应用程序集成细节
*   第 8 部分:剖析几种特定的应用程序集成架构

通过上面的链接之一来补上你错过的任何文章。

在本系列的下一篇文章中，我们来看看全渠道客户体验架构中特定元素的细节。

*Last updated: September 3, 2019*