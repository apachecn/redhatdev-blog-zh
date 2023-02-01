# Quarkus，一个 Kubernetes-native Java 运行时，现在完全受 Red Hat 支持

> 原文：<https://developers.redhat.com/blog/2020/05/28/quarkus-a-kubernetes-native-java-runtime-now-fully-supported-by-red-hat>

Java 在 25 年前被引入，直到今天，仍然是开发人员中最受欢迎的编程语言之一。然而，Java 因不太适合[云原生应用](https://www.redhat.com/en/topics/cloud-native-apps)而声名狼藉。开发人员寻找(并且经常选择)替代框架，如 Go 和 [Node.js](https://developers.redhat.com/blog/category/node-js/) 来支持他们的云原生开发需求。

当你可以使用现有的技能时，为什么还要学习另一种语言呢？ [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 允许 Java 开发者利用他们的专业知识开发云原生、[事件驱动](https://developers.redhat.com/topics/event-driven/)、[反应式](https://developers.redhat.com/coderland/reactive/)和[无服务器](https://developers.redhat.com/topics/serverless-architecture/)应用。Quarkus 提供了一个内聚的 Java 平台，感觉既熟悉又新颖。它不仅利用了现有的 Java 标准，还提供了许多优化开发人员乐趣的特性，包括实时编码、统一配置、IDE 插件等等。

最近，[红帽宣布支持夸库](https://www.redhat.com/en/about/press-releases/red-hat-advances-java-kubernetes-delivers-quarkus-fully-supported-runtime-cloud-native-development?source=pressreleaselisting)。通过 Quarkus，Red Hat 在 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 上推进了 Java，并在传统 Java 应用程序和云原生环境之间架起了一座桥梁。

## 夸库斯是什么？

Quarkus 不仅仅是一个运行时。它是一个 Kubernetes-native Java 堆栈，用于构建快速、轻量级的微服务和无服务器应用程序。它是专为利用云原生应用的优势而构建的。Quarkus 为部署在 Kubernetes 上的应用程序提供了显著的运行时效率，启动时间快，内存利用率低，映像占用空间小。

## 现代 Java 堆栈

Quarkus 项目的创始原则之一是[为企业 Java 开发人员带来开发乐趣](https://quarkus.io/vision/developer-joy)。那是什么意思，夸库斯是如何带来快乐的？

### kubernetes 原生 Java

Quarkus 是一个 [Kubernetes-native](https://developers.redhat.com/blog/2020/04/08/why-kubernetes-native-instead-of-cloud-native/) Java 框架，目标是容器和无服务器，因为它启动快、内存低、应用程序小。

### 开发者快乐

Quarkus 与流行的 Java 标准、框架和库一起开箱即用。熟悉这些的开发人员将[对 Quarkus](https://developers.redhat.com/blog/2019/10/24/bring-joy-to-development-with-quarkus-the-cloud-native-java-framework/) 如鱼得水，它简化了 80%常见用例的代码，同时提供了覆盖其余 20%的灵活性。

Quarkus 还为开发期间的快速迭代提供了实时编码，代码更改会自动立即反映在正在运行的应用程序中。

### 统一的命令式和反应式编程模型

开发人员可以选择最适合他们用例的正确编程模型，并轻松地将他们的代码与反应系统中的其他组件集成，如反应流，包括 T2 的 Vert.x 和 T4 的 Kafka、反应数据库 API 等等。

### 90 个标准和库

Quarkus 社区已经开发了超过 90 个扩展,为框架提供了额外的增强和集成，包括将应用程序编译成本地可执行文件的能力。

## 它是如何工作的？

传统的 Java 栈是针对单块应用程序优化的，在单块应用程序中，当应用程序启动时会发生大量的工作。这种动态行为在 Kubernetes 环境中产生了不必要的开销，在这种环境中，容器在相对较短的生命周期内快速地伸缩。Quarkus 将尽可能多的处理转移到构建阶段，如优化库框架、最小化依赖性和消除未使用的代码，以大大减少应用程序的启动时间和内存需求。

开发人员可以选择在 JVM 模式下部署他们的应用程序，或者在本机模式下编译和运行。与传统的 java 栈相比，这两种交付模式都提供了显著的性能改进。

## 为什么选择夸库斯的红帽建造？

Red Hat 长期以来一直是 Java 社区的领导者，并一直致力于通过开放的、社区驱动的创新来推动它向前发展。有了 Quarkus 的 [Red Hat build，开发者获得了完全支持的技术，包括活跃的社区、持续的更新和快速的发布节奏。Quarkus 发展迅速，Red Hat 致力于支持开发人员采用、部署和维护 Kubernetes-native Java 应用程序。](https://developers.redhat.com/products/quarkus/overview)

Quarkus 支持可通过 [Red Hat Runtimes](https://www.redhat.com/en/products/runtimes) 获得，它提供顶级集成产品、迁移工具和组件来创建云原生应用，同时还加快了开发和交付时间。Red Hat Runtimes 通过一组轻量级运行时和框架为开发人员和架构师提供了针对正确任务的正确工具选择，这些轻量级运行时和框架适用于高度分布式的云架构(如微服务),具有用于快速数据访问的内存缓存，以及用于现有应用程序之间快速数据传输的消息传递。

Quarkus 使用一个扩展框架来创建一个充满活力的生态系统，以与其他[红帽中间件](https://developers.redhat.com/middleware/)产品集成，如[红帽 AMQ 流](https://developers.redhat.com/products/amq/overview)(卡夫卡)[红帽融合](https://developers.redhat.com/products/fuse/overview)(骆驼 K)，以及[红帽过程自动化管理器](https://developers.redhat.com/products/rhpam/overview) (Kogito)。

Quarkus 还经过优化，可以在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 上运行，为可伸缩、快速和轻量级的应用程序提供理想的混合云应用程序开发环境。使用 Red Hat OpenShift 和包括 Quarkus 在内的云原生开发工具链，开发人员可以显著提高他们的生产力和推动创新的能力。

## 红帽支持

有了 Red Hat 订阅，您就可以访问由最有经验、最积极、最有知识的 Linux、Kubernetes 和中间件支持工程师组成的全球网络。当您在 Red Hat enterprise 产品上开发时，他们实际上可以扩展您的内部专业知识。支持工程师将在整个开发过程中为您提供建议和指导。

## 额外资源

查看这些资源，开始使用 Red Hat 的 Quarkus 构建:

*   [入门](https://developers.redhat.com/products/quarkus/getting-started)
*   [开始编码](https://code.quarkus.redhat.com/)
*   [文档](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.3/)
*   [支持的配置](https://access.redhat.com/articles/4966181)
*   [参与](https://quarkus.io/)

*Last updated: June 26, 2020*