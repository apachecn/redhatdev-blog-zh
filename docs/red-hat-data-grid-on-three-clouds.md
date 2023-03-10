# 三种云上的 Red Hat 数据网格(演示背后的细节)

> 原文：<https://developers.redhat.com/blog/2018/06/19/red-hat-data-grid-on-three-clouds>

如果你在 2018 年红帽峰会上看到或听说过[的多云演示，这篇文章详细介绍了我们如何在三家云提供商之间以主动-主动-主动模式运行](https://developers.redhat.com/blog/2018/05/10/red-hat-summit-2018-burr-sutter-demo/)[红帽数据网格](https://developers.redhat.com/products/datagrid/overview/)。这种设置使我们能够实时显示云提供商之间的故障转移，而不会丢失数据。除了红帽数据网格，我们还使用了 [Vert.x](https://developers.redhat.com/blog/2018/03/13/eclipse-vertx-first-application/) (反应式编程)[open whish](https://developers.redhat.com/blog/category/topics/serverless/)(无服务器)，以及[红帽 Gluster 存储](https://www.redhat.com/en/technologies/storage/gluster)(软件定义存储。)

今年的[红帽峰会](https://developers.redhat.com/blog/tag/red-hat-summit-2018/)对我们所有人来说都是一次冒险。旧金山之旅可能是全世界 IT 极客的遗愿清单之一。此外，我们能够满足许多其他红帽，谁为红帽远程工作，因为我们这样做。然而，最好的部分是我们有一些重要的事情要说:“我们相信混合/多云”，我们要在舞台上证明这一点。

![](img/0992086cdf5692765b86ad5d7eae2554.png)

*Photo credit: Bolesław Dawidowicz*

### 我们的主题演示

在早期的一次主题演讲演示中，我们的混合云通过 Red Hat OpenShift 在舞台上的机架中进行实时调配。看到所有工具协同工作使演示获得成功，令人印象深刻。我们与一群队友一起负责创建第四个 keynote 演示，其中涉及到一些有趣的中间件技术，如 Red Hat Data Grid、Vert.x、OpenWhisk 和 Red Hat Gluster。如果您还没有看过演示，请在线观看:

[https://www.youtube.com/embed/hu2BmE1Wk_Q?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/hu2BmE1Wk_Q?autoplay=0&start=0&rel=0)

你看不到的是，我们有一个维护团队坐在幕后，观察一切是否正常(下图，来自 Red Hat SSO 团队的 Marek Posolda 和来自 Red Hat Data Grid 团队的 Galder Zamarreñ):

![](img/fad6a03ae375f4cd451bb61db91cb902.png)

演示进行得很顺利，所以是时候解释一下我们是如何做到的了。

### 主题演讲演示#4 架构

我们使用了三个数据中心(Amazon、Azure 和 on-stage rack ),它们以主动-主动-主动复制配置工作。

![](img/9b08763e69a8b7d4554fa35a1cadba94.png)

每个站点都使用微服务和无服务器架构。我们还使用了两个存储层:Gluster 用于存储图像，Red Hat Data Grid 用于存储元数据(JSON 格式):

![](img/b141cf7041e19a8057206ed315d62373.png)

### 跨站点复制的数据网格设置

Red Hat 数据网格的主动-主动设置是使用[跨站点复制](https://access.redhat.com/documentation/en-us/red_hat_jboss_data_grid/7.2/html/administration_and_configuration_guide/set_up_cross_datacenter_replication)(也称为 x 站点复制)功能实现的。然而，我们不得不稍微调整配置:

![](img/e6203bd4d0993701e13417f20b5c99dc.png)

这些站点使用 Red Hat OpenShift 的负载平衡器服务相互通信。负载平衡器是预先分配的，它们的坐标(所有三个云)被注入到一个秘密中，创建了一个所谓的全局集群发现字符串。配置 XML 文件被放到一个`ConfigMap`中，允许我们轻松地从 UI 手动更新配置(如果需要的话)。

因为我们坚信自动化，所以我们创建了自动化设置脚本，用于将 Red Hat 数据网格提供给所有三种云。这些脚本可以在我们的 [GitHub 库](https://github.com/rhdemo/jdg-as-a-service)中找到。我们还提供了脚本的定制版本，可以使用三个不同的项目模拟 x 站点复制。您可以使用`oc cluster up`命令启动 Red Hat OpenShift 容器平台本地实例，然后运行[这个供应脚本](https://github.com/rhdemo/jdg-as-a-service/blob/master/local-full-deployment.sh)。

现在，让我们看看所有的配置位，并探索它们是如何协同工作的(您可以使用本地配置脚本来检查它们)。让我们从全局集群发现秘密开始:

https://gist.github.com/slaskawi/91ff7aff0584f5ebb52b3a2314db46ef

`DISCOVERY`字符串是专用于 [TCPPING](http://www.jgroups.org/manual4/index.html#TCPPING_Prot) 的，用于全局集群发现。下一个参数是`EXT_ADDR`，代表负载平衡器的公共 IP 地址。JGroups 通信工具包需要将全局集群传输绑定到那个特定的地址。`SITE`描述本地站点:是亚马逊还是 Azure 还是私有的(舞台架上的)站点？

下一个难题是配置。完整的 XML 文件可以在这里找到。在这篇博文中，我们将只关注关键的部分:

https://gist.github.com/slaskawi/54a7b231d69cf0f953e38baa14f3f368

本地集群(在单个数据中心内)使用 [KUBE_PING](https://github.com/jgroups-extras/jgroups-kubernetes) 协议发现所有其他成员。使用 [RELAY2](http://www.jgroups.org/manual4/index.html#RELAY2) 协议将所有消息转发到其他站点。使用来自`jdg-app`秘密的[向下 API](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/) 将`jboss.relay.site`的值注入环境变量。最后有趣的一点是`max_site_masters`被设置为`1000`。因为我们不知道负载平衡器后面有多少实例，所以我们需要确保它们中的每一个都可以充当站点主机，并将所有流量转发到另一个站点。

为`RELAY`协议定义了下一个堆栈:

https://gist.github.com/slaskawi/7483ef2915bf53e2cf2abad080c5f5ab

从 TCP 协议开始，我们需要确保我们绑定到负载平衡器公共地址。这就是为什么所有的负载平衡器都必须预先配置。同样，我们从`jdg-app`秘密中注入这个变量。另一个有趣的片段是`TCPPING`，这里我们使用了全局集群发现字符串(来自`jdg-app`秘密)。`FD_ALL`超时被设置为一个相当高的数字，因为我们希望容忍短时间的站点停机。

https://gist.github.com/slaskawi/35c57399c89168e60dbcd2a96bc8baf9

每个站点都将所有三个站点配置为异步备份。这允许实现主动-主动-主动类型的设置。

### 最后的想法

主动-主动-主动设置是处理大量全局路由流量的一种非常有趣的方法。但是，在实施之前需要考虑几件事情:

*   不要从不同的数据中心写入相同的密钥。如果你这样做，你可能会因为网络延迟而很快陷入困境。
*   当站点关闭并重新启动时，您必须触发状态转换来同步它。这是一项手动任务，但我们正在努力使其完全透明。
*   确保您的应用程序能够容忍异步复制。使用同步缓存的跨站点复制可能会有问题。

在演示工作中，我们学到了很多关于全局负载平衡设置的知识。发现新的用例是非常有趣的，我们相信我们知道如何让 Red Hat Data Grid 更好地适应这些场景。

祝跨站点复制愉快，下次峰会再见！
红帽数据网格团队

*Last updated: June 18, 2018*