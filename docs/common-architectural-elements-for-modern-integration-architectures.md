# 现代集成架构的通用架构元素(第 2 部分)

> 原文：<https://developers.redhat.com/blog/2018/11/30/common-architectural-elements-for-modern-integration-architectures>

在本系列的[第 1 部分中，我们探索了一个关于集成的用例，集成是转变客户体验的关键。](https://developers.redhat.com/blog/2018/11/28/integration-is-key-to-customer-experience/)

我展示了我如何处理用例，以及我如何使用成功的客户组合解决方案作为研究通用架构蓝图的基础。剩下要介绍的唯一一件事就是引导您浏览蓝图细节的顺序。

本文是本系列的第 2 部分，从最顶层开始真正的旅程，从一个通用架构开始，我们将逐一讨论常见的架构元素。

## 从特定到一般

在深入研究公共元素之前，最好理解这并不是对所有可能的集成解决方案都适用。这是我在多个客户实现中发现的已识别元素的集合。这里呈现的元素是我已经确定并收集到通用架构蓝图中的*通用架构元素*。

我的意图是提供一个蓝图，提供指导，而不是深入的技术细节。您足够聪明，能够为自己的体系结构找到布线集成点。在适用的情况下，您能够将您过去承诺的技术和组件纳入其中。在这里，我的工作是描述架构蓝图的一般组件，并使用可视化图表概述一些特定的案例，以便您能够从集成项目的开始就做出正确的决策。

另一个挑战是如何在视觉上表现建筑蓝图。有很多方法来表现每一个元素，但是我选择了一些图标、文字和颜色，我希望它们能让你更容易理解。欢迎在这篇文章的底部发表评论，或者[直接联系我](https://www.schabell.org/p/contact.html)提供您的反馈。

现在让我们快速浏览一下通用架构，并概述一下我在研究中发现的常见元素。

## 外部应用

从图的顶部开始，这决不是地理上的必然，有两个元素表示与架构交互的外部应用程序。通过提炼客户解决方案研究中发现的各种应用，我得出了两个常见的表述:

[![Common architectural elements for external application deployments](img/e603106e32352cb18e0a10ca4be09fe4.png)](https://1.bp.blogspot.com/-Iziyw9LYfEs/W-yPjBdM3OI/AAAAAAAAtR4/BzwK4cJxq6UbusLUs3DjzzqZcmqhcpbmQCLcBGAs/s1600/Screenshot%2B2018-11-14%2Bat%2B22.02.50.png)

第一个是*移动应用，*基本上涵盖了客户用来与公司直接互动的一切。这可以是由公司自己部署的移动应用程序，也可以是由第三方组织开发的与所提供的服务进行交互的移动应用程序。

第二个是 *web 应用程序，*一个广泛的元素，包含公司或任何第三方组织部署的所有其他类型的应用程序、网站和/或服务，以与提供的服务进行交互。

## API 网关和代理

在研究的每个客户解决方案中都可以找到通用架构中的这些元素。他们的名字被提及，由一个*应用程序编程接口(API)网关*组成，该网关在调用内部客户解决方案服务时管理来自外部应用程序的访问。

显示的代理是一个*反向代理*，这是一个标准解决方案，通过隐藏内部地址在调用内部服务的外部应用程序之间提供一个安全层。

[![Common architectural elements are API's and proxies](img/20c87a428eb3cdac15e27a186af83486.png)](https://4.bp.blogspot.com/-FDF3y0ULHPY/W-yT21e0HiI/AAAAAAAAtSI/3Erw7wbcSqc4dae7_r5aQPga_-TwRZB_wCLcBGAs/s1600/Screenshot%2B2018-11-14%2Bat%2B22.29.52.png)

## 集装箱平台

毫无疑问，每个参与全渠道集成以改善客户体验的组织都已经看到了[容器](https://developers.redhat.com/blog/category/containers/)的价值以及容器平台的使用。容器平台为开发人员和操作人员提供了一个一致的环境来管理服务、应用程序、集成点、流程集成和安全性。

[![Common architectural element is a container platform](img/f50017f1a9a2dd228765d92469e0bbcc.png)](https://1.bp.blogspot.com/-bPVroYZKt4o/W-yVdB48QwI/AAAAAAAAtSY/EiH9T_e8wLcF7wnCcQm7tkG1gldhS7GZwCLcBGAs/s1600/Screenshot%2B2018-11-14%2Bat%2B22.31.25.png)

这也是确保您可以在混合多云环境中统一利用相同容器基础设施的一种方式。它可以防止您被束缚在任何私有或云基础架构中，因为您有一个容器平台的退出策略，该平台在您的整个架构中都是一致的。

安全方面与容器平台交织在一起，因为每个容器服务、应用程序或流程集成都可以插入到组织的身份验证和授权机制中。

## 存储服务

在客户解决方案研究中发现的存储服务种类繁多。出于这个原因，没有一个单一的公共架构元素显示在最高级别。从容器原生存储到传统块存储，什么都有。

[![Common architectural elements in storage services](img/07b98194fb161a838b23d83770e5a32e.png)](https://4.bp.blogspot.com/-MLI9T5RwmPU/W-yWuOYjzDI/AAAAAAAAtSk/uYb7F7PjbT8_64yMPJlapM2kt8yiI7atwCLcBGAs/s1600/Screenshot%2B2018-11-14%2Bat%2B22.39.45.png)

在后面的文章中，当显示更多细节时，我将重点介绍客户在将数据服务与流程和应用程序集成时选择的几个选项。

## 下一步是什么

这只是对构成 ominchannel 客户体验用例架构蓝图的常见通用元素的简短概述。

全渠道客户体验组合架构蓝图系列概述可在此处找到:

*   [第 1 部分:整合如何成为客户体验的关键](https://developers.redhat.com/blog/2018/11/28/integration-is-key-to-customer-experience/)
*   第 2 部分:现代集成架构的通用架构元素
*   [第三部分:外部应用细节的整合](https://developers.redhat.com/blog/2018/12/14/integration-of-external-application-details-part-3/)
*   [第四部分:API 管理细节的整合](https://developers.redhat.com/blog/2018/12/20/integration-of-api-management-details-part-4/)
*   [第五部分:集装箱平台要素集成](https://developers.redhat.com/blog/2019/01/04/integration-of-container-platform-essentials-part-5/)
*   [第 6 部分:存储服务集成](https://developers.redhat.com/blog/2019/01/18/integration-of-storage-services-part-6/)
*   第 7 部分:应用程序集成细节
*   第 8 部分:剖析几种特定的应用程序集成架构

通过上面的链接之一来补上你错过的任何文章。

在本系列的下一篇文章中，我们将探讨在全渠道客户体验架构中外部应用程序细节的集成。

*Last updated: January 21, 2019*