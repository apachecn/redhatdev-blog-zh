# 管理使用 Istio 服务网格部署的 API

> 原文：<https://developers.redhat.com/blog/2019/04/30/manage-your-apis-deployed-with-istio-service-mesh>

随着[微服务](https://developers.redhat.com/topics/microservices/)架构的兴起，企业正在寻找一种连接、保护、控制和观察其微服务的方法。目前，[像 Istio](https://developers.redhat.com/topics/service-mesh/) 这样的服务网络是实现这一目标的最佳选择。

*   **连接** : Istio 可以智能控制服务之间的流量，进行一系列测试，并通过蓝/绿部署逐步升级。
*   **安全**:通过服务间通信的托管认证、授权和加密，自动保护您的服务。
*   **控制**:应用策略并确保它们被强制执行，资源在消费者之间公平分配。
*   **观察**:通过丰富的自动跟踪、监控和记录您的所有服务，了解发生了什么。

此外，如[“分布式微服务架构:Istio、托管 API 网关和企业集成”](https://developers.redhat.com/blog/2019/03/12/distributed-microservices-architecture-enterprise-integration-istio-and-managed-api-gateways/)中所述，服务网格并不能缓解对 API 管理解决方案的需求。服务网格管理服务和它们之间的连接，而 API 管理解决方案管理 API 和它们的消费者。在本文中，我将描述如何使用 Istio 的 Red Hat Integration adapter 管理 API。

Red Hat Integration 提供了一种 API 管理功能，允许公司围绕他们的 API 建立一个消费者生态系统，然后从中获得新的收入。

*   **可见性**:深入了解您的 API 使用情况(您的 API 最常用的方法、最常用的应用程序、API 健康状况等。).
*   **控制**:管理客户端应用及其消耗(配额、SLA、定价等)。).
*   **货币化**:通过按使用量定价推动新的收入。
*   **混合 API 管理**:将 API 连接到任何地方(本地、SaaS 或各种云提供商)，从一个位置管理它们。
*   **API 生命周期**:从想法到实现，从 API 程序的开始到整个公司的大规模管理(参见“[完整 API 生命周期管理:初级读本](https://developers.redhat.com/blog/2019/02/25/full-api-lifecycle-management-a-primer/)”了解更多信息)。

## 当 API 遇到服务网格时

最新版本的 Red Hat Integration 添加了一个 Istio 适配器，它将 Istio 服务网格连接到它，这样您就可以将服务网格中的任何服务升级到全功能 API。

该适配器为 Istio 服务网格带来了 Red Hat 集成的 API 管理功能:

*   开发人员自助服务和入职
*   API 文档
*   货币铸造
*   使用分析

在 Istio 中，适配器是一个可插拔的组件，处理服务网格的各个方面。例如，适配器可以处理诸如授权和遥测之类的方面。Red Hat 集成适配器插入混音器组件来验证 API 密钥(授权)和报告使用情况(遥测)。

![](img/a261bf19263f0f184fca9b415959aeb5.png)

## 辅导的

在本文的其余部分，我将描述使用 Red Hat 集成适配器的第一步。这些步骤假设 Istio 已经[安装在你的 Minishift 实例](https://maistra.io/docs)或红帽 OpenShift 集群上的[，以及](https://docs.openshift.com/container-platform/3.11/servicemesh-install/servicemesh-install.html)[红帽集成](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.4/html/infrastructure/onpremises-installation)。

在管理服务网格中的 API 之前，我们需要在其中部署一些示例组件:首先部署书店应用程序，如下所述:

```
oc new-project bookstore
oc adm policy add-scc-to-user privileged -z default -n bookstore
oc adm policy add-scc-to-user anyuid -z default -n bookstore
oc apply -n bookstore -f https://raw.githubusercontent.com/Maistra/bookinfo/master/bookinfo.yaml
oc apply -n bookstore -f https://raw.githubusercontent.com/Maistra/bookinfo/master/bookinfo-gateway.yaml

```

确认书店应用程序正在使用:

```
$ export GATEWAY_URL=$(oc get route -n istio-system istio-ingressgateway -o jsonpath='{.spec.host}')
$ curl -s http://$GATEWAY_URL/api/v1/products |jq
[
  {
    "descriptionHtml": "<a href=\"https://en.wikipedia.org/wiki/The_Comedy_of_Errors\">Wikipedia Summary</a>: The Comedy of Errors is one of <b>William Shakespeare's</b> early plays. It is his shortest and one of his most farcical comedies, with a major part of the humour coming from slapstick and mistaken identity, in addition to puns and word play.",
    "id": 0,
    "title": "The Comedy of Errors"
  }
]

```

如您所见，书店应用程序公开了一个尚未被管理的 API(任何人都可以自由查询)。

在下面的视频中，了解如何使用 Istio 的 Red Hat Integration adapter 管理该 API。

[https://www.youtube.com/embed/fV9z_zq7YLs?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/fV9z_zq7YLs?autoplay=0&start=0&rel=0)

*Last updated: June 3, 2022*