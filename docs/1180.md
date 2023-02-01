# Istio 如何简化微服务的生产和部署

> 原文：<https://developers.redhat.com/blog/2018/04/17/istio-dark-launch-secret-services>

对于间谍来说，危险是很大的，但当涉及到部署软件时，无聊更好。你可以通过使用 Istio 和 OpenShift 以及 Kubernetes 来轻松地将你的[微服务](https://developers.redhat.com/topics/microservices/)投入生产，这是一件好事。

这是我关于 Istio、Service Mesh、Red Hat OpenShift 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 系列的第七部分。

**查看本系列的文章:**

*   第 1 部分:[Istio 服务网格介绍](https://developers.redhat.com/topics/service-mesh/)
*   第 2 部分: [Istio 路线规则:告知服务请求去哪里](https://developers.redhat.com/blog/2018/03/13/istio-route-rules-service-requests/)
*   第 3 部分: [Istio 断路器:如何处理(池)弹出](https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection/)
*   第 4 部分: [Istio 断路器:当出现故障时](https://developers.redhat.com/blog/2018/03/27/istio-circuit-breaker-when-failure-is-an-option/)
*   第五部分: [Istio Tracing &监控:你在哪里，速度有多快？](https://developers.redhat.com/blog/2018/04/03/istio-tracing-monitoring/)
*   第 6 部分: [Istio 混沌工程:我打算这么做](https://developers.redhat.com/blog/2018/04/10/istio-chaos-engineering/)
*   第 7 部分:【Istio 如何简化微服务的生产和部署
*   第 8 部分: [Istio 智能金丝雀发布:投产](https://developers.redhat.com/blog/2018/04/24/istio-smart-canary-launch/)
*   第 9 部分: [Istio 出口:通过礼品店出口](https://developers.redhat.com/blog/2018/05/01/istio-egress-exit-through-the-gift-shop/)
*   第十部分: [Istio 服务网格博客系列回顾](https://developers.redhat.com/blog/2018/05/07/istio-service-mesh-blog-series-recap/)

## 秘密服务的 Istio 黑暗发射

当涉及到部署软件时，任何可以最小化风险的方法都值得评估。并行运行是测试你下一个版本的一种非常强大和成熟的方法，Istio 允许你使用**secret services**——你的微服务的一个看不见的版本——而不干扰生产。这个听起来很酷的术语是**黑暗发射**，这是由另一个听起来很酷的想法实现的，**流量镜像**。

我使用术语“部署”而不是“发布”,因为您应该能够根据自己的意愿随时部署和使用您的微服务。微服务应该能够接受和处理流量，产生结果，并有助于日志记录和监控，而不一定要发布到产品中。部署和发布软件的过程并不总是相同的。您可以在需要时进行部署，并在准备就绪时发布。

考虑以下 Istio 路由规则，该规则将所有 HTTP 请求定向到建议微服务的版本 1，同时将请求镜像到版本 2。

**注:**本系列中的所有例子，包括图 1，都来自我们的 [Istio 教程 GitHub repo](https://github.com/redhat-developer-demos/istio-tutorial#mirroring-traffic-dark-launch) 。

注意底部附近的`mirror:`标签。这定义了请求镜像。是的，真的就这么简单。现在，当您的生产系统(v1)正在处理请求时，镜像(精确复制)请求被异步发送到 v2。这使您可以在不中断生产的情况下，使用真实世界的数据和真实世界的容量来查看 v2 的运行情况。

**注意:**您应该在 v2 代码中考虑任何影响数据存储的请求。虽然请求镜像是透明的并且易于实现，但是如何处理它仍然是一个问题。

## 下一步是什么？

这是这个由十部分组成的系列中最短的一篇博文，因为实现非常容易。同样，我们可以实现这个黑暗的启动/请求镜像特性，而不需要对我们的源代码做任何修改。

如果不镜像您的请求，而是智能地将其中一些路由到 v2(可能是 1%或某个用户组)，会怎么样？在扩展它处理的请求的百分比之前，您可以看到它是否工作。那太好了。如果失败了，你可以很快退出，回到 v1。如果成功，您可以继续将更多的工作负载转移到 v2，直到它达到 100%的请求。

让我们在下一篇文章[Istio Smart Canary Launch:Easing to Production](https://developers.redhat.com/blog/2018/04/24/istio-smart-canary-launch/)中进一步探讨这个问题。

*Last updated: October 26, 2022*