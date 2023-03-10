# 存储服务的集成(第 6 部分)

> 原文：<https://developers.redhat.com/blog/2019/01/18/integration-of-storage-services-part-6>

在本系列的[第 5 部分中，我们研究了决定您的集成如何成为转变客户体验的关键的细节。](https://developers.redhat.com/blog/2019/01/04/integration-of-container-platform-essentials-part-5/)

它从展示我如何通过研究成功的客户组合解决方案作为通用架构蓝图的基础来处理用例的过程开始。现在是时候涵盖各种蓝图细节了。

本文涵盖了蓝图中的最后几个要素，*存储服务，*它们是通用架构概述的基础。

## 建筑细节

如前所述，这里涉及的架构细节基于使用开源技术的真实客户集成解决方案。这里呈现的元素是我在通用架构蓝图中识别和收集的*通用通用架构元素*。我的意图是提供一个蓝图，提供指导，而不是深入的技术细节。

[![Storage services](img/dd3b4aec5691d1ff16bb3f8be4c725cd.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/Screenshot-2018-12-21-at-10.47.57.png)

请注意，我们涵盖了所呈现的视觉表示，但预计它们会随着时间的推移而在视觉上发生变化。在这个建筑蓝图中，有许多方法来表示每个元素，但是我选择了图标、文本和颜色，我希望它们能让你更容易理解。欢迎在这篇文章的底部发表评论，或者[直接联系我](https://www.schabell.org/p/contact.html)提供您的反馈。

现在让我们来看看这个架构中的细节，并概述一下我在研究中发现的元素。

## 储存；储备

虽然每个组织都需要并且肯定已经选择了本文中描述的一种或多种存储服务，但为了完整起见，我还是给出了我在研究中发现的最常见的选择。

我研究的每个组织都拥有的基本传统解决方案是*虚拟块存储(VBS)* 解决方案。它可以在您的数据中心，在您的开发人员机器上，或者由几乎任何云提供商托管。它提供固定大小的原始存储容量，并且必须具有一致的 I/O 性能和低延迟连接。

[![Virtual block storage](img/ae733f58ac842d89b64a43f4c425d8e6.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/Screenshot-2018-12-21-at-10.37.52.png)

当文件和数据集变得非常大时，*基于对象的存储(OBS)* 成为首选服务。它可以在内部提供，或者作为大多数云提供商托管的服务提供，以确保您可以针对您的特定用例利用您选择的持久性。

[![Object-based storage](img/bc0b1ca754ba7670ab18f1ebe1f7d995.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/Screenshot-2018-12-21-at-10.38.10.png)

对于基于容器的应用和服务，持久性是通过*容器本地存储(CNS)* 解决方案实现的。如前所述，我进行的所有研究的中心是明显倾向于使用[容器](https://developers.redhat.com/blog/category/containers/)平台进行应用和[微服务](https://developers.redhat.com/blog/category/microservices/)。

[![Container-native storage](img/2d212758bb662e3fbd665fa099170aeb.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/Screenshot-2018-12-21-at-10.38.02.png)

对这些基于容器的元素的存储需求导致组织寻找 CNS 解决方案。这种解决方案是容器平台的原生解决方案，提供了开发人员和架构师为 omnichannel 构建集成解决方案所需的性能和易用性。

我们与这些存储服务的通用集成的一个关键在于之前讨论的*集成数据微服务*，它使所有形式的存储服务在您的架构中都可用。这些细节并不能说明一切，但是应该给你在你自己的架构环境中开始工作所需要的指导。

## 下一步是什么

本概述涵盖了构成我们全渠道客户体验用例架构蓝图的容器平台元素。

全渠道客户体验组合架构蓝图系列概述可在此处找到:

1.  [第 1 部分:整合如何成为客户体验的关键](https://developers.redhat.com/blog/2018/11/28/integration-is-key-to-customer-experience/)
2.  [第 2 部分:现代集成架构的通用架构元素](https://developers.redhat.com/blog/2018/11/30/common-architectural-elements-for-modern-integration-architectures/)
3.  [第三部分:外部应用细节的整合](https://developers.redhat.com/blog/2018/12/14/integration-of-external-application-details-part-3/)
4.  [第四部分:API 管理细节的整合](https://developers.redhat.com/blog/2018/12/20/integration-of-api-management-details-part-4/)
5.  [第五部分:集装箱平台要素集成](https://developers.redhat.com/blog/2019/01/04/integration-of-container-platform-essentials-part-5/)
6.  第 6 部分:存储服务的集成(本文)
7.  第 7 部分:应用程序集成细节
8.  第 8 部分:剖析几种特定的应用程序集成架构

通过上面的链接之一来补上你错过的任何文章。

在本系列的下一篇文章中，我们将开始研究特定的集成架构，这些架构将我们作为全渠道客户体验架构中特定案例的一部分所讨论的所有元素联系在一起。

*Last updated: September 3, 2019*