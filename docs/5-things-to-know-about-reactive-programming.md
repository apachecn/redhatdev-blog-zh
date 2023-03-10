# 关于反应式编程要知道的 5 件事

> 原文：<https://developers.redhat.com/blog/2017/06/30/5-things-to-know-about-reactive-programming>

**无功**，多么超载的一个词。如今，许多事情变得神奇地活跃起来。在本帖中，我们将讨论**反应式编程**，即围绕异步数据流构建的开发模型。

我知道你迫不及待地要写你的第一个反应式应用程序，但是在这之前，有几件事情需要知道。使用反应式编程改变了你设计和编写代码的方式。在跳上火车之前，最好知道你要去哪里。

在这篇文章中，我们将解释关于反应式编程的 5 件事，看看它为你带来了什么变化。

# 1.反应式编程是用异步数据流编程。

当使用反应式编程时，数据流将成为应用程序的主干。事件、消息、呼叫甚至故障都将通过数据流来传递。使用反应式编程，您可以观察这些流，并在发出值时做出反应。

因此，在您的代码中，您将创建任何内容和来自任何内容的数据流:点击事件、HTTP 请求、摄取的消息、可用性通知、变量的更改、缓存事件、来自传感器的测量，实际上是任何可能更改或发生的内容。这对您的应用程序有一个有趣的副作用:它变得天生异步。

![Reactive Programming is about asynchronous data streams](img/7b17438765d18e01733d33282b204915.png)

