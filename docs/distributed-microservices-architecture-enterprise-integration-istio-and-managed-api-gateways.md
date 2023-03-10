# 分布式微服务架构:Istio、托管 API 网关和企业集成

> 原文：<https://developers.redhat.com/blog/2019/03/12/distributed-microservices-architecture-enterprise-integration-istio-and-managed-api-gateways>

[微服务](https://developers.redhat.com/topics/microservices/)架构的兴起彻底改变了软件开发的前景。在过去的几年中，我们已经看到了从集中式单片计算机到分布式计算的转变，这得益于云基础设施。随着分布式部署、微服务的采用以及系统向云级别的扩展，新的问题以及试图解决这些问题的新组件出现了。

到现在为止，你很可能已经听说过[服务网](https://developers.redhat.com/topics/service-mesh/)或 [Istio](https://developers.redhat.com/topics/service-mesh/) 来拯救世界了。然而，您可能想知道它如何适应您当前的企业集成投资和 API 管理计划。这就是我在本文中讨论的内容。

## 解决互动挑战

网飞是第一个将应用程序网络层分离出来的人，他们创建了著名的 OSS 堆栈，包括 Hystrix、Eureka 和 Ribbon 等。反过来，Docker 开发了容器运行时和图像格式，使谷歌能够抽象他们的基础设施和开源 Kubernetes，这是新的云原生浪潮中最重要的项目之一。很快，全世界的开发者都意识到 Kubernetes 提供了新的工具来解决网飞过去针对的问题。

随后，Envoy 代理——最初由 Lyft 开发的高性能分布式代理——为服务网格提供了基础。服务网格是应用程序服务之间的连接，它增加了弹性、安全性、可观察性、路由控制和洞察力等功能。Istio 是一个服务网格的实现，它允许应用程序将这些功能从应用程序级库卸载到下一层。

与此同时，随着上述技术变革，组织开始了数字化转型的业务之旅。传统的从现有渠道中获利的方法被各个垂直行业中新的数字原生挑战者的出现所震惊。

从优步到 Airbnb 的软件颠覆，新的玩家出现在这个领域，抢占市场份额，或者提供新的方式进入未开发的市场。这些参与者之间的互动促进了欧盟等地区法规的变化，导致了通过提供对其系统和流程的访问来创造价值的新方法的增长。因此，我们称之为“API 经济”，即开发软件应用程序的组织通过 API 访问软件，为他们自己和更广阔的世界创造新的功能和价值。我的同事 Steve Willmott 和 Manfred Bortenschlager 就这个话题写了一本颇有见地的书，名为《赢得 API 经济》。我鼓励你去看一看。

## 控制您的 API

在 API 经济中，关注 API 的价值是至关重要的。这就是为什么大多数 API 设计规范都是从业务角度精心制作的，并且超越了原始服务到服务通信。在业务 API 领域，管理内部和外部 API 消费者和 API 创建者之间的关系需要一个精心设计的策略。

API 管理使团队能够轻松发现和记录数百种可用的 API 资产，并提供灵活的方式来解决不同开发团队的访问类型，以及直接的消费计费和发票。API 管理为 API 所有者处理这些类型的需求。

然而，API 管理解决的不仅仅是 API 计划的发现和文档。最好的产品还提供:

*   访问控制和安全性
*   “作为产品的 API”
*   API 订阅计划和使用限制
*   分析和报告
*   开发者参与
*   账单和发票

API 网关是传统上用于实施这些需求并遵守其他一般策略的元素。正如我的同事 Joaquim Moreno Prusi 在他的博客文章中所说的，

> “API 管理(不仅仅)是速率限制。”

顺着同样的思路，我们不要就此打住；相反，让我们用下面的语句来扩展这个想法:

> “你不能通过使用 API 网关来获得 API 管理。”

正如我之前提到的，API 管理不仅仅是实现 API 调用的网关模式。大多数时候，您需要将实际的管理组件与策略实施点分离。

## 微服务网络

因为一个托管 API 响应一个业务目标，很多时候你会发现在实现方面的背后是许多服务调用处理一个 API 请求。随着服务的增加，它们之间的交互也在增加，并增加了管理网络的复杂性。您添加到架构中的部分越多，它可能失败的地方就越多。Christian Posta 在他的博客文章[微服务最难的部分:调用你的服务](https://blog.christianposta.com/microservices/the-hardest-part-of-microservices-calling-your-services/)中对这些问题做了有趣的回顾。

因此，服务网格有助于团队以一种更优雅的方式解决之前的一些问题，如服务调用、负载平衡、可观察性和弹性。例如，用 Istio 实现的 mesh 删除了嵌入在服务中的所有网飞代码，并将实现委托给代理 sidecar。因此，配置也是独立于业务逻辑和代码进行管理的。通过这种方式，系统管理员团队可以收回对如何监控和实施服务交互的控制权。

到目前为止，API 和服务之间的大多数交互都是对非常普通的问题做出响应。然而，当我们需要处理特定领域的交互时，第三个阶段开始了。基于服务调用的业务上下文做出决策远远超出了 API 管理或服务网格所能提供的一般方法。一些提供商建议将所有东西捆绑在一起，以证明遗留组件的合理性。就我而言，我更倾向于避免这个提议，因为我同意 ThoughtWorks 关于处理过于雄心勃勃的 API 网关的风险的观点。

因此，在处理“连接和组合”需求时，最好将实现委托给经过充分测试的分布式集成框架，这些框架实现了已知的企业集成模式。开发人员和集成商可以使用这些工具在业务领域级别正确处理编排、数据转换或内容路由。

## 服务网格、API 管理和企业集成的共同基础

正如我们在前面的段落中所看到的，我们有解决现代应用程序开发的共同基础的解决方案。然而，它们专注于解决问题的目标方面，因为它们最初是为了解决不同的问题而设计的。因此，我们不会将它们视为竞争对手或彼此的替代者，而是需要在设计良好的架构中部署的补充选项。

下图高度概括了目标用户以及 API 管理、服务网格和应用程序集成的关注点和常见功能。

[![API management, service mesh, and integration overlap](img/a6f5efef892551273a0a8993e6ea2da2.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/02/Untitled-presentation.png)

Service mesh 将能够进行一些速率限制，但它无法处理基于订阅计划的安全性。它将能够进行一些路由和重试，但是基于内容的决策会有很大的开销。

我必须承认，一起玩是一个挑战，因为技术永远不会静止，像无服务器和 K-native 这样的东西到处都在涌现。正如 Christina Lin 在她的[My two cents on the future of integration](http://wei-meilin.blogspot.com/2019/01/my-2cents-on-future-of-integration-with.html)中所分享的，当处理现实生活中的架构设计时，您仍然需要逻辑“被写在某个地方”因此，我相信会有许多其他解决方案来解决新发现的问题。

想做决定？记住一句谚语“当你是一把锤子时，所有东西看起来都像钉子。”当你真正需要广泛的工具集时，要小心那些试图用同一个工具解决所有问题的人。

## 欲了解更多信息

*   [完整的 API 生命周期管理:初级读本](https://developers.redhat.com/blog/2019/02/25/full-api-lifecycle-management-a-primer/)
*   API 之旅:以敏捷的方式从想法到部署(第 1 部分，共 3 部分)
*   [使用 Red Hat 集成轻松创建完整 API 生命周期的 API](https://developers.redhat.com/blog/2019/02/11/red-hat-integration-effortless-api-creation/)
*   关于 Red Hat 开发人员的书籍、视频和文章，涵盖了 Istio 和服务网
*   [将 Coolstore 微服务引入服务网络:第 1 部分——探索自动注入](https://developers.redhat.com/blog/2018/04/05/coolstore-microservices-service-mesh-part-1-exploring-auto-injection/)
*   [观察您的 Istio 微服务网格如何处理 Kiali](https://developers.redhat.com/blog/2018/09/20/istio-mesh-visibility-with-kiali/)
*   [借助 Red Hat 3scale API 管理，添加 API 网关策略变得更加容易](https://developers.redhat.com/blog/2018/05/30/3scale-api-gateway-policies/)

*Last updated: January 24, 2022*