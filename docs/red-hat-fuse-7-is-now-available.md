# 红帽保险丝 7 现已上市

> 原文：<https://developers.redhat.com/blog/2018/06/04/red-hat-fuse-7-is-now-available>

红帽 Fuse 7(原名红帽 JBoss Fuse)现已正式上市。这种云原生分布式解决方案允许开发人员轻松开发、部署和扩展集成应用。架构师可以使用 Red Hat Fuse 组合和编排[微服务](https://developers.redhat.com/topics/microservices/),为系统引入敏捷性。在此版本中，Fuse 还支持集成专家和业务用户使用自助式低代码平台提高工作效率。有了这个新的敏捷集成解决方案，企业现在可以更快地与合作伙伴进行更广泛的协作。

你可以在这里下载:[https://developers.redhat.com/products/fuse/download/](https://developers.redhat.com/products/fuse/download/)。

# 保险丝 7 里有什么？

[![](img/6ef5d10d1f730a123494a9e3028ad103.png)](http://2.bp.blogspot.com/-9E79_vFzxts/Ww110GhueiI/AAAAAAAAFiA/JFISyFzIR4o7ZT-X0tNUmhG74vzPZ3owgCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-29%2Bat%2B11.45.33%2BAM.png)

骆驼仍然是保险丝中的核心部件。由于集成解决方案的复杂性，基于模式的集成解决方案对于防止开发人员陷入复杂、混乱和意大利面条般的集成逻辑至关重要。因此，预构建的组件和预定义的模式总是开发集成解决方案的方式，不管它们是在本地还是在云中。

Narayana 是现在的事务技术，让您为处理更复杂的分布式事务做好准备。

Jetty 已被弃用，取而代之的是一个更加精简的轻量级 web 容器:Undertow。

刚开始使用 Fuse 的人向我提出的最常见的问题之一是，“为什么您需要有这么多不同的运行时选项？”这是因为我们相信开发人员应该能够选择最适合他们的方式，例如:

*   微服务 Spring Boot
*   OSGi 情人的卡拉夫 4
*   面向 Java EE 开发者的 Red Hat JBoss 企业应用平台

# 云-原生集成平台

构建云原生应用可能很复杂。集成它们以及使集成解决方案本身为云就绪更具挑战性。您需要将整个软件生命周期作为一个整体来处理，例如，创建占用空间更小的应用程序，自动化部署流程，以及在分布式环境中监控应用程序。由于 Fuse 7 平台的特性和技术处理了这种复杂性，集成团队现在可以专注于最重要的事情:集成应用程序的业务逻辑。

**轻量级开发**

当谈到将集成迁移到云中时，构建小尺寸的微服务是最基本的事情。Spring Boot 和 Karaf 运行时都是轻量级的，支持微服务。开发人员可以利用 Camel 中基于模式的集成，插件可以帮助开发人员将他们的集成应用程序打包成一个容器，为云做好准备。

[![](img/21720534810abc1a7523260a977d86ac.png)](http://4.bp.blogspot.com/-t0lL6-5-Z1w/Ww6lbUhGUAI/AAAAAAAAFiM/TJ5EHgC6adQLm6Fw1BEjP8yWmnlbXLnogCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B9.21.14%2BAM.png)

**混合部署**

部署后，Fuse 7 帮助开发人员将应用程序打包成不可变的映像。根据您的版本和部署策略，这些映像可以被标记并发送，以便在环境的不同阶段进行部署。该平台还负责应用程序的可用性、可伸缩性和弹性；放大和缩小应用程序只需要在用户界面上点击几下。或者，如果你喜欢自动化，你可以。该应用程序不仅运行在云中，还可以作为单个独立的应用程序运行。这三个运行时都可以在两种场景中运行。

[![](img/fd18631a766f29b9cfd2c36c06b06898.png)](http://4.bp.blogspot.com/-6l0NHGizR6Q/Ww8HSQd1wMI/AAAAAAAAFik/v31J_1Hl_K8ZZV2s2XS5h3sIWRg6zRSpgCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B4.18.44%2BPM.png)

**提供集中应用视图的管理**

管理一个分布式云环境是一场噩梦:您现在拥有数千个单独运行的实例，而不是几个实例。Fuse 在云环境中为您提供了一个集中的查看控制台，因此定位所有正在运行的 Fuse 实例会容易得多。此外，每个运行的应用程序都由 Red Hat OpenShift 管理，它为管理云中的应用程序提供了全面的功能。

[![](img/a53374ad56889068a3b1ac59ce63b3bd.png)](http://2.bp.blogspot.com/-5vuMgeWbtvA/Ww6nYebQzZI/AAAAAAAAFiY/lmGgxlc1pr0zVc6Dec4b9bawRN1-fLfvACK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B9.29.54%2BAM.png)

**面向公民集成商的低代码集成**

Fuse 7 中引入的另一个新功能是红帽 Fuse 在线自助服务平台，它允许公民集成商自行构建集成服务。Fuse 7 允许公司拥有一个单一的标准化平台，而不是针对不同用户特征的多项技术，从而允许开发人员和公民集成商之间的最大协作。凭借直观的界面和高度可定制的组件功能，市民集成商可以快速交付集成解决方案，而无需等待开发人员的实施。开发人员可以专注于开发更复杂的集成逻辑，并将其打包到平台中，以便市民集成商可以随时重用。

当您登录到平台时，您将看到所有可用集成和连接器的概述，以及正在运行的集成的状态。

[![](img/f42bdc687ffbc25f42b69f13a1d1df1d.png)](http://1.bp.blogspot.com/-VScVy1BY8A8/Ww8xTOYTJcI/AAAAAAAAFiw/dkQqgW8WAZcSg2Qg3w53-ZidQFQRch_yQCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B7.16.51%2BPM.png)

集成允许集成者通过简单地拖放预先构建的连接器或者通过输入源和输出目标数据之间的映射来创建应用程序。

[![](img/43d6feab3cc716f0c11a71d75abbf981.png)](http://3.bp.blogspot.com/-LNWmeWDvtpI/Ww8yN5OWqbI/AAAAAAAAFi8/3CPUPIFR5-sDOQhw8pIgxBGstdeVPRiZgCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B7.20.38%2BPM.png)

[![](img/7c614ac0216f767ca9cc1d24243d8b19.png)](http://1.bp.blogspot.com/-BY8QWnfJUxg/Ww8yPOmnLCI/AAAAAAAAFjE/jp1cS7EAggsysvI-UDl6dx8LCQV6MXm2gCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B7.20.48%2BPM.png)

当需要更复杂的过程时，您还可以通过简单地将项目包导入到系统中来添加定制的连接器。定制的连接器可以由内部开发人员创建，或者从我们的开源社区在线提供。

[![](img/dd1fb4864d23467b6a696e77ccdbcd3a.png)](http://4.bp.blogspot.com/-EGRk1K_ScWs/Ww83KV31FUI/AAAAAAAAFjU/0hyM9rqiO9UOAfbmY2OLzL23EbYqpYAXQCK4BGAYYCw/s1600/Screen%2BShot%2B2018-05-30%2Bat%2B7.43.15%2BPM.png)

# 现在试试保险丝 7

您可以在此试用该平台的新版本:

[https://www.redhat.com/en/explore/fuse-online](https://www.redhat.com/en/explore/fuse-online)

你还在等什么？立即开始您的新 Fuse 7 冒险。

*Last updated: November 15, 2018*