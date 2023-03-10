# 诅咒你的选择！Kubernetes 还是应用服务器？(第三部分)

> 原文：<https://developers.redhat.com/blog/2019/01/30/curse-you-choices-kubernetes-or-application-servers-part-3>

这是关于 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 是否是新的应用服务器系列的最后一集。在这一部分中，我将讨论在传统应用服务器 Kubernetes 和替代服务器之间的选择。这种替代方案可以称为[、【刚好够用的应用服务器】、](https://antoniogoncalves.org/2016/07/13/just-enough-app-server-with-wildfly-swarm/)，比如 Thorntail。在[红帽开发者博客](https://developers.redhat.com/blog/)上有几篇关于 Thorntail(之前称为 Wildfly Swarm)的文章。关于 Thorntail 的一个很好的介绍在 [2.2 产品发布](https://developers.redhat.com/blog/2018/10/17/announcing-thorntail-2-2-general-availability/)中。

## 快速回顾

我们已经在 [第一部分](https://developers.redhat.com/blog/2018/09/05/kubernetes-new-operating-environment) 中讨论了 Kubernetes 环境中容器的优点和缺点。特别是容器有助于打包部署，但它们不是万能的。仍然有可能创建包含不良内容的容器。

几十年来，应用服务器一直是中间件的关键部分。它们提供与关键服务的集成，例如开发复杂应用程序时所需的安全和事务**。在[第 2 部分](https://developers.redhat.com/blog/2018/10/02/are-app-servers-dead-in-the-age-of-kubernetes-part-2/)中，我们概述了应用服务器的用例，以及它们今天的持续相关性。应用服务器解决的问题仍然存在。Kubernetes 和 Linux 容器并没有消除对安全性或事务的需求。**

 **## 适合工作的合适工具！

虽然这是老生常谈，但在这个例子中也是完全准确的。我们当然被选择宠坏了:

*   库伯内特斯
*   传统应用服务器，如 WildFly / [红帽 JBoss 企业应用服务器](https://developers.redhat.com/products/eap/overview/)
*   [刚好够用的应用服务器](https://antoniogoncalves.org/2016/07/13/just-enough-app-server-with-wildfly-swarm/)，比如 Thorntail

对于许多不需要应用服务器提供服务的应用来说，Kubernetes 本身就很棒。另一方面，有些应用程序确实需要这些服务或框架。这些应用程序需要 WildFly 和 Thorntail 提供的功能。无论他们最终的操作环境是 Kubernetes 还是裸机。

## 那会给我们带来什么？

作为一个整体，IT 行业经常在各种观点之间摇摆，就好像它们是截然相反的。更糟糕的是，那一个想法是 **只** 解决一切！对于当今的大多数企业来说，这既不现实也不实用。

绿地开发非常罕见。而频繁地在“解决方案”之间切换是完全不切实际的。随着时间和成本的转换，新“解决方案”也有不同的开发人员技能。大多数企业没有资源每隔几年就对他们的开发人员进行新技术的再培训。坚持通用的编程模型是长期成功的关键。

自从微服务第一次崭露头角，我们就意识到并不是所有的东西都需要符合一个模式。一整块石头足够好吗？几千个分布式微服务是不是太复杂了？也许一个简单的 Java 应用程序不需要应用服务器，只需要 JVM 作为容器。尽管您可能不会将 JVM 视为一个容器。它提供了与 Linux 容器相似的隔离，但是很久以前就有了。

这些都是很好的问题。在为应用程序选择操作环境之前，应该回答其中的一些或全部问题。

归根结底，这是关于了解您的应用需求。然后使用该信息来确定最佳的应用服务器或容器、框架和环境组合，以使其发挥作用。有时是 Kubernetes，有时是应用服务器，无论是传统的还是“刚刚好”。应用程序的目标和现有资源的技能组合是决定适合度的关键因素。

## 结论

Kubernetes 是新的应用服务器吗？是也不是。对某些用途来说会是。对其他人来说不会。

有没有新的应用服务器，至少对于那些处理 Java 的人来说是这样的？不是严格意义上的。JVM 容器是新的“应用服务器”，但肯定不是新的。随着可执行 jar(胖 jar)和足够多的应用服务器的出现，JVM 又开始增长了。

无论是应用服务器、胖罐子、空罐子、分层容器映像，还是 Java 未来可能出现的任何东西。JVM 是新的容器选择，以 Kubernetes 作为操作环境。为应用程序提供选择应用服务器或使用普通 Java 的灵活性。JVM 容器是应用程序间的共同点。

或许“集装箱”真的统治了世界？！

*Last updated: October 14, 2021***