# 选择哪款 Camel DSL，为什么？

> 原文：<https://developers.redhat.com/blog/2017/12/21/camel-dsl-choose>

Apache Camel 是一个强大的集成库，它主要提供三种东西:大量的集成[连接器](http://camel.apache.org/components.html) +多个集成[模式的实现](http://camel.apache.org/enterprise-integration-patterns.html) +一个更高级的领域特定的[语言](http://camel.apache.org/dsl.html) (DSL)抽象来很好地将所有这些结合在一起。虽然连接器和模式的选择是由用例和特性驱动的，并且很容易做出，但是选择使用哪种 Camel DSL 可能有点难以推理。我希望这篇文章能对你的第一次骆驼之旅有所帮助。

我在 Red Hat Consulting 工作，是一名集成架构师，我的主要目标之一是帮助客户获得尽可能正确的未来系统设计和架构，从而从 Apache Camel 中获得最佳价值。在每一个新的基于 Camel 的项目开始时，我得到的一个常见问题是:"*我们应该使用哪种 Camel DSL？各有什么利弊？*

我有两条好消息要告诉你。首先，通过选择使用 Apache Camel，您已经做出了正确的选择。Camel 将会成为你的类库中一个非常有用的工具包，用于未来的用例以及未来的项目。第二，DSL 只是一个技术问题，不会影响项目的成功。以后随时可以改变主意，甚至混搭。

如果你是一个大公司的一部分，有多个独立的两个比萨饼大小的团队到处使用 Java 语言，有可能一些团队已经在使用 Camel 了。即使在小公司，团队使用 Camel 也不会互相察觉。Camel 对所有类型的任务都很有用，它有一个足够小的库，您可以将其添加到 pom.xml 中，并在没有技术设计委员会许可的情况下使用它。如果是这种情况，就和你的同事谈谈，直接了解他们选择 DSL 的体验。

如果您需要更全面的比较和选择 DSL 的理由，下面的图表是来自开发 Apache Camel 的多名工程师和在全球多个客户项目中使用 Camel 的顾问的头脑风暴。挑选在你的上下文中有效的论点，并做出你的选择。

[![Comparing Apache Camel's XML and Java DSLs](img/378b9c34a1e2720ec480495592e2d057.png "Camel XML DSL vs Java DSL")](/sites/default/files/blog/2017/12/Camel-XML-DSL-vs-Java-DSL.png)Comparing Apache Camel's XML and Java DSLs">

如果这张表没有给你想要的直接答案，很可能答案是:**没关系。Camel 有多种 DSL，但是有充分的理由让 Java 和基于 XML 的 DSL 同样受欢迎。更重要的是，开发人员要习惯用[管道和过滤器](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html)来思考，学习[企业集成模式](http://www.enterpriseintegrationpatterns.com/)和它们的符号。使用 Camel DSLs 之一来表达这些模式是一个技术问题，没有技术后果。通常是团队偏好和基于文化的选择，比如“*我们是一家铁杆 Java 商店，我们讨厌 XML* ”或者“*我们可以通过拖放来完成所有工作吗？*”。**

综上所述，我唯一能建议的是努力保持一致性。避免在同一个服务中使用不同的 DSL，即使是在同一个项目中的不同服务。如果你能说服公司里的每个人都使用同一个 DSL，那就更好了。

![Bilgin Ibryam](img/05e55175ae394f04f0245ef4d983c0ad.png)

### 关于作者:

Bilgin Ibryam 是 Apache 软件基金会的成员，Red Hat 的集成架构师，软件工匠和 T2 的博客作者。他是一个开源狂热者，对分布式系统、消息传递和应用程序集成充满热情。他是[骆驼图案](https://leanpub.com/camel-design-patterns) & [Kubernetes 图案](http://leanpub.com/k8spatterns)书籍的作者。关注 [@bibryam](http://twitter.com/bibryam) 以获取相关主题的未来博文。

* * *

**[红帽 JBoss Fuse](http://developers.redhat.com/products/fuse/overview/) 是一个开源、轻量级、模块化的集成平台。利用 Apache Camel，JBoss Fuse 附带了超过 160 个内置组件。[使用](https://developers.redhat.com/blog/2017/10/10/fuse-development-environment-development-suite-installer/) [Red Hat 开发套件](https://developers.redhat.com/products/devsuite/overview/)开始【JBoss Fuse 开发。**

*Last updated: December 18, 2017*