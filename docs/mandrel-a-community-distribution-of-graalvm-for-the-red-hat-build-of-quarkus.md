# mandrel:quar kus 的 Red Hat build 的 GraalVM 社区发行版

> 原文：<https://developers.redhat.com/blog/2020/06/05/mandrel-a-community-distribution-of-graalvm-for-the-red-hat-build-of-quarkus>

Java 社区已经一次又一次地展示了它进化、改进和适应以满足其开发者和用户需求的能力。即使在经历了 25 年的语言和框架选择之后， [Java](https://developers.redhat.com/topics/enterprise-java/) 由于其在企业用例中强大的跟踪记录和能力，一直在今天使用的 [顶级](https://www.tiobe.com/tiobe-index//) [语言](https://spectrum.ieee.org/computing/software/the-top-programming-languages-2019) [中排名第一。长期以来，Red Hat 一直是 Java 和开源软件开发的强大领导者，并致力于在 Java 不断发展的过程中保持领先地位。](https://redmonk.com/sogrady/2020/02/28/language-rankings-1-20/)

今天，红帽与 [GraalVM 社区](https://www.graalvm.org/community/) 共同建立了一个新的 GraalVM 下游分布，名为 [【心轴】](https://github.com/graalvm/mandrel) 。 这个分配将权力的 [红帽打造的](https://access.redhat.com/products/quarkus) ，最近一个[宣布除了 红帽之外还有](https://developers.redhat.com/blog/2020/05/28/quarkus-a-kubernetes-native-java-runtime-now-fully-supported-by-red-hat/) 。这篇文章解释了什么是心轴，为什么它是必要的。

## Java 向前发展

一段时间以来，Red Hat 一直在关注 Java 的 [未来，以及它的客户和他们的开发者如何在这个由](https://www.redhat.com/en/blog/looking-next-20-years-enterprise-java)[容器](https://developers.redhat.com/topics/containers/)、[微服务](https://developers.redhat.com/topics/microservices/)和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 组成的新世界中继续使用他们多年的 Java 经验。这种 Java 体验不仅仅适用于这种语言——它还包括许多已经在开发人员的记忆中根深蒂固的库和框架，例如:Hibernate、CDI、RESTEasy、MicroProfile 或诸如 Eclipse Vert.x 等反应式框架。

[quar kus 项目](https://quarkus.io/) 于 2019 年启动，为这个 Kubernetes 和[无服务器](https://developers.redhat.com/topics/serverless-architecture/)的新世界中的 Java 开发者提供了所需的进化步骤。本质上，Quarkus [改变了 Java 游戏](https://siliconangle.com/2020/04/29/quarkus-makes-java-compatible-with-new-cloud-native-app-development-rhsummit-rhsummit/) 的规则。I t 优化了 Java 应用程序和支撑它们的框架，以更好地匹配部署它们的受限环境，并逆转了 Java 早期的架构和设计选择。Quarkus-native 应用程序以牺牲吞吐量为代价带来了更小的内存占用，现在通过伸缩和弹性来处理这一点——与 Kubernetes 中的方式相同。它还以动态运行时行为为代价带来了更快的启动速度，这在不可变的部署架构中是不必要的开销——就像您在 Kubernetes 中发现的一样。

## Quarkus 和 GraalVM

[GraalVM](https://www.graalvm.org/) 是一个生态系统和共享运行时，为包括 Java 在内的多种语言提供性能优势。它能够提前编译以创建极其优化的 Java 应用程序，这使得它特别适合于在比传统 JVM 部署更小的内存中运行 Java。随着几年前 GraalVM 项目的引入，设计 Quarkus 和它提供的许多框架，使它有可能与这个工具一起使用变得很有意义。这一决定为 Quarkus 应用程序提供了进一步的优化，并帮助它们无缝、轻松地协同工作，而不会牺牲开发人员熟悉和喜爱的丰富 API。GraalVM 已经成为 Quarkus 故事的重要组成部分，Red Hat 致力于它的成功。Red Hat 是 GraalVM 项目顾问委员会[](https://medium.com/graalvm/announcing-the-graalvm-project-advisory-board-282223cde700)的成员，并定期为 GraalVM 社区提供功能和修复，如改进的原生映像可调试性、AArch64 原生映像支持，以及继续支持 Java 飞行记录器(JFR)。

红帽最近 [宣布支持夸尔库斯](https://www.redhat.com/en/about/press-releases/red-hat-advances-java-kubernetes-delivers-quarkus-fully-supported-runtime-cloud-native-development) 供客户生产使用。通过使用 Quarkus 的 [Red Hat build，我们的客户现在已经为他们的 Kubernetes 和无服务器应用程序提供了完全支持和高度优化的 Java 解决方案。开发人员还可以使用 GraalVM 将他们的 Quarkus 应用程序编译成本地二进制文件，进一步优化云和 Kubernetes。这项功能目前在技术预览版](https://access.redhat.com/products/quarkus)[](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.3/html/release_notes_for_red_hat_build_of_quarkus_1.3/ref-rn-technology-preview)中，因为我们与 GraalVM 社区合作，所以我们可以在 Quarkus 中使用它时支持 Red Hat 客户——这将我们带到了 Mandrel。

## 心轴

Red Hat 对开源社区的贡献巩固了其企业级支持模式。我们坚信，开源创新不仅对 Red Hat 的成功至关重要，对它所服务的社区也是如此。GraalVM 是一个包含许多活动部分的大型项目，每天都有来自 Red Hat、Oracle 和许多其他 GraalVM 社区成员的贡献。

我们发现，在支持我们的客户的同时保持对我们的开源承诺的最佳方式是建立 *下游* 开源发行版，与上游的同行合作。你看这个用 Linux 的下游发行版用[Fedora](https://getfedora.org/)和[CentOS](https://www.centos.org/)，Kubernetes 用[](https://www.okd.io/)，现在用 [心轴](https://github.com/graalvm/mandrel) 作为 GraalVM 的下游。这些社区携手合作，以对双方都有意义的方式推进开源技术。这也使得红帽在开放中不断创新，甚至在其产品化过程中，都抱着“上游第一”的心态，宁愿不偏离上游。

对于 Quarkus 来说，GraalVM 的重要部分是其生成本机可执行文件的本机映像特性，这是使 Java 在云本机工作负载中具有竞争力的关键特性。Mandrel 允许我们将 GraalVM 捆绑在 Red Hat Enterprise Linux 和其他 OpenJDK 11 发行版中的 OpenJDK 11 之上。在 GraalVM 方面，如果发布时间要求的话，这允许在 Mandrel 中比 GraalVM 更快地支持正在进行的 Java 飞行记录器等特性。因此，可以将 Mandrel 最好地描述为带有特殊打包的 GraalVM 本机映像的常规 OpenJDK 的发行版。

对于用户来说差别很小，但是对于可维护性来说，与 OpenJDK 11 和 GraalVM 的上游一致性是至关重要的。这意味着 Red Hat 可以为客户提供更好的支持，因为我们有熟练的工程师在 OpenJDK 和 GraalVM 社区工作。

有了 Mandrel，Red Hat 客户和 GraalVM 社区都从真正的开放开发中受益，Red Hat 可以用可靠的机制支持其客户，同时回馈上游社区，继续推进开源计算的发展。

*Last updated: January 17, 2022*