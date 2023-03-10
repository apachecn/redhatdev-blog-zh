# 移动集成的集成蓝图示例(第 8 部分)

> 原文：<https://developers.redhat.com/blog/2019/05/28/integration-blueprint-example-for-mobile-integration-part-8>

在本系列的[第 7 部分中，我们探讨了决定您的集成如何成为转变客户体验的关键的细节。它从展示我如何通过研究成功的客户组合解决方案作为通用架构蓝图的基础来处理用例的过程开始。让我们继续看看这些蓝图如何解决特定集成用例的更具体的例子。](https://developers.redhat.com/blog/2019/04/19/integration-blueprint-example-for-process-automation-part-7/)

本文将带您浏览一个示例集成场景，展示扩展前面讨论的细节如何为您自己的集成场景提供蓝图。

### 蓝图场景

如前所述，这里涉及的架构细节基于使用开源技术的真实客户集成解决方案。这里展示的示例场景是在研究客户解决方案时发现的*通用通用蓝图*。我的意图是提供一个蓝图，提供指导，而不是深入的技术细节。

本节涵盖了所呈现的视觉表示，但是预计它们会随着时间的推移而在视觉上发生变化。在这个建筑蓝图中，有许多方法来表示每个元素，但是我选择了图标、文本和颜色，我希望它们能让你更容易理解。请随意在本文底部发表评论，或者直接联系我提供您的反馈。

现在，让我们看看这个蓝图中的细节，并概述解决方案。

[![omnichannel customer experience](img/46289dbe7bcc17a6eda8c0815fb281d2.png)](https://4.bp.blogspot.com/-prz_cZSYcIM/XH5wogfQkUI/AAAAAAAAtgE/P7suUlWINegFf-_kZfs9r__McGPkZpXzgCLcBGAs/s1600/Screenshot%2B2019-03-05%2Bat%2B13.50.19.png)

### 移动集成

标题为*示例:移动集成*的图中所示的示例蓝图概述了如何将移动集成到您的架构中。在这个例子中，从顶部开始，使用一个移动设备通过 API 网关连接到您的服务。它利用了一组提供*前端*功能的微服务。这些*前端微服务*通过*集成微服务*从各种组织后端系统收集数据和信息。

另一组微服务展示了*移动服务*(在[的上一篇文章](https://developers.redhat.com/blog/2019/04/19/integration-blueprint-example-for-process-automation-part-7/)中以通用术语讨论过)如何提供特定于移动的服务，比如推送通知、同步等等。这显示了客户如何使用微服务等通用架构集成概念来解决他们对特定移动平台解决方案的依赖。

图中没有显示存储服务，相反，容器本地存储用于维护应用程序数据时的各种移动服务、移动分析以及任何其他持久性需求。这里有意识地努力保持这个蓝图例子尽可能的简洁；因此，各种潜在的*后端系统*的集成已经简化为单个代表性的盒子。

### 下一步是什么

本概述涵盖了全渠道客户体验用例流程集成的首个架构蓝图示例。

全渠道客户体验组合架构蓝图系列概述可在此处找到:

1.  [第 1 部分:整合如何成为客户体验的关键](https://developers.redhat.com/blog/2018/11/28/integration-is-key-to-customer-experience/)
2.  [第 2 部分:现代集成架构的通用架构元素](https://developers.redhat.com/blog/2018/11/30/common-architectural-elements-for-modern-integration-architectures/)
3.  [第三部分:外部应用细节的整合](https://developers.redhat.com/blog/2018/12/14/integration-of-external-application-details-part-3/)
4.  [第四部分:API 管理细节的整合](https://developers.redhat.com/blog/2018/12/20/integration-of-api-management-details-part-4/)
5.  [第五部分:集装箱平台要素集成](https://developers.redhat.com/blog/2019/01/04/integration-of-container-platform-essentials-part-5/)
6.  [第 6 部分:存储服务集成](https://developers.redhat.com/blog/2019/01/18/integration-of-storage-services-part-6/)
7.  [第 7 部分:过程自动化集成蓝图示例](https://developers.redhat.com/blog/2019/04/19/integration-blueprint-example-for-process-automation-part-7/)
8.  第 8 部分:移动集成的集成蓝图示例(本文)
9.  更多集成蓝图示例

通过上面的链接之一来补上你错过的任何文章。

在本系列的下一篇文章中，我们将探讨更具体的集成架构蓝图，这些蓝图将我们作为全渠道客户体验架构中特定案例的一部分所讨论的所有元素联系在一起。

*Last updated: May 23, 2019*