# 成功的 Java Web 应用程序的 5 个支柱(第 3/3 部分)

> 原文：<https://developers.redhat.com/blog/2017/11/08/5-pillars-successful-java-web-application-part-33>

在这一系列帖子中，我们将详细介绍我们在 Java One San Francisco 2017 上的演讲:“ [一个成功的 Java Web 应用的 5 大支柱](https://speakerdeck.com/ederign/5-pillars-of-a-successful-java-web-application-1) ”，在这里我们分享了多年来为 Drools 和 jBPM 平台构建工作台和 Web 工具的累积经验。如果你没有读过前几本书，那就找个机会接触一下这些柱子[第一本的链接]。

## 第四支柱:5~10 年寿命

下一个支柱是如何让我的 web 应用持续 5~10 年以上。您的后端的预期寿命是多少？大概 2 年内你不打算扔了吧。

然而，如果你与一些前端工程师交谈，他们的回答会略有不同，令人惊讶。有些人说，由于 JavaScript 框架的发展，你应该预期每两年扔掉你的前端代码。真的吗？我是否应该每两年重写一次我的前端应用程序的业务逻辑？

众所周知，让应用程序持久是一项架构挑战。几个建筑模型分享相似的原则，即阿利斯泰尔考克伯恩的六边形建筑，杰弗里帕勒莫的洋葱建筑，罗伯特马丁的清洁建筑。这些原则是:

*   与框架分离
*   可测试
*   与 UI 解耦
*   与数据库分离
*   独立于外部系统

为了说明这些原理，请看下面的 [洁净建筑](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) 示意图:

![](img/28408bd76cb71d6dff381d81ad039a80.png)

应该保护实体和用例不受任何外部代理的影响。这意味着 UI 中的外部修改永远不会影响它们。我们的应用程序的真正价值是业务规则和核心逻辑，它们应该是稳定的，并且只在业务变化时才变化。

对于后端开发人员来说，这个原则的重要性是显而易见的。例如，在 AppFormer 上，我们有一个虚拟文件系统 API 它定义了 I/O 操作的契约。

![](img/8495ffc8f0c120db0d4184f6c59cf1b7.png)

这个 API 是常规文件系统上 I/O 操作的 NIO2 包装器，或者在分布式环境中可以切换为分布式 GIT NIO2 backport。这样，我们可以从简单的文件存储转移到分布式 git 后端，而不必改变任何用例。怎么会？因为我们用虚拟文件系统接口保护了我们的用例。

对于后端开发者来说，这并不新鲜。这就是我们目前实现大多数后端架构的方式。

然而，作为一个行业，当我们设计前端应用程序时，我们有类似的关注吗？或者我们只是评估和采用 web 框架并遵循他的模型？我们不是依赖 JS 框架来定义我们的 web 架构吗？

这种方法的问题在于，我们需要在项目的第一天就决定 JS 框架。但现实是，只要业务需要，这种支持就必须持续下去。

![](img/854a3b05f1e35b3792c849febe96b8eb.png)

但是这种方法有一个警告。很有可能在当前 JS 框架的第二年和第四年之间，该项目将被取消或被与当前版本不兼容的新版本所取代。类似于[棱角分明的 1.x 和 2 层楼的](https://toddmotto.com/future-of-angular-1-x)。

那我们该怎么办？我们要重写整个客户端代码吗？为什么我们认为这样做是可以的？而留在老版本有什么风险？更新呢？我如何找到资源来使用过时的技术？安全隐患怎么办？ [数据泄露](https://thehackernews.com/2017/09/equifax-apache-struts.html) 为例。

Robert C. Martin 说过，一个好的架构允许不稳定的决策被轻易改变。如果我们把 JS 框架的易变性作为一个事实来处理会怎么样？

这个问题的实现是我和 Alex Porcelli 领导的 Appformer 项目的一部分。

AppFormer(以前称为 Uberfire)是 Red Hat JBoss Business Rules Management System(BRMS)和 Red Hat JBoss BPM Suite (BPMS)背后的基于 web 的工作台框架。AppFormer 也是 BRMS 和 BPMS 平台的下一个业务线的基础:一个开发现代商业应用程序的低代码/无代码平台。我们的计划旨在允许商业用户通过混合组件并将其连接到其他 Red Hat 模块和软件来轻松构建应用程序。

我们的主要架构目标是我们的核心业务不依赖于任何 web 框架。我们是怎么做到的？

![](img/cf361d09955ab2cedb82f386f4980c06.png)

我们已经创建了一个编程模型，它有一个基于屏幕、编辑器、透视图和弹出窗口的定义良好的组件模型。这些组件中的每一个都有明确的生命周期。

从用户的角度来看，屏幕就是一个组件。编辑器是与文件类型相关联的组件。透视图是一个页面。

![](img/da00e99c01058889a6c8d727803262ea.png)

这并不新鲜；这是旧的基于契约的架构。每个组件最终都是一个 Java 接口。这种方法的优点是什么？

还记得 Errai 在浏览器中有一个 Bean 管理器吗？定义良好的编程模型允许我们在组件接口的实现之间快速切换，而不是与特定的 web 技术耦合。我们基于接口类型和 bean 名称来呈现组件，而不是基于实际的实现/框架。

![](img/f8df8387ccf4ea1e2f713e7c35fc00a2.png)

我们有遗留代码，这些代码是使用旧技术开发的，比如 GWT 小部件，但是当我们使用纯 HTML/CSS 实现现代化时，这只是一个切换实现的问题，因为两个实现都遵循相同的契约。这就是在浏览器中拥有基于契约的模型和 CDI 的好处。

这就是我们如何设法透明地运行 7 年前实现屏幕界面的 GWT 代码，以及在 Errai UI 中实现的新代码。

这就是成功的网络应用的第四个支柱。您应该准备好您的架构，以度过 JS 框架超过 2~4 年的生命周期，为了做到这一点，您的 web 架构应该受到与您对待后端相同的尊重。

## 第五大支柱:互操作性

最后但同样重要的是，成功的 web 应用程序的第五个支柱是互操作性。为了保持时尚，你需要灵活变通。正如我们在第四支柱中所讨论的，我们需要一个可靠的架构，但我们也不能停留在旧技术上，需要为我的第三方提供互操作性。我们需要拥抱新技术。我们如何做到这一点？

我们在上一篇文章中看到，我们使用 Java 接口来定义我们的组件。为了避免样板代码，我们使用 Java 注释处理器来自动生成从用户客户端到接口的适配器。(即@[workbench perspectives](https://speakerdeck.com/ederign/5-pillars-of-a-successful-java-web-application-1?slide=81)注释触发 PerspectiveActivity 接口实现的生成)。

为了与任何外部 web 框架集成，需要实现一个映射到目标接口并注册的新适配器。它的实现发生在 Errai Bean 管理器中。在这之后，一切都将成为同一个契约的实现，并且对用例是透明的。 [这里的](https://speakerdeck.com/ederign/5-pillars-of-a-successful-java-web-application-1?slide=84) 是一个我们已经用角度代码做到这一点的例子。

![](img/8b52e0541da9085b4898028a300286fd.png)

在这个帖子系列中，我们展示了我们认为是每个成功的网络应用的基础的五大支柱。它们都已经在 Drools 和 jBPM 工作台的开发实践中证明了自己的价值。

Web 应用程序是我们架构的重要组成部分。尽管您的应用程序可能没有实现所有这 5 个支柱，但是您应该始终记住它们，因为我们确实相信所有这 5 个支柱为任何 web 应用程序提供了一个坚实的基础。

我要感谢 Max Barkley 和 Alexandre Porcelli 在本文发表前审阅了本文，为最终文本做出了贡献，并提供了很好的反馈。]

* * *

**For a development environment with superior support for your entire development lifecycle download [Red Hat JBoss Developer Studio](https://developers.redhat.com/products/devstudio/download/?intcmp=7016000000124eFAAQ).***Last updated: November 1, 2017*