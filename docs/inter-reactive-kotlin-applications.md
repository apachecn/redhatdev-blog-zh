# 交互式 Kotlin 应用

> 原文：<https://developers.redhat.com/blog/2017/12/07/inter-reactive-kotlin-applications>

[Kotlin 协同程序](http://vertx.io/docs/vertx-lang-kotlin-coroutines/kotlin/)是 Eclipse Vert.x 3.5 的主要特性之一。

我们大多数人都习惯于编写交互式代码，并且对每个人来说，采用反应式方法并不是一个微不足道的范式转变:使用异步 API 编程可能比使用直接同步风格更具挑战性，特别是如果您想要按顺序执行几个操作的话。此外，当使用异步 API 时，错误传播通常更复杂。

使用 Kotlin，您可以编写交互式 Vert.x 应用程序:以同步方式执行异步操作和接收事件。Inter-reactive 允许您通过使用您已经熟悉的直接同步风格来使用异步 API。

几周前，在 KotlinConf 2017 上展示 Vert.x 和 Kotlin 协程如何完美契合的绝佳机会:

[https://www.youtube.com/embed/__IUrGBWreM?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/__IUrGBWreM?autoplay=0&start=0&rel=0)

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**