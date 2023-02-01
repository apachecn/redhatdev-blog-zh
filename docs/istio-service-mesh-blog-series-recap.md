# Istio 服务网格博客系列摘要

> 原文：<https://developers.redhat.com/blog/2018/05/07/istio-service-mesh-blog-series-recap>

过去九周的博客文章介绍、解释和演示了 Istio 服务网格与 Red Hat OpenShift 和 Kubernetes 结合使用时的一些特性。这是本系列的最后一篇文章，是一个总结。

【这是我的 Istio 服务网系列 十篇 **[介绍的第十篇。我之前的文章是](https://developers.redhat.com/topics/service-mesh/)[第九部分:Istio Egress:通过礼品店退出](https://developers.redhat.com/blog/2018/05/01/istio-egress-exit-through-the-gift-shop/)。]**

[第一周](https://developers.redhat.com/blog/2018/03/06/introduction-istio-makes-mesh-things/)介绍了服务网格的概念。Kubernetes sidecar 容器的概念得到了解释和图解，它是贯穿整个博客文章的一个不变主题的开始:*你不必改变你的源代码*。

[第二周](https://developers.redhat.com/blog/2018/03/13/istio-route-rules-service-requests/)展示了 Istio 最基本、最核心的方面:路线规则。路由规则为 Istio 的其他功能打开了大门，因为您能够基于代码外部的 YAML 文件智能地将流量定向到您的微服务。在这篇文章中，还暗示了金丝雀部署模式。

[第三周](https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection/)展示了 Istio 实现池弹射的能力，与断路器模式配合使用。Istio 的一个强大特性是能够基于糟糕的性能(或者无性能)从负载平衡中删除一个 pod，这篇博文演示了这一点。

第四周曝光了断路器。前一周已经暗示过了，这篇文章提供了断路器和 Istio 模式实现的更详细的解释。同样，在不更改源代码的情况下，我们看到了如何通过 YAML 配置文件和一些终端命令来引导流量和处理网络故障。

[第五周](https://developers.redhat.com/blog/2018/04/03/istio-tracing-monitoring/)重点介绍了 Istio 中内置或易于添加的跟踪和监控功能。Prometheus、Jaeger 和 Grafana 等工具与 OpenShift 的扩展相结合，展示了如何轻松管理您的微服务架构。

[第六周](https://developers.redhat.com/blog/2018/04/10/istio-chaos-engineering/)从监控和处理错误切换到制造错误:故障注入。能够在不改变源代码的情况下将故障注入你的系统*是测试的一个重要部分。测试未受干扰的代码意味着您可以确信您没有添加任何“测试代码”，这些代码本身可能会导致问题。重要的东西。*

[第七周](https://developers.redhat.com/blog/2018/04/17/istio-dark-launch-secret-services/)发生了黑暗的转折。良好的...转向黑暗启动，这是一种部署模式，您可以在不中断系统的情况下部署代码并使用生产数据进行测试。使用 Istio 来分流流量是一个你可能经常使用的有价值的工具。能够在不影响您的系统的情况下使用实时生产数据进行测试是最有说服力的测试。

[第八周](https://developers.redhat.com/blog/2018/04/24/istio-smart-canary-launch/)以黑暗发布为基础，展示了如何使用 Canary 部署模型将新代码轻松投入生产，同时降低您的风险。Canary 部署(或“Canary Release”)并不新鲜，但是能够通过一些简单的 YAML 文件来实现它却很新鲜，这要感谢 Istio。

[第九周](https://developers.redhat.com/blog/2018/05/01/istio-egress-exit-through-the-gift-shop/)最后，演示了如何使用 Istio 通过 Istio Egress 访问集群之外的服务。这扩展了包含整个网络的能力。

## 你自己试试

过去的九周不是深度潜水，也不打算是。这个想法是为了介绍概念，激发兴趣，并鼓励你亲自尝试一下 Istio。零成本、 [Red Hat 开发人员 OpenShift Container Platform](https://www.openshift.com/products/container-platform/trial/) 和我们的 [Istio 教程](https://github.com/redhat-developer-demos/istio-tutorial)，加上我们的 [Service Mesh 微型网站](https://developers.redhat.com/topics/service-mesh/)上的其他可用资产，您拥有了零风险开始探索 OpenShift、Kubernetes、Linux containers 和 Istio 所需的所有工具。不要等待:拿起工具，今天就开始。

* * *

### “Istio 简介”系列的所有文章:

*   第 1 部分:[Istio 服务网格介绍](https://developers.redhat.com/topics/service-mesh/)
*   第 2 部分: [Istio 路线规则:告知服务请求去哪里](https://developers.redhat.com/blog/2018/03/13/istio-route-rules-service-requests/)
*   第 3 部分: [Istio 断路器:如何处理(池)弹出](https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection/)
*   第 4 部分: [Istio 断路器:当出现故障时](https://developers.redhat.com/blog/2018/03/27/istio-circuit-breaker-when-failure-is-an-option/)
*   第五部分: [Istio Tracing &监控:你在哪里，速度有多快？](https://developers.redhat.com/blog/2018/04/03/istio-tracing-monitoring/)
*   第 6 部分: [Istio 混沌工程:我打算这么做](https://developers.redhat.com/blog/2018/04/10/istio-chaos-engineering/)
*   第七部: [Istio 黑暗发射:特勤局](https://developers.redhat.com/blog/2018/04/17/istio-dark-launch-secret-services/)
*   第 8 部分: [Istio 智能金丝雀发布:投产](https://developers.redhat.com/blog/2018/04/24/istio-smart-canary-launch/)
*   第 9 部分: [Istio 出口:通过礼品店出口](https://developers.redhat.com/blog/2018/05/01/istio-egress-exit-through-the-gift-shop/)
*   第十部分: [Istio 服务网格博客系列回顾](https://developers.redhat.com/blog/2018/05/07/istio-service-mesh-blog-series-recap/)

了解有关 [Istio 以及服务网格如何改善 developers.redhat.com](https://developers.redhat.com/topics/service-mesh/)[的](https://developers.redhat.com)微服务的更多信息。

*Last updated: September 3, 2019*