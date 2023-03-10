# 由云原生 Java 支持的事件驱动的业务自动化

> 原文：<https://developers.redhat.com/blog/2019/09/23/devnation-live-event-driven-business-automation-powered-by-cloud-native-java>

Kogito 是一个新的 [Java](https://developers.redhat.com/developer-tools/java) 工具包，基于 Drools 和 jBPM，是为了给 [Quarkus](https://developers.redhat.com/courses/quarkus/getting-started/) 世界带来规则和流程。本文结尾的 [DevNation Live tech talk](https://youtu.be/6lZ2d7ZDpFk) 解释了如何使用 Kogito 构建云就绪、事件驱动的业务应用程序，其中包括一个实现复杂领域业务逻辑的演示。

Kogito 本身被定义为一个云原生业务自动化工具包，帮助您构建智能应用程序。它不仅仅是一个业务流程或单个业务规则，而是一系列业务规则，并且基于久经考验的能力。

### 专注于业务领域

在 Kogito 生态系统中，或者说在你用 Kogito 构建的方式中，最重要的事情之一是关注业务领域而不是技术本身。

![Kogito business](img/3013d48357fb7dafe278f78ab9f2bfee.png)

在示例用例中，我们有一家新成立的旅行社，名为 Kogito Travel Agency，希望增加其在线业务。在这个例子中，我们有一个旅行者、一个签证官和一个旅行助理，他们都希望在需要对等待他们的某个操作做出反应时得到通知。就整体架构而言，我们有两项主要服务:Kogito 旅行社服务和 Kogito 签证服务。那些是微服务。

应用程序的主干实际上是作为服务公开的业务流程。流程本身有几个您需要遍历的节点，这些节点定义了应用程序的实际构建方式。然后，我们可以使用“流程组合”将实际的业务流程分成多个步骤或更小的部分。

### 消息事件

负责签证申请的另一个服务仅由消息事件启动，该消息事件与 Kafka 主题相关联。当 Kafka 主题的消息出现时，它将创建该过程的新实例。

![Kogito event process](img/8efeedd39fbdab6c4751be514cb26825.png)

我们的想法是开始进一步增强流程，这样这些事件就可以产生与业务相关的其他事件。

最后一步，完成差旅申请。但是，由于一切都是领域驱动的和业务相关的，我们现在可以利用 Grafana 和我们公开的度量标准。如您所见，我们不仅有关于旅行请求的基本信息，还有多少是开放的、取消的或完成的，等等。我们还有关于调用了什么服务或者应用了什么规则的间接信息。

![Kogito metrics](img/eaa523886dc13c1768b65a5abb42ff95.png)

因此，现在我们不仅可以获得与业务相关的信息，还可以获得特定于领域的信息。

*观看下面完整的 [DevNation 现场技术演讲](https://developers.redhat.com/devnation/)。开发现场技术讲座由创造我们产品的红帽技术专家主持。这些会议包括真实的解决方案、代码和示例项目来帮助您开始。在这个演讲中，由 Red Hat 的首席软件工程师 Maciej Swiderski 和首席开发人员布道者 [Burr Sutter](https://developers.redhat.com/node/204405/) 主持，您将了解使用 [Kogito](https://developers.redhat.com/blog/2019/08/29/kogito-for-quarkus-intelligent-applications/) 、Quarkus 等工具实现事件驱动的业务自动化。*

[https://www.youtube.com/embed/6lZ2d7ZDpFk?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/6lZ2d7ZDpFk?autoplay=0&start=0&rel=0)

在演示中获得更多细节:[https://github.com/mswiderski/kogito-travel-agency-tutorial](https://github.com/mswiderski/kogito-travel-agency-tutorial)

### **了解更多信息**

加入我们即将到来的[开发者活动](https://developers.redhat.com/events/)，看看我们收集的[往届开发现场技术讲座](https://developers.redhat.com/devnation/?page=0) [。](https://developers.redhat.com/events/)

*Last updated: July 1, 2020*