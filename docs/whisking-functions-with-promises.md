# 使用 OpenWhisk 实现搅拌功能

> 原文：<https://developers.redhat.com/blog/2018/02/26/whisking-functions-with-promises>

在过去的几周里，我一直在学习和提高我的技能，围绕着新的热门词汇“无服务器”，并试图理解这个热门话题是怎么回事。作为一名热情的开源开发者，我一直在寻找一个可以开发和部署无服务器功能的平台，这时我偶然发现了 [Apache OpenWhisk](https://openwhisk.apache.org/) 。

在这篇博客中，我将演示如何构建一个简单的 nodejs 函数，它可以使用 [Google Maps API](https://developers.google.com/maps/documentation/javascript/examples/) 进行反向地理编码，以及如何将这些函数部署到 Apache OpenWhisk 上。

上下文显示构建一个涉及回调的[Apache open whish](https://openwhisk.apache.org/)JavaScript[动作](https://github.com/apache/incubator-openwhisk/blob/master/docs/actions.md)。我们大多数人都熟悉[谷歌地图 API (](https://developers.google.com/maps/documentation/javascript/examples/) 有很多回调函数)，它为这个博客提供了一个很好的例子。

这个博客的源代码可以在我的 [github](https://github.com/kameshsampath/location-finder.git) 仓库中找到。

由于我是 nodejs 开发的新手，我确实在配置、功能、定义和调用函数方面犯了一些错误。这个博客将解释我做错了什么，以及我做了什么来使功能按预期工作。

有了上下文设置，让我们开始编写函数(首先是错误的方式；))使用 [Google Maps API](https://developers.google.com/maps/documentation/javascript/examples/) 为我们进行反向地理编码:

https://gist.github.com/kameshsampath/efc3c3fe396b34af56ff93d44796675c

为了简洁和坚持这篇博客的上下文，我跳过了[源回购](https://github.com/kameshsampath/location-finder)和相关 npm 脚本的细节。对于本博客的其余部分，我们只需要知道:

*   **建造**是`npm run build`
*   行动**部署**是`npm run deploy`
*   动作**调用**是`npm run dev`

在我们构建`npm run build`，部署`npm run deploy `函数之后，我们通过`npm run dev`调用动作，总是返回如下结果:

> {状态 : 状态，地点::'未知 ' }

我不知道为什么这不起作用。:(
然而，通过一点研究和查阅 open whish[actions](https://github.com/apache/incubator-openwhisk/blob/master/docs/actions.md#creating-asynchronous-actions)文档，我发现我没有正确处理谷歌地图客户端“reverseGeocode”方法的回调函数。然后我决定将回调封装在[承诺](https://developers.google.com/web/fundamentals/primers/promises)中，并返回一个[承诺](https://developers.google.com/web/fundamentals/primers/promises)作为 OpenWhisk nodejs 动作的响应。

按照 open whish[actions](https://github.com/apache/incubator-openwhisk/blob/master/docs/actions.md#creating-asynchronous-actions)文档，我尝试将代码更新为:

https://gist.github.com/kameshsampath/efcf46fdf190befe83e6f3ffc9266a40

调用 action post doing `npm run build`让我每况愈下，动作挂起，没有响应。:(

通过`wsk activation poll`轮询 OpenWhisk 日志显示了下面几行:

> 激活:' location-finder '(750 f 66 BD 750d 426 D8 f 66 BD 750d 026 d2a)[
> " 2018-02-23t 05:27:06.453 z stderr:收集您的日志时出现问题。数据可能会丢失。
> ]

随着进一步的分析和调试，我发现我需要让谷歌地图客户端 promise aware。

我对该函数做了进一步的最终修改，如下所示:

https://gist.github.com/kameshsampath/bc47f23e885b4e282557b6a42b9936f1

两个重要的变化:

*   **第 11 行-** 我在那里创建了谷歌地图客户端 promise aware。
*   我修改了节点函数，通过 **asPromise()** 方法从位置函数返回承诺。

执行重建、部署和运行操作返回了预期的响应:

> {
> "位置":"新排，伦敦 WC2N 4LH，英国"，
> "状态":" OK"
> }

在这个例子中，我们看到了如何在谷歌地图客户端上配置[承诺](https://developers.google.com/web/fundamentals/primers/promises)。如果您正在使用来自其他 API 的类似函数，您需要检查如何挂钩到 API 调用，该调用可以为您提供一个[承诺](https://developers.google.com/web/fundamentals/primers/promises)的句柄。

总之，这里的关键知识是关于如何从一个[Apache open whish](https://openwhisk.apache.org/)JavaScript 动作中正确地返回一个[承诺](https://developers.google.com/web/fundamentals/primers/promises)。当调用 OpenWhisk 动作时，它应该返回将来的响应( [Promise](https://developers.google.com/web/fundamentals/primers/promises) )，而不应该在主函数结束后立即退出函数。

*Last updated: May 17, 2018*