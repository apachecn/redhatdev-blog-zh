# 使用 Eclipse Vert.x 和分布式跟踪构建和理解反应式微服务

> 原文：<https://developers.redhat.com/blog/2019/05/13/building-and-understanding-reactive-microservices-using-eclipse-vert-x-and-distributed-tracing>

最近有机会在[红帽峰会 2019](https://www.redhat.com/en/summit/2019) 上发言。在我题为“使用 Jaeger 分布式跟踪的 Vert.x 应用程序开发”的会议中，我讨论了如何使用用于构建反应式应用程序的 [Eclipse Vert.x、](https://developers.redhat.com/videos/youtube/mcbdnMDERX0/)Java 虚拟机工具包来构建可伸缩的事件驱动应用程序。

多亏了许多开发工具，创建这些应用程序不再是 IT 部门最费力的任务。相反，我们现在必须理解我们的应用程序的各个部分如何一起工作来交付服务(跨开发、测试和生产环境)。这可能很困难，因为对于分布式体系结构，外部监控只能告诉您总体响应时间和调用次数，而不能提供对单个操作的洞察。此外，请求的日志条目分散在许多日志中。本文讨论了在这个问题的上下文中使用 [Eclipse Vert.x](https://vertx.io/) 、分布式跟踪和 [Jaeger](https://www.jaegertracing.io/) 。

正如[反应宣言](https://www.reactivemanifesto.org/)所定义的，反应系统是有弹性的、有弹性的、反应灵敏的，并且基于消息驱动的设计。

Eclipse Vert.x 是一个开源工具包，用于在 Java 虚拟机上构建反应式系统和流。Vert.x 是非个人化和多语言的，这使得开发人员可以自由地使用他们认为合适的工具包。Vert.x 的核心组件包括它的 actors，称为*vertices*，称为 *Event Bus* 的消息总线，以及称为 *Eventloops* 的事件调度器。

## Eclipse Vert.x 基础知识

Eclipse Vert.x 实现了 eventloops 支持的多反应器模式。在 reactor 模式中，存在一个由称为 eventloop 的线程委托给处理程序的事件流。因为 eventloop 观察事件流并调用处理程序来处理事件，所以永远不要阻塞 eventloop 是很重要的。如果事件循环没有可用的处理程序，那么事件循环必须等待；因此，我们有效地将事件循环称为阻塞的。

![](img/38c012f0797469e2c460d407f5a251af.png)

在这种模式中，多核机器上的单个 eventloop 有缺点，因为单个线程不能同时在多个 CPU 核上运行。对于使用实现 reactor 模式的技术的开发人员来说，这意味着必须用 eventloop 来管理和启动更多的进程，以提高性能。

Eclipse Vert.x 实现了一个多反应堆模式，默认情况下，每个 CPU 内核有两个 eventloops。这为使用 Vert.x 的应用程序在事件数量增加时提供了所需的响应能力。

![](img/4dd2fc47f41b71cf1a3ba59d5e3afa78.png)

在上图中，处理程序是 vertices，它是 Vert.x 中的主要参与者。vertices 在部署时被分配给一个随机事件循环。

另一个重要的概念是事件总线，这是垂直设备如何以发布-订阅的方式相互通信。Verticles 被注册到事件总线，并被给予一个监听地址。事件总线允许 verticles 伸缩，因为我们只需要指定 verticles 监听哪个地址上的事件，以及它应该将这些事件发布到哪里。

## 可观察性

Vert.x 有助于开发反应式[微服务](https://developers.redhat.com/topics/microservices/)，但是应用程序的可观察性如何呢？在分布式环境中，我们仍然可以观察到应用程序正在处理的请求，这一点很重要。例如，考虑一个电子商务应用程序。在应用程序完成处理该过程之前，单个签出请求可能被传递给数十或数百个服务；无论是在开发还是生产环境中，开发人员和支持团队都需要工具来理解和调试他们的服务中可能出现的问题。

跟踪可以提供失败的背景。分布式跟踪涉及代码检测，例如:

*   每个请求都有一个唯一的外部请求 id。
*   外部请求 id 被传递给处理请求所涉及的所有服务。
*   外部请求 id 包含在日志消息中。
*   当在集中式服务中处理外部请求时，记录关于所执行的请求和操作的信息(例如，开始时间、结束时间)。

这个代码工具是由 [OpenTracing 规范](https://opentracing.io/guides)提供的。使用分布式跟踪的核心概念，我们可以使用 OpenTracing 库来检测我们的应用程序。

应用性能管理(APM)工具，如[云原生计算基金会](https://www.cncf.io/)的 Jaeger，使用 [OpenTracing](https://opentracing.io/docs/overview/) 提供额外的功能，如用户界面，供用户进行交互，下面是使用 Jaeger 的架构图。

![](img/54e92e197e0bfe93c28ad359908b2d03.png)

应用程序的节点包含应用程序和 jaeger 客户端库。一旦跨度完成，它们将被报告给 jaeger-agent，jaeger-collector 将与数据库后端进行交互，以存储报告的轨迹，供用户查看 jaeger-ui 时查询。你可以在这里找到关于每个 Jaeger 组件[的更多细节。](https://www.jaegertracing.io/docs/1.11/architecture/)

反应式事件驱动架构提供了响应性、弹性、伸缩性和消息传递的优势。然而，随着应用程序的扩展和增长，理解甚至调试应用程序会变得很困难。本文(也是我的演讲)的目的是分享如何使用 Vert.x 来创建反应式微服务应用程序，以及分布式跟踪如何提供更好地处理此类应用程序的能力。

*本文基于 Tiffany Jachja 在 2019 年红帽峰会[上发表的“使用 Jaeger 分布式跟踪进行 Vert.x 应用开发”会议。](https://www.redhat.com/en/summit/2019)*

*Last updated: January 24, 2022*