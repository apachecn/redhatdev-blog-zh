# 通往 Quarkus GA 之路:完成第一个受支持的 Kubernetes-native Java 堆栈

> 原文：<https://developers.redhat.com/blog/2020/06/04/the-road-to-quarkus-ga-completing-the-first-supported-kubernetes-native-java-stack>

这些年来，我在红帽度过了许多骄傲的时刻。例如，当我们发布 WildFly 的第一个版本时，当我们收购 Camel 团队时，当我们与其他供应商合作创建 Eclipse MicroProfile 时，Strimzi 团队为进入云计算原生计算基金会所做的巨大工作，我们整个 Red Hat 托管集成工作， [Kogito](https://quarkus.io/guides/kogito) ，等等。我觉得我几乎每周都在增加这个例子。

嗯，我现在可以用 Quarkus 的第一个产品发布来更新这个列表，正式名称是 Quarkus 的 [Red Hat build。(您还可以在](https://access.redhat.com/products/quarkus) [Quarkus 项目网站](https://quarkus.io/support/)上找到更多支持选项。Quarkus 出现在这个名单上并不奇怪。我想让一些人感到惊讶的是 Quarkus 现在只是一个产品。鉴于自 2019 年我们正式[推出 Quarkus 项目](https://developer.jboss.org/blogs/mark.little/2019/03/07/quarkus-is-here)以来的所有活动，你可能会认为它已经是一个产品。

## 生产用户帮助我们实现了这一目标

我们已经听到了许多将它投入到[生产](https://quarkus.io/blog/tag/user-story/)的用户，包括[沃达丰希腊](https://quarkus.io/blog/vodafone-greece-replaces-spring-boot/)、 [Talkdesk](https://quarkus.io/blog/talkdesk-chooses-quarkus-for-fast-innovation/) 和[汉莎技术](https://quarkus.io/blog/aviatar-experiences-significant-savings/)。这些公司也有一些很棒的名言:

*   Thorsten Pohl **，**Lufthansa Technik aviata 产品所有者自动化&平台架构师:“我们可以运行 3 倍密度的部署，而不会牺牲服务的可用性和响应时间”。
*   Roberto Cortez **，** Talkdesk 首席架构师:“当你采用 Quarkus 时，你从第一天起就会很有效率，因为你并不真的需要学习新技术。”
*   沃达丰希腊公司的 DXL 技术负责人克里斯特斯·索蒂里欧说:“Quarkus 似乎提供了我们需要的性能提升，同时拥有一个好的支持者(红帽)并依赖于久经考验的技术。”

## 社区把我们带到了那里

在任何开源项目的这个阶段，即使只有一个很好的参考，也是对项目开发人员及其社区工作的极大支持，所以 Quarkus 有这么多参考是非常了不起的。当你将此与大量的[社区兴趣](https://www.adam-bien.com/roller/abien/entry/supersonic_subatomic_java_ee_skimmedjars)结合起来，其中大部分反映在 Twitter 等社交媒体平台上，那么我认为可以公平地说，我们开发了其他开源项目没有的东西。

![@AdamBien Twitter post for Quarkus 1.4 Final](img/6bf1372cc27878ef92912467d53eb5c8.png)

![@agoncal Quarkus Twitter post](img/21ed405136b17f295d7b5dc7683e9c2d.png)

哦，让我们不要忘记 Quarkus 已经赢得的奖项，包括 [DEVIES](https://quarkus.io/blog/quarkus-wins-devies-award/) 。【T2![](img/4f397bbce725188475629db8b0e387bd.png)

我们中的许多人，包括我自己，已经详细地讲述了是什么让夸库与众不同，所以我尽量不重复太多。相反，我会立即跳到房间里的大象，说 Quarkus 的特别之处不是我们编译 Java 的事实。是的，这有助于减少执行占用空间和启动时间，但是这并不是什么灵丹妙药，因为我们多年来在尝试诸如 GNU 编译器 Java (GCJ)和 Avian 之类的编译技术时已经发现了这一点。正如之前多次描述过的，但经常被忽略的是，该团队重新架构了我们的许多流行项目，因此它们在 Kubernetes 环境中工作得很好——甚至在股票 OpenJDK 中也是如此。这是难题的关键部分，当通过 GraalVM 与编译后的 Java 结合时，Quarkus 将处于真正的 Kubernetes-native 堆栈的中心。

通过社区的努力，我们也看到了 Quarkus 在无服务器环境中的应用，因为它能够让应用程序启动并在几毫秒内准备好处理请求。具体来说，Quarkus 现在支持 [Azure Functions](https://quarkus.io/guides/azure-functions-http) 和 [Amazon Lambda](https://quarkus.io/guides/amazon-lambda) ，还会有更多的集成。是的，甚至还有围绕 Knative 的[夸夸其谈。很高兴看到其他的努力，如 Camel K](https://quarkus.io/guides/getting-started-knative)，即将在产品中出现，这将使大部分庞大的 Camel 生态系统能够在无服务器的环境中使用。

## 红帽运行时中的 quartus ga

我可以继续讲述我们在 Quarkus 周围看到的巨大的社区参与，已经利用 Quarkus 的其他项目和产品，如 Kogito，现在能够在云中运行更高密度的 Java 应用程序以及利用现有 Java 技能的好处，以及每个人正在做的许多其他事情。然而，这篇文章的真正目的是强调这样一个事实，即 [Quarkus 现在可以作为 Red Hat 的受支持产品](https://developers.redhat.com/products/quarkus/overview)使用，并具有所有这些功能。Quarkus 现在也是 [Red Hat Runtimes](https://www.redhat.com/en/products/runtimes) 的一部分，面向 [IBM 客户](https://developer.ibm.com/technologies/java/articles/deploy-reactive-quarkus-microservices-on-ibm-cloud-kubernetes-service/)也可以在 IBM 的 CloudPaks For Applications (CP4A)中获得。正如我将在后面讨论的，我们已经确保 Quarkus 在 OpenJDK 上运行良好，但是如果您在 IBM 的 zSeries 上运行，那么 OpenJ9 是您的首选 JVM，Quarkus 在那里也同样运行良好。

Quarkus 团队和社区的发展速度非常快，所以经常很难跟上最新的发展。但是，在 Quarkus GA 中有几个需要考虑的问题:

*   从 Quarkus 开始，我们就一直确保反应式编程是核心，而不是试图翻新它，这很少能很好地工作。对我们来说幸运的是，我们也参与了领先的反应式编程 Java 框架 [Eclipse Vert.x](https://vertx.io) 很多年了，这两个团队将 Vert.x 构建到 Quarkus 中。然而，最近，这两个团队还合作了 [SmallRye 兵变项目](https://smallrye.io/smallrye-mutiny/)，该项目基于从 Vert.x 吸取的经验教训，并在其基础上简化了各种概念。
*   Eclipse MicroProfile 是 Red Hat 整体上的一项关键工作，自创建以来，我们一直处于这项工作的最前沿。最新和最棒的版本 3.3 以及相关的规范是第一个 Quarkus 产品的一部分。
*   有许多 Spring 开发者热衷于使用 Quarkus，我们已经和他们中的许多人合作了一段时间，包括前面提到的沃达丰希腊公司。事实上，我们已经付出了很多努力，试图确保这些开发人员能够[利用 Quarkus](https://quarkus.io/blog/quarkus-for-spring-developers/) 的技能和应用，在这个版本中，我们增加了 Quarkus 应用在运行时从 Spring Cloud Config Server 中[读取配置属性的能力。](https://quarkus.io/guides/spring-cloud-config-client)
*   Red Hat 在消息传递实现方面有着丰富的历史，包括 HornetQ 和 ActiveMQ。从一开始我们就参与其中的一个关键标准是[绿洲 AMQP](https://www.oasis-open.org/committees/amqp/) ，并且有许多该标准的实现，包括[我们自己的](https://access.redhat.com/documentation/en-us/red_hat_enterprise_mrg/3/html/messaging_programming_reference/amqp___advanced_message_queuing_protocol)。AMQP 社区中的一个重要项目是 [Apache Qpid](https://qpid.apache.org) ，在 Quarkus 产品发布中，我们包含了对 Qpid JMS 适配器的[支持，因此您的微服务可以使用该标准进行通信。](https://quarkus.io/guides/jms#qpid-jms-amqp)
*   任何与事务有关的东西，比如 MicroProfile 的 [LRA、](https://github.com/eclipse/microprofile-lra)和一直在推动它的 [Narayana](https://narayana.io) 项目。

所以我希望这足以说明 Quarkus 产品的发布有多重要。最后，我想再次祝贺整个 Quarkus 产品团队和社区。无论您是致力于核心 Quarkus 功能、提交特性请求，还是撰写关于使用它的经验的博客，您都帮助世界上第一个 Kube-native Java 栈从项目到产品。拿着应得的弓！

*Last updated: June 25, 2020*