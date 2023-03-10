# Istio 智能金丝雀发布:轻松投入生产

> 原文：<https://developers.redhat.com/blog/2018/04/24/istio-smart-canary-launch>

> 当气氛不够完美时，第一个倒下
> 
> 最轻微的缺陷都会动摇你的感情
> 
> 你像煤矿里的金丝雀一样生活...

Sting 和警察唱那些歌词的时候，我怀疑他们脑子里有 microservices，Istio，Kubernetes，OpenShift。然而，几年后，我们在这里使用金丝雀部署模式来简化代码的生产。

【这是我十周**[Istio 服务网系列](https://developers.redhat.com/topics/service-mesh/)** 介绍的第八部分。我之前的文章是[第七部分:Istio 黑暗发射:特勤局](https://developers.redhat.com/blog/2018/04/17/istio-dark-launch-secret-services/)。]

## 小心行事

如果您不熟悉 Canary 部署模式，这非常简单:您启动下一个版本的软件——在我们的例子中是微服务——然后向一小组用户授予有限的访问权限。如果这是成功的，你慢慢地增加用户群，直到软件失败——煤矿里的金丝雀死了——或者你成功地达到 100%的用户。通过有目的地、谨慎地将软件投入生产，并明智地决定哪些用户将请求新版本，您可以限制风险并最大化反馈。

当然，Istio 使这变得容易，同时为您提供了几个智能路由的好选项。而且——您可能以前听说过——您可以在不更改源代码的情况下完成所有这些工作。

## 搜索 Safari

一个简单的路由标准是只允许某些浏览者访问你的网站。例如，假设您想要限制 Safari 用户的访问权限，以便他们使用您的微服务版本 2。下面的 Istio 路由规则就可以做到这一点:

![](img/c70f523d7ca9deb438d4efac0a97d79d.png)

在应用这个路由规则之后，我们可以从命令行向微服务发起一个循环的`curl`请求，以模拟现实生活中的活动。结果是只有我们的微服务版本 1 响应请求:

![](img/cca28e4b09a5644ec123c42e195bb5ee.png)

2 版流量在哪里？在我的例子中，由于我从命令行运行`curl`，所有的流量都被路由到版本 1。请注意，在上面的屏幕截图结束时，我从浏览器(Safari)运行请求，结果如下:

![](img/aeace1645e9eb296376285281228b515.png)

## 无限的权力

您可能已经注意到，使用正则表达式来路由请求是非常强大的。考虑下面这个例子；我相信你很容易就能明白它的作用:

![](img/b4cd505f84e9005dbe7f37394ebe41f5.png)

给出这些例子，你可能已经在想象你能做什么了。

## 聪明地对待它

智能路由，尤其是对请求头使用正则表达式的能力，意味着在将新代码投入生产时，您可以按照自己的意愿引导流量。这很简单，不需要修改你的源代码，如果需要的话可以快速撤销。

## 我想要更多

想要更多吗？想在自己的 PC 上开始尝试 Istio、Kubernetes 和 OpenShift 吗？或许跟随一个教程？你很幸运:我们([红帽开发团队](https://developers.redhat.com/about/))已经为[准备了这个很棒的教程](https://github.com/redhat-developer-demos/istio-tutorial)。我们还提供了您需要的所有信息。简单地浏览一下教程，把自己弄晕。

我想你可以说它真的会让你的部署大放异彩。

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

*Last updated: September 3, 2019*