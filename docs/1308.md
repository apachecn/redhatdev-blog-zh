# Devoxx 2017 红帽会议

> 原文：<https://developers.redhat.com/blog/2017/11/16/red-hat-sessions-devoxx-2017>

2017 年的传奇 [Devoxx](https://devoxx.be/) 大会结束了，一如既往，这是梦幻般的一周。

在比利时安特卫普举办，门票提前几个月就销售一空，这是 Java 社区的顶级活动之一。五天的时间里，研讨会、定期会议、BOF、ignite 会议，甚至午休时间的简短谈话都排得满满的——每个人都有自己的事情。

Devoxx 会场的超级舒适的电影院座位是传奇，但是如果你不能参加，你也不会错过任何东西，因为会议是现场直播的。但是更好的是:所有的录像都可以在 YouTube 上免费获得。

Red Hat 有十多位演讲者出席，因此 Devoxx 是我们展示最新项目的绝佳机会。我们的会议涵盖了软件开发的所有方面，从展示新的垃圾收集器，到 Java 编码模式和 Hibernate 等流行库的更新，再到与微服务相关的几次讲座，包括如何在 Kubernetes 和 OpenShift 上测试、保护和部署微服务。

为了方便起见，这里有一份由 Red Hat 工程师主持或共同发表的演讲汇编。让我们从 Aleksey Shipilev 在 [Shenandoah](http://openjdk.java.net/projects/shenandoah/) 的会议开始，这是一个新的用于大堆的超低暂停时间垃圾收集器:

https://www.youtube.com/watch?v=VCeHkcwfF9Q

马里奥·富斯科谈到了由 Java 8 lambdas 支持的[懒惰模式和数据结构](https://cfp.devoxx.be/2017/talk/YNZ-4718/Lazy_Java):

https://www.youtube.com/watch?v=84MfG4tp30s

## 库和 API 更新

在库和 API 更新方面，Sanne Grinovero 对 [Hibernate 生态系统](http://hibernate.org/)的当前状态做了一个[精彩的概述](https://cfp.devoxx.be/2017/talk/ZON-7778/Hibernate_you_know_it..._but_actually_you_don't)，包括关于 Hibernate ORM(最流行的 Java 对象关系映射工具)、Hibernate Search(全文搜索的集成)、Hibernate OGM(NoSQL 商店的对象映射)等的最新消息:

https://www.youtube.com/watch?v=mJDqxfXyNdM

大量项目使用它来编写参考指南、操作指南等。， [Asciidoctor](http://asciidoctor.org/) 是*在文档方面的*首选解决方案。Alex Soto 向[提供了这个伟大工具的更新](https://cfp.devoxx.be/2017/talk/TWI-5560/Asciidoctor:_New,_Noteworthy,_and_Beyond)，包括最近的添加、扩展和对未来计划的展望:

https://www.youtube.com/watch?v=T7RVT2_ntRU

我有机会[展示了](https://cfp.devoxx.be/2017/talk/TKL-4941/Bean_Validation_2.0_-_you%E2%80%99ve_put_your_annotations_everywhere!)[豆验证 2.0](http://beanvalidation.org/) 专家组(JSR 380)的作品。在 15 分钟的简短演讲中，我展示了新的验证特性，比如对容器元素的约束，包括一个使用 JavaFX 的简短演示:

https://www.youtube.com/watch?v=GdKuxmtA65I

Bean Validation 2.0 是 Java EE 8 的一部分，您可能已经听说了最近发布的开源 Java EE 平台并将其转移到 Eclipse Foundation 的消息。WildFly 和 JBoss EAP 的工程经理 Dimitris Andreadis 参加了围绕这些发展的小组讨论:

https://www.youtube.com/watch?v=HRNskFH1MoU

## 微服务开发、测试和部署

微服务及其生命周期的不同方面以及相关模式是许多讲座的主题。永不停歇的 Edson Yanaga 做了一个关于微服务数据模式的[精彩演讲](https://cfp.devoxx.be/2017/talk/ZPQ-4079/Microservices_Data_Patterns:_CQRS_&_Event_Sourcing)，这个话题通常没有得到应有的关注:

https://www.youtube.com/watch?v=eyf2Fs7GBo0

Debezium 是 Red Hat 的一个相当新的开源项目，支持 Edson 演讲中描述的一些模式。它捕获数据库中的更改，并将带有更改数据的事件推送到 Apache Kafka 中。在[这个演讲](https://cfp.devoxx.be/2017/talk/EXR-3680/Streaming_Database_Changes_with_Debezium)中，我首先讨论变更数据捕获(CDC)的一般概念，然后展示如何使用 Debezium 实现 CDC:

https://www.youtube.com/watch?v=IOZ2Um6e430

很多时候，我们没有那么幸运从零开始一个新的架构，而是需要发展现有的系统。在 Edson 的这个[深度现场编码会议](https://cfp.devoxx.be/2017/talk/DII-1775/Slice_&_Dice_your_Monolith_with_Domain-Driven_Design_)中，您可以了解如何使用领域驱动设计来分割您的整体:

https://www.youtube.com/watch?v=TYgHtZhS1jI

当然，微服务也需要适当的测试。这就是 Alex 和 Andy Gumbrecht(来自 Tomitribe)主持的本次会议的切入点:

https://www.youtube.com/watch?v=mH9TEXhmmwc

亚历克斯并没有就此罢休。在[的另一个演讲](https://cfp.devoxx.be/2017/talk/UDF-1134/Deploy_microservices_with_Certainty)中，他解释了如何处理微服务架构中的 API 变化，以及如何使用消费者驱动的契约模式来防止意外破坏下游应用:

https://www.youtube.com/watch?v=ZyZP9EpTr30

身份和访问管理是软件开发的一个方面，通常被认为是复杂和困难的。但这并不一定。在 Sebastien Blanc 的[编码会议](https://cfp.devoxx.be/2017/talk/IJH-3966/Easily_secure_and_add_Identity_Management_to_your_Spring_Boot_applications)中，了解如何使用 [Keycloak](http://www.keycloak.org/) 保护微服务:

https://www.youtube.com/watch?v=3I4TXPxCCVE

## Kubernetes 和 OpenShift

为了部署微服务，轻量级容器在最近几年里越来越受欢迎。Kubernetes 和 Red Hat 的 [OpenShift 容器平台](https://www.openshift.com/)帮助编排分布式微服务架构所需的容器。

在这个简短的演讲中，Bilgin Ibryam 展示了 Kubernetes 如何用新的原语和抽象来补充 Java，以创建分布式应用程序:

https://www.youtube.com/watch?v=ERSGc8OzJmw

在[的另一个演讲](https://cfp.devoxx.be/2017/talk/KWL-6615/SOLID_Principles_for_Cloud_Native_Containers)中，Bilgin 讨论了容器化应用程序为了成为云原生公民应该遵守的原则和指导方针:

https://www.youtube.com/watch?v=4n9N3lvqySk

最后，Marek Jelen [对 OpenShift 进行了概述](https://cfp.devoxx.be/2017/talk/SGB-4152/OpenShift_-_the_power_of_Kubernetes_for_developers)，以及它对软件工程师的优势。当然，里面也有现场演示:

https://www.youtube.com/watch?v=7aAKeCfdrNI

我们在 Devoxx 2017 的演讲到此结束。

这个会议是一个极好的机会，可以与我们项目的用户取得联系，了解他们的需求，回答问题和交流想法，与同事们交流，会见来自 Java 社区的老(和新)朋友，并相互学习。

期待 2018 年在安特卫普或另一场全球 Devoxx 大会上再次见到您！

*Last updated: September 3, 2019*