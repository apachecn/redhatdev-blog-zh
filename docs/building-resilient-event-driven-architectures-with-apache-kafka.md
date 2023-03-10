# 用 Apache Kafka 构建弹性事件驱动架构

> 原文：<https://developers.redhat.com/blog/2021/05/05/building-resilient-event-driven-architectures-with-apache-kafka>

尽管云原生计算已经存在了一段时间，但[云原生计算基金会](https://www.cncf.io/)始于 2015 年；计算机时代的一个世纪——不是每个开发人员都体验过处理分布式系统的“快乐”。旧的思维模式和架构系统已经让位于新的想法和新的问题。例如，连接到数据库并运行事务并不总是可能的(或不明智的)。数据库本身正在让位于事件和[命令查询责任分离(CQRS)](https://github.com/redhat-developer-demos/cloud-native-compass/blob/master/cqrs.md) 和[最终一致性](https://github.com/redhat-developer-demos/cloud-native-compass/blob/master/eventual-consistency.md)。两阶段提交被队列和[数据库传奇](https://github.com/redhat-developer-demos/cloud-native-compass/blob/master/sagas.md)取代，而单片被[微服务](https://github.com/redhat-developer-demos/cloud-native-compass/blob/master/microservice.md)、[容器](/topics/containers/)和 [Kubernetes](/topics/kubernetes/) 取代。“小而局部”的思维占据了主导地位。

现在将这与分布式处理的谬误结合起来，T2 事件驱动架构变得非常有吸引力。谢天谢地，有工具可以做到这一点。[阿帕奇卡夫卡](https://kafka.apache.org/)就是这些工具之一。

卡夫卡使事件处理成为可能；[Red Hat open shift Streams for Apache Kafka](/products/rhosak/overview)使事件处理变得简单。

## 为什么是事件？

随着向云计算的转移，架构师和开发人员被迫重新审视数据的处理方式。随着人们认识到即时更新并不总是必要的，关于及时性和数据紧迫性的问题也迎刃而解。另外，焦点转移到了有时(大多数时候；理想情况下*所有时间*都继续运行，即使所有部件都不工作。系统的目标从“绝不能失败”变成了“失败是不可避免的，所以我们需要处理它。”这导致了一种新的思维方式的兴起，包括断路器、多个数据库、事件等等。

事件驱动模型的美妙之处在于，您可以触发事件并继续，将结果留给“系统”您的代码不会等待四个数据库更新或一个对象通过 CDN 在全球传播。开发人员在处理事件时有一定的自由度。你要么把它们推出去，然后忘记它们，要么只是等待一个事件的到来，然后处理它。换句话说，你的代码是典型的单行道。低延迟，或者至少减少延迟，几乎是自动的。

和松散耦合的服务？从本质上来说，事件强制松散耦合的服务。API 就是事件。我的代码没有调用您的代码中的方法；我只是提供一个信息。您的代码如何处理它取决于您自己。

## 开发者为什么要关心卡夫卡？

作为一名开发人员，如果您想要一个有弹性的、有弹性的、高性能的系统，您将需要接受事件处理。这只是事实。

卡夫卡带来了一定程度的成熟以及一个强大的社区。阿帕奇卡夫卡屡试不爽。它被到处使用，周围有一个完整的生态系统。例如，Kafka 连接器允许您不用编写代码就可以处理一些事件。这对于开发人员来说很重要，因为:如果我知道我不需要写特定的代码，我就可以把精力放在我需要写的代码上。

撇开生意不谈；很好玩。

附注:事件与功能即服务(FaaS)完美匹配。

## 接下来呢？

获取编码。编写一些 PoC(概念验证)应用程序——或者使用我们的一个例子——做我们开发人员最擅长的事情:展示软件能做什么。

我建议看一下[这个关于 Apache Kafka](https://www.youtube.com/watch?v=7hp1MdnjH94&t=3s) 的 Red Hat OpenShift Streams 的简短而翔实的视频，然后继续您的分布式处理之旅。变得了不起只有四个步骤:

1.  获取 Apache Kafka 的 [Red Hat OpenShift Streams 的(免费)实例。](/products/rhosak/overview)
2.  获取 Red Hat OpenShift 的[开发者沙盒的(免费)实例。](/developer-sandbox)
3.  编写事件处理代码。
4.  利润。

*Last updated: October 14, 2022*