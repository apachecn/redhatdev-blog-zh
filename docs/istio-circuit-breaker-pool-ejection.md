# Istio 断路器:如何处理(池)弹出

> 原文：<https://developers.redhat.com/blog/2018/03/20/istio-circuit-breaker-pool-ejection>

所有人都离开游泳池！

良好的...不是所有人。只有那些糟糕的演员。你知道，那些[微服务](https://developers.redhat.com/topics/microservices/)表现不好，没有做好自己的工作，太慢，等等。我们说的是 Istio，断路器和泳池弹射。

【这是我十周**[Istio 服务网系列](https://developers.redhat.com/topics/service-mesh/)** 介绍的第三部分。我之前的文章是[第 2 部分:Istio 路由规则:告诉服务请求去哪里](https://developers.redhat.com/blog/2018/03/13/istio-route-rules-service-requests/)。宁愿在视频里看到这个？点击查看视频版本[。]](https://developers.redhat.com/videos/youtube/OEo99GjUv6Q/)

## 事情应该是怎样的

当您使用 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 管理您的微服务时——例如 open shift——您的容量会根据需求自动扩大或缩小。因为微服务是在 pod 中运行的，所以在一个端点上可能会有几个微服务实例在容器中运行，由 Kubernetes 处理路由和负载平衡。这太好了；事情本该如此。一切都好。

正如我们所知，微服务是小而短暂的。短暂可能是一种轻描淡写；服务会像新小狗的吻一样出现又消失。pod 中微服务的特定实例的诞生和死亡是意料之中的，OpenShift 和 Kubernetes 处理得相当好。再说一遍，这就是它应该有的样子。一切都好。

## 事情的真相

但是，当一个特定的微服务实例——容器——由于崩溃(503 个错误)或更隐蔽的原因——响应时间过长而变坏时，会发生什么呢？也就是说，它并没有自动退出存在；它自己失败了或者变得很慢。你再试一次吗？改变路线？谁定义“耗时太长”，我们是否应该等待，稍后再试？多长时间后？

这个微小的微服务什么时候突然变得如此复杂了？

## Istio Pool 弹射:现实遇到它的比赛

再次，Istio 来拯救(不要表现得很惊讶，毕竟这些博客帖子是关于 Istio 的)。我们来看看 Istio 中带池弹射的断路器模式是如何工作的。

Istio 检测故障实例或异常值。在 Istio 词典中，这被称为*异常值检测*。策略是首先检测一个异常容器，然后让它在预先配置的时间内不可用，也就是所谓的*睡眠窗口*。当容器处于休眠窗口时，它被排除在任何路由或负载平衡之外。一个类比是万圣节晚上前廊的灯:如果灯是关着的，不管出于什么原因，房子没有参与进来。你可以跳过它，节省时间，只访问活跃的房子。如果房主 30 分钟后到家，打开门廊灯，去买些糖果。

为了了解这在 Kubernetes 和 OpenShift 中是如何进行的，这里有一个正常运行的微服务示例的屏幕截图，取自 [Red Hat Developer Demos repo](https://github.com/redhat-developer-demos/istio-tutorial) 。在本例中，有两个单元(v1 和 v2)，每个单元运行一个容器。在没有应用路由规则的情况下，Kubernetes 默认采用均衡的循环路由:

![](img/de6f4f968cb8b6f4b5b32cae5d86e599.png)

## 为混乱做准备

为了执行池弹出，你首先需要确保你有一个`routerule`在适当的位置。让我们使用 50/50 流量分配。此外，我们将使用一个命令将 v2 容器的数量增加到两个。以下是纵向扩展 v2 机架的命令:

`oc scale deployment recommendation-v2 --replicas=2 -n tutorial`

看一下路由规则的内容，我们可以看到流量在两个 pod 之间对半分割。

![](img/d0a7d1b14e1285e652f1f430cff5bd6e.png)
下面是该规则运行的屏幕截图:
![](img/6f35cb671413eda84cfbb5e155e4544b.png)

敏锐的观察者会注意到这不是一个均匀的 50/50 混合(它是 14:9)。然而，随着时间的推移，它会扯平。

## 让我们打破东西！

现在让我们介绍一个 v2 容器中的故障，剩下:一个健康的 v1 容器、一个健康的 v2 容器和一个故障的 v2 容器。结果如下:

![](img/9d09895d4bc5224dce90e8d1fe69c601.png)

## 最后，让我们修理东西

所以现在我们有一个容器正在失败，这就是 Istio pool 弹射的亮点。通过激活一个简单的配置，我们能够从任何路由中弹出故障容器。在本例中，我们将弹出它 15 秒钟，认为它会自我纠正(例如，通过重新启动或返回到更高的性能)。下面是配置文件和结果的屏幕截图:

![](img/dcdeb120fbf91c3903d8fa6cb5fdf652.png)

![](img/1e55913fe24f72d9ca0caff23bff4d68.png)

失败的 v2 容器没有被使用。15 秒后，容器会自动添加回池中。这是 Istio 池弹射。

## 开始构建一个架构

将 Istio pool ejection 与监控相结合，您可以开始构建一个框架，自动移除和替换有故障的容器，减少或消除停机时间和可怕的寻呼机呼叫。

下周的博文将介绍 Istio 提供的监控和跟踪功能。

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