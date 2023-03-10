# 无服务器和 FaaS 的演变:创新带来变化

> 原文：<https://developers.redhat.com/blog/2019/03/26/serverless-faas-knative>

[无服务器](https://developers.redhat.com/topics/serverless-architecture/)和功能即服务(FaaS)是一回事吗？

不，他们不是。

等等。是的，他们是。

很沮丧，对吧？在会议上，在文章中(我现在正看着自己)，在对话中，这些术语被抛来抛去。，事情可能会令人困惑(或者，可悲的是，有时会误导)。让我们来看看无服务器和 FaaS 的一些方面，看看情况如何。

## 什么是无服务器？

这很简单:它是没有服务器的计算。这是魔法。

其实不是；不是这样的。当然，这涉及到一个服务器(或多个服务器)。无服务器的租户之一是开发人员不需要关心服务器的“东西”不需要在 RAM 使用和扩展等方面大惊小怪。简单地(现在这是一个加载的词，“简单”)部署您的代码，它的工作。

下面是来自[云本地计算基金会](https://www.cncf.io)的官方定义:

> “无服务器计算是指构建和运行不需要服务器管理的应用程序的概念。它描述了一个更细粒度的部署模型，其中应用程序被捆绑为一个或多个功能，上传到一个平台，然后执行、扩展和计费，以响应当前所需的确切需求。”

重要的一点是关于按需执行、扩展和计费的最后一句话。直到最近——也就是说，直到 [Knative](https://developers.redhat.com/topics/knative/) 出现——一个[微服务【24x7 运行，按照上面的定义，它不是无服务器的。](https://developers.redhat.com/topics/microservices/)

由于无服务器和 FaaS 传统上被用作可互换的术语，它们被认为是同一个术语。这一点是不变的:他们都没有*而不是*描述一个一直可用的典型微服务。事实上，讨论(争论？)关于 FaaS 和/或微服务的使用浮出水面，有些人甚至声称所有的微服务实际上都应该是无服务器功能。尽管对一些人来说这似乎有些极端，但这几乎已经成为一个争论点。为什么？

## 输入有价值的服务

当谷歌宣布推出基于 [Knative](https://cloud.google.com/knative/) 、 [Kubernetes](https://developers.redhat.com/topics/Kubernetes) 的平台时，FaaS 的产品通常是基于作为功能运行的小块源代码。通常没有必要编译它们。[open whish](https://openwhisk.apache.org/)——一个开源的 FaaS 平台——使用命令`wsk action create...`将例如 Node.js 文件转换成一个函数(在 open whish 的说法中它们被称为“动作”)。一个小文件和一个命令，你就有了一个函数。您甚至不需要编写任何 HTTP 处理程序或路由；这些都内置在平台中。而且只有一条路。该函数有一个入口和出口点；这不是一个复杂的 RESTful 服务，有多个 URIs。

Knative 通过将任何服务作为一种功能来使用，打破了这一切，因为 Knative 允许服务在配置的一段时间后扩展到零。换句话说，服务停止运行，这意味着没有 CPU 周期，没有磁盘活动，并且在空闲时间内*没有计费活动。那可不是小事。这是无服务器功能的本质，突然之间，一个处理四条不同路线的 RESTful 服务可以被认为是无服务器功能。*

诚然，扩展到零也有自己的挑战:如何最小化重启时间。那完全是另一篇文章。

因此，根据定义，任何可以扩展到零并按需响应的服务都是无服务器的(或 FaaS 或无服务器功能)。

## 但是等等，还有更多

Knative 还为开发人员带来了其他功能:构建、事件和服务是 Knative 的三个部分。我简单地讨论了服务，但是构建和事件也很重要。

例如，事件允许您通过使用事件来启动服务。将事件放入队列中，您就有了一个真正的事件驱动架构应用程序。如果您曾经在基于消息的平台上构建过应用程序(例如，Windows 桌面应用程序)，那么您会对事件和消息“到处飞”的想法很熟悉，从而使系统工作。如果处理得当，这是和谐代码的美妙交响乐。

(好吧，最后一部分有点夸张，但你明白了。)

Knative 利用先进快速的技术，包括 [Istio 服务网格](https://developers.redhat.com/topics/service-mesh)和 gRPC。虽然开发人员可能不需要知道这些事情，但是*有人*知道，而*知道*重要。简而言之，Knative 更像是一个平台，而不仅仅是一个实现。它为您自己的函数实现提供了一个广泛而健壮的基础。

## 我应该使用哪一个？

Knative 和类似 OpenWhisk 的东西都有理由。Knative 正在加速无服务器/FaaS 解决方案的发展(以及可能的采用),而 OpenWhisk 等现有技术仍然有用。此外，传统的 FaaS 平台是否以及如何接纳 Knative 还有待观察。

与任何技术一样，由您来决定哪种技术最适合您。有了知识的武装，有了零成本测试这些技术的机会，您就处于一个选择和前进的有利位置。随着技术的发展，您的解决方案也将随之发展。

有关 Knative 的更多信息，请访问 [GitHub repo](https://github.com/knative/) 和 [Knative 文档](https://github.com/knative/docs)。

*Last updated: May 2, 2019*