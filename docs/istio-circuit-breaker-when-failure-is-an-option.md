# Istio 断路器:当故障成为一种选择时

> 原文：<https://developers.redhat.com/blog/2018/03/27/istio-circuit-breaker-when-failure-is-an-option>

“失败不是一个选项”这句话被虚张声势地抛来抛去，好像一个人仅凭意志力就能让事情成功。但事实是，事情最终会失败。一切。那么，您如何处理微服务不可避免的故障呢？嗯，通过组合容器、 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 、 [Red Hat OpenShift、](https://developers.redhat.com/products/openshift/overview/)和 [Istio](https://developers.redhat.com/topics/service-mesh/) ，我们可以跳过夸张的炫耀，让系统处理事情，并在晚上睡个好觉。

【这是我十周**[Istio 服务网系列](https://developers.redhat.com/topics/service-mesh/)** 介绍的第四部分。我之前的文章是[第三部分:Istio 断路器:如何处理(池)弹射](https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection/)。]

Istio 再一次提供了一种流行的、久经考验的技术的基础:断路器模式。

就像电路中的断路器一样，软件版本允许关闭流向服务的流量。在端点不能正常工作的情况下，电路断开。端点可能已经失败或者可能太慢，但是它表示相同的问题:这个容器不工作。

滞后的性能尤其麻烦:不仅延迟会通过任何调用服务级联并导致整个系统滞后，而且对已经很慢的服务重试只会使情况变得更糟。

## 突破:很好

断路器是控制流向端点的流量的代理。如果端点失败或太慢(基于您的配置)，代理将打开到容器的电路。在这种情况下，由于负载平衡，流量被路由到其他容器。电路在预先配置的睡眠窗口(假设两分钟)内保持打开，在此之后，电路被视为“半开”。尝试的下一个请求将确定电路是否移动到“闭合”(此时一切都再次工作)，或者它回复到“断开”并且睡眠窗口再次开始。这是断路器的简单状态转换图:

![](img/453abba608fd53d8722c943fe7b95ae6.png)

需要注意的是，可以说这都是系统架构层面的。在某种程度上，您的应用程序需要考虑断路器模式；常见的响应包括提供默认值或(如果可能的话)忽略服务的存在。隔板模式解决了这个问题，但这超出了本文的范围。

## Istio 断路器正在运行

首先，我在 OpenShift 中推出了两个版本的微服务“建议”。版本 1 运行正常，而版本 2 有一个内置的延迟。这模拟了一个缓慢的服务器。使用工具[围攻](https://github.com/JoeDog/siege)我们可以观察结果:

```
siege -r 2 -c 20 -v customer-tutorial.$(minishift ip).nip.io
```

![](img/760fa5b0f6706796a5388d5e2b63e582.png)

一切都在运转，但代价是什么？乍一看，100%的可用性似乎是一个胜利，但仔细观察就会发现。最长的交易耗时超过 12 秒。这可不算快。我们需要设法避免这个瓶颈。

我们可以使用 Istio 的断路器功能来避免这些缓慢的容器。以下是实现断路器的配置文件示例:

![](img/ecf0159af598997d46d89a531cd05397.png)

最后一行“httpMaxRequestsPerConnection”意味着如果试图对一个已经有连接的容器进行第二次连接，电路将打开。因为我们有意让我们的容器模拟一个缓慢的服务，所以它偶尔会遇到这种情况。发生这种情况时，Istio 将返回一个 503 错误。这是另一次使用围攻的截屏:

![](img/35d323de30801f064ff8bef0c04ffad9.png)

## 电路断了；现在怎么办？

在不改变源代码的情况下，我们能够实现断路器模式。结合上周的博客文章( [Istio Pool Ejection](https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection/) )，我们可以消除缓慢的容器，直到它们恢复。在这个例子中，容器在被重新考虑之前被弹出两分钟(“sleepWindow”设置)。

请注意，您的应用程序响应 503 错误的能力仍然是您的源代码的一个功能。有许多处理开路的策略；你选择哪一个取决于你的具体情况。

* * *

### “Istio 简介”系列的所有文章:

*   第一部分:[Istio 介绍；它制造了一个网状的东西](https://developers.redhat.com/topics/service-mesh/)
*   第 2 部分: [Istio 路线规则:告知服务请求去哪里](https://developers.redhat.com/blog/2018/03/13/istio-route-rules-service-requests/)
*   第 3 部分: [Istio 断路器:如何处理(池)弹出](https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection/)
*   第 4 部分: [Istio 断路器:当出现故障时](https://developers.redhat.com/blog/2018/03/27/istio-circuit-breaker-when-failure-is-an-option/)
*   第五部分: [Istio Tracing &监控:你在哪里，速度有多快？](https://developers.redhat.com/blog/2018/04/03/istio-tracing-monitoring/)
*   第 6 部分: [Istio 混沌工程:我打算这么做](https://developers.redhat.com/blog/2018/04/10/istio-chaos-engineering/)
*   第七部: [Istio 黑暗发射:特勤局](https://developers.redhat.com/blog/2018/04/17/istio-dark-launch-secret-services/)
*   第 8 部分: [Istio 智能金丝雀发布:投产](https://developers.redhat.com/blog/2018/04/24/istio-smart-canary-launch/)
*   第 9 部分: [Istio 出口:通过礼品店出口](https://developers.redhat.com/blog/2018/05/01/istio-egress-exit-through-the-gift-shop/)
*   第十部分: [Istio 服务网格博客系列回顾](https://developers.redhat.com/blog/2018/05/07/istio-service-mesh-blog-series-recap/)

*Last updated: September 3, 2019*