# 观察您的 Istio 微服务网格如何处理 Kiali

> 原文：<https://developers.redhat.com/blog/2018/09/20/istio-mesh-visibility-with-kiali>

Istio 服务网格是构建服务网格的强大工具。如果你还不了解 Istio，可以看看[介绍 Istio](https://developers.redhat.com/blog/2018/03/06/introduction-istio-makes-mesh-things/) 系列文章或者下载电子书[介绍微服务的 Istio 服务网格](https://developers.redhat.com/books/introducing-istio-service-mesh-microservices/)。

Istio 的强大是以配置和运行时的复杂性为代价的。为此， [Kiali](https://www.kiali.io/) 项目提供了网格和网格中服务的可观察性。Kiali 将网格与其服务和工作负载可视化。它指示网格的健康状况，并显示有关应用的配置选项的提示。然后，您可以深入查看单个服务或设置的详细信息。

这篇文章描述了如何使用 Kiali 来观察您的 Istio 服务网格中的微服务正在做什么，验证 Istio 配置，并查看任何问题。

## 可视化 Istio 服务网格

这里是一个 [Istio 服务网格](https://developers.redhat.com/topics/service-mesh/)的 Kiali 图形视图:

[![Istio service mesh graph in Kiali](img/67feb9c3978abf2057e48fd4bb315890.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Bildschirmfoto-2018-09-18-um-21.32.46.png)

在上面的截图中，你可以看到 Istio Bookinfo 演示的网格。 *reviews* 应用程序由三个版本化的工作负载组成，其中两个工作负载与 *ratings* 应用程序对话。

*productpage* 和 *reviews:v1* 之间的边缘被突出显示(通过点击它)，这在右侧显示了关于请求流量和响应时间的更详细的统计数据。

## 可视化问题

在下面的截图中，你可以看到网格中发生了一些不好的事情。图表的一部分显示为红色，在右边栏中，您可以看到大约三分之一的请求以错误告终。

[![detecting problems in the Istio service mesh](img/ff91dabd15e3362b31438bf1257f678f.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Bildschirmfoto-2018-09-18-um-21.48.31.png)

当你仔细观察*收视率:v1* 和*收视率*三角形时，你会看到一个 y 形图标。该图标表示*评级*服务应用了 Istio VirtualService。从*审查:v2，*请求达到*等级:v1* 工作负载，但是从*审查:v3，*请求失败。

点击*评级*服务的三角形图标，然后点击侧边栏中的链接，即可获得该服务的定义。在那里，您可以看到安装了一个虚拟服务，它将错误注入到数据流中。下面的 Kiali 屏幕截图显示了虚拟服务的详细信息:

[![Kiali showing details of a virtual service](img/e75c02c61160dc082b479aef343104ee.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Bildschirmfoto-2018-09-18-um-21.49.39.png)

Kiali 还可以验证 Istio 配置。例如，如果上述 HTTP 路由的权重之和不等于 100%，Kiali 会显示一个标志。

或者，当配置对象出现问题时，Kiali 可以进行标记。“详细信息”屏幕会显示该问题:

[![Kiali highlighting a configuration issue in a virtual service](img/96465cc852624e6b72791cae3c8c32d5.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Bildschirmfoto-2018-09-19-um-11.36.30.png)

## Kiali 入门

Kiali 文档中有一页是关于如何开始使用 OpenShift 或普通 Kubernetes 的。如果你安装了上游 Istio，你也可以选择用它来安装 Kiali。看看头盔的[配置选项或者用 Ansible](https://istio.io/latest/docs/reference/config/) 定制[。](https://istio.io/latest/search/?q=ANSIBLE)

使用 *admin/admin* 登录 Kiali(目前)是完成的。

## **为吉利做贡献**

Kiali 是开源的，在 Apache V2.0 许可下发布。[源代码位于 GitHub](https://github.com/kiali) 上，欢迎任何贡献。在你开始之前，请看看自述文件的[贡献部分。](https://github.com/kiali/kiali/blob/master/README.adoc#contributing)

## 更多信息

Kiali 网页和 GitHub 网页是一个良好的开端。Kiali 也有一个[博客](https://medium.com/kialiproject)和一个 [YouTube 频道](https://www.youtube.com/channel/UCcm2NzDN_UCZKk2yYmOpc5w)，专门记录冲刺结束时的演示。最后但同样重要的是，还有[推特](https://twitter.com/KialiProject)。:)

*Last updated: September 6, 2022*