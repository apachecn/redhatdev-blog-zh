# Jug 夏令营 2017，Vert.x 和协作 DJ 混音

> 原文：<https://developers.redhat.com/blog/2017/10/19/jug-summer-camp-2017-vert-x-collaborative-dj-mix>

有幸呈现***“Eclipse vert . x 为了 Dj 乐趣，为了利益！”*** 在法国拉罗谢尔举行的最新一届 [Jug 夏令营](http://www.jugsummercamp.org/edition/8)。

Jug 夏令营是一个受欢迎的开发者大会，由 Serli 在法国西部组织，聚集了地区参与者以及来自其他法国 Java 用户组的演讲者和参与者。

我的演讲是对使用 Eclipse Vert.x 的[反应式编程的介绍，包括基于 RxJava 的 edge 服务的演示以及一个协作 DJ mix 会话。Vert.x 的伟大之处在于它可以很好地适应各种分布式应用。](http://vertx.io/)

https://twitter.com/k33g_org/status/908710313360576513

[DJ 混音演示(名为锅炉轰鸣)](https://github.com/jponge/boiler-vroom)允许与会者连接到一个“实时”网络应用程序，他们可以看到现场的 DJ 动作，控制一些元素(过滤器、音序模式...)且听溪水。还有一个 WiFi 连接的 RaspberryPi 来提供音量表:

https://twitter.com/jponge/status/874279518579642368

Vert.x 在几个方面大放异彩:

1.  构建可伸缩的 HTTP 后端。
2.  将大量数据传输到几个客户端，并应对反压力。
3.  由于用于 JavaScript 应用程序的 SockJS 事件总线桥，为后端和前端提供了一致的基于事件的编程模型。
4.  在 web 客户端和 MIDI 驱动的音乐软件之间执行协议适配。

这些是幻灯片:

https://speakerdeck.com/jponge/eclipse-vert-dot-x-for-dj-fun-and-for-profit

如果你懂法语，这是演讲的视频:

https://www.youtube.com/watch?v=9GJwxudfnoI

非常感谢这次盛会的组织者和参与者。

* * *

**下载**[**Eclipse vert . x**](https://developers.redhat.com/promotions/vertx-cheatsheet/)**备忘单，该备忘单提供了一步一步的详细信息，让您以自己想要的方式创建应用。**

*Last updated: October 16, 2017*