**Reactive eXtension**([http://react vex . io](http://reactivex.io)，a.ka. RX)是 Reactive 编程原则的一个实现，用来“*使用可观察序列*组成异步的和基于事件的程序。使用 RX，您的代码创建并订阅名为 *Observables* 的数据流。虽然反应式编程是关于概念的，但 RX 为您提供了一个惊人的工具箱。通过结合 o *bserver* 和*迭代器模式*和函数习惯用法，RX 给了你超能力。你有一个组合、合并、过滤、转换和创建数据流的功能库。下图说明了 RX 在 Java 中的用法(使用[https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava))。

![](img/668b0b5e8a465e428370cb3abccb7433.png)

虽然 RX 不是反应式编程原则的唯一实现(例如，我们可以引用 bacon js-[http://bacon js . github . io](http://baconjs.github.io))，但它是当今最常用的。在这篇文章的剩余部分，我们将使用 Rx Java。

# 2.可观察的事物可以是冷的或热的——这很重要。

在这一点上，你正试图看到你将在你的程序中处理的不同的流(或可观察的)。但是有两种流:热和冷。理解差异是成功使用反应式编程的关键。

冷观者懒。他们什么也不做，直到有人开始*观察*他们(*在 RX 中订阅*)。它们只有在被消耗的时候才开始运行。冷流用于表示异步操作，例如，只有当有人对结果感兴趣时才会执行。另一个例子是文件下载。如果没有人打算对数据做些什么，它就不会开始提取字节。冷流产生的数据不会在订阅者之间共享，当您订阅时，您会获得所有项目。

热点流在订阅之前是活跃的，如股票行情自动收录器，或者由传感器或用户发送的数据。数据独立于单个订户。当观察者订阅一个热可观察对象时，它将获得在它订阅的之后**发出的流中的所有值。这些值由所有订户共享。例如，即使没有人订阅温度计，它也会测量并发布当前温度。当订户注册到流时，它会自动接收下一个度量。**

为什么了解你的溪流是热的还是冷的如此重要？因为它改变了您的代码使用所传递的项目的方式。如果你没有订阅热点观察，你就不会收到数据，这些数据就丢失了。

# 3.误用异步咬

反应式编程*定义*中有一个很重要的词:异步。当数据在流中异步发出时，您会得到通知——这意味着独立于主程序流。通过围绕数据流构建程序，您可以编写异步代码:您可以编写当数据流发出新项时调用的代码。在这种情况下，线程、阻塞代码和副作用是非常重要的。先说副作用。

没有副作用的函数只通过它们的参数和返回值与程序的其余部分进行交互。副作用可能非常有用，在许多情况下是不可避免的。但是他们也有陷阱。当使用反应式编程时，你应该避免不必要的副作用，并且当他们使用时要有明确的意图。所以，拥抱不变性和无副作用的函数。虽然有些情况是合理的，但滥用副作用会导致雷暴:线程安全。

这是第二个要点:线程。观察流并在有趣的事情发生时得到通知是很好的，但是您必须永远不要忘记谁在调用您，或者更准确地说，您的函数是在哪个线程上执行的。强烈建议避免在程序中使用过多的线程。依赖于多线程的异步程序变成了一个棘手的同步难题，通常以寻找死锁而告终。

那就是第三点:千万不要屏蔽。因为你不拥有调用你的线程，所以你必须确保永远不要阻塞它。如果你这样做，你可以避免其他项目被发射，他们将被缓冲，直到…缓冲区满了(反压力可以在这种情况下踢，但这不是这篇文章的主题)。通过结合 RX 和异步 IO，您拥有了编写非阻塞代码所需的一切，如果您想要更多，可以看看 Eclipse Vert.x，这是一个测试反应性和异步性的反应性工具包。例如，下面的代码显示了 Vert.x Web 客户端及其 RX API 从服务器检索 JSON 文档并显示 *name* 条目:

```
client.get("/api/people/4")
.rxSend()
.map(HttpResponse::bodyAsJsonObject)
.map(json -> json.getString("name"))
.subscribe(System.*out*::println, Throwable::printStackTrace);
```

注意最后一个代码片段中的 *subscribe* 方法。当其中一个处理阶段抛出异常时，调用第二个方法。总是捕捉异常。如果你不这样做，你会花上几个小时去试图理解哪里出了问题。

# 4.保持事情简单

众所周知，权力越大，责任越大。“RX 提供了很多非常酷的功能，很容易偏向黑暗面。链接 *flapmap* 、*重试*、*去抖*和 *zip* 让你感觉自己像个忍者……**但是**，永远不要忘记好的代码需要别人也能读懂。

让我们来看一些代码...

```
manager.getCampaignById(id)
  .flatMap(campaign ->
    manager.getCartsForCampaign(campaign)
      .flatMap(list -> {
        Single<List<Product>> products = manager.getProducts(campaign);
        Single<List<UserCommand>> carts = manager.getCarts(campaign);
        return products.zipWith(carts, 
            (p, c) -> new CampaignModel(campaign, p, c));
      })
     .flatMap(model -> template
        .rxRender(rc, "templates/fruits/campaign.thl.html")
        .map(Buffer::toString))
    )
    .subscribe(
      content -> rc.response().end(content),
     err -> {
      log.error("Unable to render campaign view", err);
      getAllCampaigns(rc);
    }
);

```

举一个这样的例子可能很难理解，不是吗？它链接了几个异步操作(flatmap)，加入另一组操作( *zip* )。反应式编程代码首先需要思想转变。异步事件会通知您。然后，API 可能很难掌握(只要看看[的操作符列表](https://github.com/ReactiveX/RxJava/wiki/Alphabetical-List-of-Observable-Operators))。不要谩骂，不要写评论，不要解释，不要画图表(我确定你是一个 asciiart 艺术家)。RX 是强大的，滥用它或不解释它会让你的同事脾气暴躁。

# 5.反应式编程！反应系统

可能是最混乱的部分。使用**反应式编程不会构建反应式系统**。在[反应宣言](http://www.reactivemanifesto.org/)中定义的反应式系统，是*构建反应式分布式系统*的一种架构风格。反应式系统可以被视为分布式系统。反应系统有四个特征:

*   **响应式**:响应式系统需要在合理的时间内处理请求(我让你定义合理)。
*   弹性:反应式系统必须在面对故障(崩溃、超时、500 个错误……)时保持响应，因此它必须针对故障进行设计，并适当地处理它们。
*   **弹性**:反应式系统必须在各种负载下保持响应。因此，它必须可伸缩，并且能够以最少的资源处理负载。
*   **消息驱动**:来自反应式系统的组件使用异步消息传递进行交互。

尽管反应式系统的这些基本原理很简单，但构建其中之一却很棘手。通常，每个节点都需要采用异步非阻塞开发模型、基于任务的并发模型，并使用非阻塞 I/O。如果您不首先考虑这些要点，它很快就会变成意大利面条板。

反应式编程和反应式扩展提供了一种驯服异步野兽的开发模型。通过明智地使用它，你的代码将保持可读性和可理解性。然而，使用反应式编程并不会将您的系统转换成反应式系统。反应系统是下一个层次。

# 结论

我们终于到了这篇文章的结尾。如果你想更进一步，并且对 reactive 感兴趣，我建议你看看 Eclipse vert . x——一个构建反应式和分布式系统的工具包( [http://vertx.io)](http://vertx.io) ，以及从[https://developers . red hat . com/books/building-Reactive-micro services-Java](https://developers.redhat.com/books/building-reactive-microservices-java/old)获得的[Java 小册子](https://developers.redhat.com/books/building-reactive-microservices-java/old)中的反应式微服务。结合 Vert.x 和反应式伸展释放你的反应式超能力。你不仅可以使用反应式编程，还可以构建反应式系统，并进入一个激动人心的不断发展的生态系统。

编码快乐！

* * *

**下载**[**Eclipse vert . x**](https://developers.redhat.com/cheat-sheets/eclipse-vertx-cheat-sheet)**备忘单，该备忘单提供了一步一步的详细信息，让您以自己想要的方式创建应用。**

*Last updated: August 6, 2020*