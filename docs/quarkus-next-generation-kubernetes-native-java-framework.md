# Quarkus 简介:下一代 Kubernetes 原生 Java 框架

> 原文：<https://developers.redhat.com/blog/2019/03/07/quarkus-next-generation-kubernetes-native-java-framework>

Java 在 20 多年前被引入开源社区，至今仍在开发者中广受欢迎。事实上，Java 在 [TIOBE 指数](https://www.tiobe.com/tiobe-index/) 上的排名从未低于#2。Java 诞生于 20 世纪 90 年代中期，经过近 20 年的优化，它已经可以运行高度动态的单块应用程序，这些应用程序假定拥有(虚拟化的)主机 CPU 和内存的唯一所有权。然而，我们现在生活在一个由云、移动、物联网和开源主导的世界，其中容器、Kubernetes、微服务、反应式、功能即服务(FaaS)、 [12 因素](https://12factor.net/) ，以及云原生应用程序开发可以提供更高水平的生产力和效率。作为一个行业，我们需要重新思考如何最好地利用 Java 来处理这些新的部署环境和应用程序架构。

我们将向您介绍**和 *超音速亚原子 Java* ！**

 **Quarkus 是为 GraalVM 和 HotSpot 量身定制的 Kubernetes 原生 Java 框架，采用了一流的 Java 库和标准。*quar kus 的目标是让 Java 成为 Kubernetes 和无服务器环境中的领先平台，同时为开发人员提供一个统一的反应式和命令式编程模型，以优化解决更广泛的分布式应用架构。*

## 集装箱第一

Quarkus 提供了显著的运行时效率(基于 Red Hat 测试)，例如:

![](img/f961f2115e3277c59ea80937256a9fed.png)

*   快速启动(几十毫秒)允许容器和 Kubernetes 上的微服务自动缩放，以及 FaaS 现场执行
*   在需要多个容器的微服务架构部署中，低内存利用率有助于优化容器密度
*   更小的应用程序和容器映像占用空间

## 命令式和反应式的统一

大多数 Java 开发人员都熟悉命令式编程模型，并愿意在采用新平台时利用这种经验。与此同时，开发人员正在快速采用云原生、事件驱动、异步和反应式模型来解决业务需求 以构建高度并发和响应迅速的应用 。Quarkus 旨在将这两种模型无缝地结合到同一个平台中，从而在组织内产生强大的杠杆作用。

## 开发者的喜悦

优化开发人员乐趣的内聚平台:

![](img/3f5ef5be3b46834e8ee9430737d3a9c5.png)

*   统一配置，所有配置都在一个属性文件中。
*   零配置，一眨眼就能实时重新加载
*   80%常见用法的简化代码，20%的灵活代码
*   轻松生成本地可执行文件

## 同类最佳的库和标准

Quarkus 带来了一个有凝聚力的、有趣的全栈框架，它利用了您喜爱的、在标准主干网上使用的同类最佳库，包括 Eclipse MicroProfile、JPA/Hibernate、JAX-RS/RESTEasy、Eclipse Vert.x、Netty 等等。

Quarkus 还包括一个扩展框架，第三方框架作者可以利用它来扩展它。Quarkus 扩展框架降低了在 Quarkus 上运行第三方框架并编译成 GraalVM 本地二进制文件的复杂性。

## 总结

Quarkus 为在这个无服务器、微服务、容器、Kubernetes、FaaS 和云的新世界中运行 Java 提供了一个有效的解决方案，因为它在设计时就考虑到了这些。其面向云原生 Java 应用的容器优先方法统一了微服务开发的命令式和反应式编程范式，并提供了一套可扩展的基于标准的企业 Java 库和框架，结合了极高的开发效率，有望彻底改变我们的 Java 开发方式。

我们希望您加入 Quarkus 开源社区。如果您有兴趣帮助我们继续改进 Quarkus，开发第三方扩展，使用 Quarkus 开发应用程序，或者您只是对此感到好奇，请加入我们:

*   夸尔库斯网址:[http://夸尔库斯. io](http://quarkus.io)
*   Quarkus GitHub 项目:【https://github.com/quarkusio/quarkus 
*   夸尔库斯推特:[T3【https://twitter.com/QuarkusIO】](https://twitter.com/QuarkusIO)
*   夸库聊天:[](https://quarkusio.zulipchat.com/)

被炒鱿鱼！让我们开始制造一些强子

*Last updated: April 7, 2022***