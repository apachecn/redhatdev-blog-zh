# 为什么是 Kubernetes 原生而不是云原生？

> 原文：<https://developers.redhat.com/blog/2020/04/08/why-kubernetes-native-instead-of-cloud-native>

首先，我不是指 *Knative* ，基于 Kubernetes 的现代无服务器工作负载平台，而是 *Kubernetes native* 。在本文中，我将解释什么是 Kubernetes native，它意味着什么，以及为什么它对开发人员和企业很重要。在我们深入研究 Kubernetes 原生应用程序之前，我将回顾一下什么是云原生应用程序开发，以及它如何引导我们进行 Kubernetes 原生应用程序开发。

## 云原生:概述

我们都听说过用于开发应用程序和服务的 *云原生* 方法，更有甚者，自 2015 年 成立 [云原生计算基金会(CNCF)以来，这个术语是从何而来的呢？*云原生*这个术语最早是由 Bill Wilder 在他的著作](https://www.infoq.com/news/2015/07/kubernetes-v1-released/) [*云架构模式*](http://shop.oreilly.com/product/0636920023777.do) (O'Reilly Media，2012) 中使用的。根据 Wilder 的说法，云原生应用是任何被设计为充分利用云平台的应用。这些应用:

*   使用云平台服务。
*   水平缩放。
*   使用主动和被动操作自动扩展。
*   在不降级的情况下处理节点和瞬时故障。
*   以松散耦合架构中的非阻塞异步通信为特色。

与云原生技术相关的是 [十二要素应用](https://12factor.net/) ，一套用于构建应用的模式(或方法)即服务。云架构模式通常被描述为开发云原生应用程序所必需的。12 因素与 Wilder 的云架构模式重叠，但是 12 因素深入到与云原生开发不特别相关的应用程序开发的细节中。它们同样适用于一般的应用程序开发以及应用程序如何与基础设施集成。

怀尔德写这本书的时候，人们对开发和部署云原生应用的兴趣正在增长。开发者有各种公共和私有平台可供选择，包括 Amazon AWS、Google Cloud、Microsoft Azure 和许多较小的云提供商。混合云部署也在那时变得越来越普遍，这带来了挑战。

**注:**关于混合云的问题并不新鲜， *[水库——当一片云不够用的时候](https://ieeexplore.ieee.org/abstract/document/5719569)* (IEEE 2011) ，涵盖了其中的几个。

## 协调混合云

作为一种架构方法，[混合云](https://www.infoworld.com/article/2683561/the-case-for-the-hybrid-cloud.html)支持将相同的应用程序部署到私有云和公共云的混合体中，通常跨越不同的云提供商。混合云提供了不依赖单一云提供商或地区的灵活性。如果一家云提供商出现网络问题，您可以将部署和流量切换到另一家提供商，以最大限度地减少对客户的影响。

不利的一面是，每个云提供商都有自己的首选机制，例如命令行界面(CLI)、发现协议和事件驱动协议等，开发人员必须使用这些机制来部署他们的应用程序。这种复杂性使得以自动化的方式向多个云提供商部署相同的应用变得不可能。

为了解决这个问题，许多编排框架开始出现。然而，没过多久， [Kubernetes](https://developers.redhat.com/topics/kubernetes) 就成为了事实上的编排标准。如今，很难找到不提供将应用程序部署到 Kubernetes 的能力的云提供商。谷歌、亚马逊和微软都提供 Kubernetes 作为编排层。

## 为库伯内特土著做好准备

对于将应用程序部署到混合云的开发人员来说，将重点从云本地转移到 Kubernetes 本地是有意义的。有一篇关于[Kubernetes-native](https://medium.com/@cloudark/towards-a-kubernetes-native-future-3e75d7eb9d42)future的精彩文章介绍了 Kubernetes-native 堆栈的含义。关键的一点是，Kubernetes-native 是 cloud-native 的一个专门化，并没有脱离 cloud native 的定义。云原生应用是为云设计的，而 *Kubernetes 原生*应用是为 Kubernetes 设计和构建的。

在讨论以 Kubernetes-native 应用程序和服务为目标的好处之前，我们需要理解所有的区别。上面我们提到了 Kubernetes native 对于基础设施管理的好处。跨云提供商使用相同的工具将应用程序部署到 Kubernetes 是关键。接下来，我们重新审视是什么让应用(或微服务、功能或部署)成为云原生的。应用程序真的是云原生的吗？应用程序可以在任何云提供商上无缝运行吗？

在云原生开发的早期，编排差异阻碍了应用成为真正的云原生应用。Kubernetes 解决了编排问题，但 Kubernetes 没有涵盖云提供商服务或活动骨干。要回答任何应用程序是否可以是云原生的，我们需要涵盖可以开发的不同类型的应用程序。

### 有状态应用程序与隔离应用程序

我们通常将应用程序分为无状态的和 T2 状态的。同样关键的是*隔离*和*耦合*的分裂。确定一个应用程序是否是云原生的会受到这两个因素的影响。当我说耦合时，我并不是指应用程序的内部代码是紧密耦合还是松散耦合。我指的是一个应用程序是否完全依赖于外部应用程序和服务。不与自身之外的任何事物交互的应用程序是*隔离的*。使用一个或多个外部服务的应用程序是*耦合的*。

考虑图 1，它在不同的轴上显示了状态和隔离。

[![A graph showing four application types on an axis: Isolated and Stateful; Coupled and Stateful; Isolated and Stateless; and Coupled and Stateless.](img/f42e4ecff6f49462531b4539da759113.png "AppTypes")](/sites/default/files/blog/2020/02/AppTypes.png)Figure 1\. Four variations of isolation and statefulness.">

请注意，*独立的和有状态的*应用程序需要一个状态来运行，但是状态保留在应用程序本身中，而不是存储在外部。没有多少应用程序属于这一类，因为它们不能很好地处理重启。相比之下，c *耦合和无状态*应用程序是无状态的，但是必须与外部服务耦合来响应请求。这种类型的应用程序更常见，因为它包括与数据库、消息代理和 Apache Kafka 的交互。

任何被隔离的应用程序，无论是无状态的还是有状态的，都可以遵循云原生原则。这样的应用程序可以在混合云中为不同的提供商打包。需要耦合到外部服务的应用程序也可以被认为是云本地的，但是这样的应用程序可以部署到混合云吗？这个问题的答案是大家最喜欢的:看情况。

### 两种服务的故事

如果外部服务是在 Kubernetes 上提供给应用程序使用的数据库，那么答案是肯定的:应用程序独立于云提供商，可以部署到混合云。这种类型的体系结构给运营带来了更大的负担。需要在 Kubernetes 环境中安装和管理数据库或其他数据存储。那远非理想。

另一种选择是使用云提供商提供数据库服务。这样做会减轻运营负担，但我们会被特定的云服务提供商所束缚。在这种情况下，我们将无法跨多个提供商将应用程序部署到混合云中。

## 面向混合云服务的 Kubernetes 供应

现在让我们考虑一个真实世界的例子。如果一个应用程序被编写为使用亚马逊 S3 进行数据存储，那么开发团队将使用定制的亚马逊 S3 API 与它进行交互。如果我们决定将应用程序部署到 Google Cloud，我们不能期望它在没有针对 Google Cloud Storage APIs 进行修改的情况下与 Google Cloud Storage 一起工作。这两个版本可能都是云原生的，但它们不是云提供商不可知的。这个因素给完全支持混合云模型带来了问题。

Kubernetes 社区正在研究一个对象桶存储的解决方案，但是这个补丁并没有解决 API 可移植性的更普遍的问题。即使 Kubernetes 有办法像亚马逊 S3 一样提供桶型存储挂载，开发人员仍然需要利用亚马逊 S3 API 与他们进行交互。开发人员仍然需要在他们的应用程序中使用特定于云提供商的 API。

企业可以通过编写不同服务的通用包装器来保护他们的业务代码。然而，大多数企业不愿意花费宝贵的资源来开发服务的包装器。更重要的是，服务包装器仍然需要根据每个云提供商环境的不同依赖关系重新包装应用程序。

从开发人员的角度来看，缺少的部分是一个应用程序 API，它被充分抽象，可以跨云提供商使用，提供相同类型的服务。

## Kubernetes:新的应用服务器？

尽管 Kubernetes 正在不断发展，以便为开发人员需要的服务提供环境，但我们仍然需要一种方法来填补应用程序的业务逻辑和与这些服务的交互之间的空白。这也是我之前认为应用服务器和框架在 Kubernetes 上仍然有作用的一个原因。

采用 Kubernetes-native 环境确保了混合云的真正可移植性。然而，我们还需要一个 Kubernetes-native 框架来为应用程序无缝集成 Kubernetes 及其服务提供“粘合剂”。没有应用程序的可移植性，混合云只能带来环境方面的好处。

这个框架就是 Quarkus。

## quar kus:Kubernetes-native 框架

Quarkus 是一个 Kubernetes-native Java 框架，可用于[促进混合云环境的应用程序可移植性](https://developers.redhat.com/blog/2019/03/07/quarkus-next-generation-kubernetes-native-java-framework/)。例如，Quarkus 允许您从应用程序内部生成部署到 Kubernetes 的描述符。使用`application.properties`文件中的配置值也很容易调整。使用 health 扩展，生成的描述符包含就绪性和活性探测，这对 Kubernetes 了解您的应用程序是否健康至关重要。这些特性通过在 Kubernetes 中创建应用程序部署并了解与 Kubernetes pod 生命周期处理集成的正确方法，简化了开发人员的工作。

Quarkus 还结合了构建时优化和反应模式来减少应用程序的内存消耗。这个因素对潜在的部署密度有直接影响。能够在相同的基础设施上运行更多的吊舱是 Quarkus 的一个重要方面(阅读更多关于这方面的内容，请参见[汉莎技术公司](https://quarkus.io/blog/aviatar-experiences-significant-savings/)、[亚洲航空公司集团](https://quarkus.io/blog/asiakastieto-chooses-quarkus-for-microservices/)和[沃达丰希腊](https://quarkus.io/blog/vodafone-greece-replaces-spring-boot/))。在公共云上，该特性允许使用较小的实例来运行应用程序，从而降低成本。

对于 Quarkus 来说，以及对于我们最大限度地实现 Kubernetes-原生应用程序开发的目标来说，现在还为时尚早。然而，我们在短时间内取得了很大的进步，我们致力于确保 Quarkus 为所有开发者提供最好的 Kubernetes-native 体验。

## 结论

Kubernetes-native 是 cloud-native 的专门化，这意味着它们之间有许多相似之处。主要区别在于云提供商的可移植性。

为什么这种区别很重要？ 充分利用混合云并使用多个云提供商要求应用程序可部署到任何云提供商。如果没有这样的功能，您将被束缚在一个单一的云提供商中，并依赖他们 100%的时间运行。 享受混合云的好处需要开发人员采用 Kubernetes-native 应用程序开发，不仅针对环境，也针对应用程序。 Kubernetes native 是云可移植性问题的解决方案。[quar kus](https://quarkus.io)是应用程序和 Kubernetes 之间的管道，促进了混合云的可移植性。通过正在进行的为所提供的各种服务提供抽象的工作， Quarkus 将帮助您实现这一目标！

*Last updated: June 29, 2020*