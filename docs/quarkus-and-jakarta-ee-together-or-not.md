# 夸库斯和雅加达 EE:在一起，还是不在一起？

> 原文：<https://developers.redhat.com/blog/2020/09/11/quarkus-and-jakarta-ee-together-or-not>

在这篇文章中，我回答了一个我在各种论坛上看到被问到的问题: [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 会和 [Jakarta EE](https://jakarta.ee/) 兼容吗？为了理解我们对这个问题的回答，了解夸库的历史以及我们试图用它来达到什么目的是很有帮助的。所以，在我打基础时候，请原谅我。

## quartus 和 java ee 简史

当 Emmanuel Bernard、Jason Greene、Bob McWhirter 和我第一次讨论启动 ThornFly.x 概念验证时，我们讨论了 T2 Java EE T3(现在的 Jakarta EE)最终适合哪里。我想我们都同意，我们已经有了最好的[开源](https://developers.redhat.com/topics/open-source)实现 Java EE 的 WildFly 和[红帽 JBoss 企业应用平台](https://developers.redhat.com/products/eap/overview) (JBoss EAP)的形式。为这个空间创造另一个空间看起来很令人困惑。最坏的情况是，我们担心这会分裂我们的工程和开源社区的努力。

不幸的是，我们有另一个理由拒绝让 Quarkus 与 Java EE 兼容:尽管[最好的](https://developer.jboss.org/blogs/mark.little/2019/05/03/jakarta-ee-and-the-future)和[的](https://developer.jboss.org/blogs/mark.little/2016/06/12/does-java-ee-have-a-future)[红帽工程师](https://developer.jboss.org/blogs/mark.little/2015/11/15/is-java-ee-still-relevant)、[我们的社区](https://developer.jboss.org/blogs/mark.little/2011/07/27/application-servers-are-dead)和其他供应商的努力，Java EE 被认为是行动缓慢、单一的，对新开发人员来说不是一个好的选择——特别是任何希望[构建微服务](https://developers.redhat.com/topics/microservices)和瞄准绿地工作的人。

一些批评者的负面报道夸大了这种看法，但是基于 20 多年前开发 Java EE 的用例，这也是事实。

## Eclipse 微文件

如果您已经看过 Java EE 一段时间，并且[观察过 Eclipse MicroProfile](https://developers.redhat.com/videos/youtube/GKYROutwJHU) 的开发，那么所有这些听起来可能都很熟悉。2016 年，Red Hat、Tomitribe、IBM 和其他一些公司一起推出了 MicroProfile。我们打算利用 Java EE 的精华，将它们与其他被认为不适合 Java EE 的开源成果结合起来(例如，[网飞操作系统](https://netflix.github.io)当时的一些好事情)。在此基础上，我们将发布一套专注于企业微服务的开放标准。

从那以后，Eclipse MicroProfile 越来越强大，拥有一个充满活力的社区、许多规范版本和一系列开源实现。它也受到了 Java 社区领导者的欢迎，他们希望能够使用选定的 Java EE APIs，并在 Java EE 应用服务器中运行某些微文件实现。

### quartus 创新和 microprofile

让 Quarkus 跟踪和影响 MicroProfile 是有意义的:已经熟悉 Java EE 的开发人员可以重用他们的技能和知识，我们可以重用许多现有的、兼容的规范实现作为 Quarkus 扩展。也就是说，我们必须评估 Quarkus 是否需要与 MicroProfile 完全兼容。这个决定是基于我们在 Quarkus 中创新的速度，以及这些创新是否普遍适用于 MicroProfile 社区。正如 Red Hat 从一开始就说过的，我们不相信在标准内创新。并不是每一个 Quarkus 的创新都一定会成为 MicroProfile 的一部分，也不是我们所有的建议都会被 MicroProfile 接受。这完全没问题；这是一个充满活力的社区的标志之一。

### 反应范式

作为我的意思的一个具体例子，许多 MicroProfile 社区拒绝接受反应式范例。在 Red Hat，我们多年来一直积极参与反应空间，做出了各种努力，如 [Eclipse Vert.x](https://developers.redhat.com/videos/youtube/o-cBfanMJ8A) 和现在的[Knative/无服务器](https://developers.redhat.com/topics/serverless-architecture)。然而，我们看到了将现有的 Java EE 规范引入反应性工作的局限性，因为 Java EE 的原始设计是同步的。我们并不总是能够说服更广泛的社区，所以我们将我们的反应式创新集中在 Vert.x 上，而[现在集中在 Quarkus](https://developers.redhat.com/blog/2020/08/07/reactive-quarkus-a-java-mutiny/) 上。我们相信，我们可以展示这些新方法的重要性，以及我们的社区采用这些方法的情况。通过在其他地方创新，我们可能最终说服微剖析社区采用反应范式。

## 夸库斯和雅加达 EE 怎么样？

大多数开发人员都知道 Java EE 已经转移到 Eclipse Foundation，现在更名为 Jakarta EE。Eclipse 下企业 Java 的新使命是[驱动云原生的关键任务应用](https://jakarta.ee/about/)。我们不能把它折叠成夸库吗？

好吧，我们不要忘乎所以。到目前为止，Jakarta EE 8 在技术上相当于 Java EE 8。今年晚些时候发布的 Jakarta EE 9 主要关注包名从`javax`到`jakarta`的变化。Jakarta EE 10 会变成什么样，Jakarta 和 MicroProfile 一起玩的好不好(或者是否)将会被决定。

与任何开源努力一样，要获得最佳结果，许多因素必须结合在一起。至少，Jakarta EE 必须为更新和创新提供更快的发布节奏。Jakarta EE 还必须将自己与 Java EE 相关的负面看法区分开来。为了做好，它应该看起来与 Java EE 如此不同，以至于实际上有一个迫在眉睫的新问题。

Java EE 的成功是基于优先考虑向后兼容性和缓慢的变化。这一优先级别使运营团队可以放心地部署任务关键型应用程序，而无需一次进行几个月或几年的更改。虽然 Jakarta EE 可能既能完成它的新使命，又能解决过去的用例，但这不是一件容易的杂耍行为。这不太可能在一夜之间实现。

## Quarkus 会兼容雅加达 EE 吗？

我已经给出了背景的简要版本，让我们回到最初的问题:Quarkus 会兼容 Jakarta EE 吗？简而言之，我认为雅加达 EE 8 或雅加达 EE 9 不会出现这种情况。当然，我永远不会说永远不会:没有人能预测 Jakarta EE 会走向何方。然而，要使 Quarkus Jakarta EE 兼容完全的本地 Java 支持是非常困难的。试图这样做也可能会混淆今天对夸库的理解。Quarkus 不需要兼容 Jakarta EE 就能成功。此外，通过 WildFly 和 JBoss EAP，我们有了一个更好的机会来跟踪和影响 Java 企业标准的未来。

## 结论

现在，我认识到这个消息对于一些社区成员来说可能很难接受，特别是基于我们所看到的关于这个话题的兴趣。但是我希望你能理解我们的推理，即使你不同意。在 Kubernetes 的世界里，Red Hat 和更广泛的社区有一个巨大的机会用 Quarkus 重新定义企业 Java。这样做需要我们所有的注意力和努力。所以，我将留给你道格拉斯·麦克洛伊关于 Unix 哲学的话:“让每个程序做好一件事；要做一项新工作，就要重新构建，而不是通过添加新功能来使旧程序变得复杂。”

*Last updated: September 10, 2020*