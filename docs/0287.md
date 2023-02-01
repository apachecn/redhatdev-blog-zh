# 使用 Camel K 的事件驱动的无服务器应用程序

> 原文：<https://developers.redhat.com/blog/2020/11/17/event-driven-serverless-applications-with-camel-k>

开发技术讲座由创造我们产品的红帽技术专家主持。这些会议包括真正的解决方案加上代码和示例项目，以帮助您开始。在这次演讲中，你将从[尼古拉·费拉罗](https://developers.redhat.com/blog/author/nferraro/)、[卢卡·布尔加佐利](https://developers.redhat.com/blog/author/lburgazz/)和[伯尔·萨特](https://developers.redhat.com/blog/author/burrsutter/)那里了解事件驱动的无服务器应用和 Apache Camel K。

事件驱动的无服务器应用程序最近很流行。 [Knative](https://developers.redhat.com/topics/serverless-architecture) 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 为创建它们提供了不错的原语，但是如果你曾经尝试过超越“Hello World”的例子，你就会知道编写现实生活中的应用程序比预期的要困难得多。

您的应用程序被分解得越小，您就越需要更好的通信模式来管理所有固有的复杂性。Camel K 来了，这是一个专门为解决这些问题而创建的轻量级集成工具。Camel K 允许你使用漂亮的语言在一个[无服务器环境](https://developers.redhat.com/topics/serverless-architecture)中声明性地编排事件。它允许您将功能和服务连接到任何类型的外部数据源或数据宿，从企业服务到云服务或 SaaS。

[Camel K 基于 Apache Camel](https://camel.apache.org/camel-k/latest/index.html) ，这是最强大的开源集成框架，它利用 Knative 以无服务器的方式提供集成模式，允许您有效地创建真实的无服务器应用程序。我们将展示 Camel K 是如何工作的，并且通过代码示例，我们还将展示 Camel K 如何使用集成模式和 Apache Camel 提供的 300 多个组件来简化(几乎)任何东西的连接。

观看整个演讲:

[https://www.youtube.com/embed/hlUzLC71nAM?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/hlUzLC71nAM?autoplay=0&start=0&rel=0)

## 了解更多信息

加入我们即将到来的开发者大会，看看我们收集的[过去的开发技术演讲](https://developers.redhat.com/devnation/?page=0)。

*Last updated: January 11, 2021*