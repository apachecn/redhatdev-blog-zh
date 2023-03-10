# Eclipse Vert.x 中的 JUnit 5 支持用于测试异步操作

> 原文：<https://developers.redhat.com/blog/2018/01/23/vertx-junit5-async-testing>

JUnit 5 是著名的 Java 测试框架的重写，带来了新的有趣特性，包括:

*   嵌套测试，
*   给出测试和测试用例的人类可读描述的能力，
*   比 JUnit 4 runner 机制更强大的模块化扩展机制( ***@RunWith*** 注释)，
*   条件测试执行，
*   参数化测试，包括来自诸如 CSV 数据的源，
*   在重新设计的内置断言 API 中支持 Java 8 lambda 表达式，
*   支持运行以前为 JUnit 4 编写的测试。

## 测试异步操作并不简单

Eclipse Vert.x 是一个越来越流行的在 JVM 上编写反应式应用程序的工具包。

用异步操作测试代码比乍看起来更具挑战性。的确，让我们考虑以下(不完整！)测试片段:

https://gist.github.com/jponge/aa990083c857160bb12ae9a1f627855b

该测试每 100ms 定义一个周期性任务，我们希望在周期性任务回调被执行 3 次时完成测试。因为***set period***定义了一个正在另一个线程上执行的异步操作，所以测试方法在调用***set period***之后立即返回，测试框架认为该方法已经成功。

解决方案是让测试框架运行者*等待*，直到所有的异步操作都成功完成。

Vert.x 已经提供了一个名为 [vertx-unit](https://github.com/vert-x3/vertx-unit/) 的模块来测试异步操作。它的优势在于它是多语言的，所以它适用于 Vert.x 支持的所有 JVM 语言，并且它提供了一个 JUnit 4 runner。

随着 JUnit 5 的出现，我们决定为 JUnit 5 开发一个[特定的集成(vertx-junit5)](https://github.com/vert-x3/vertx-junit5) ,它当然可以很好地用于 Java，也可以用于与 Java 无缝互操作的语言，以及使用 JUnit 很流行的语言:Kotlin 和 Groovy。值得注意的是，vertx-unit 并没有被放弃，因为它对于其他 JVM 语言和使用 JUnit 4 的项目仍然有用。

## 一个例子胜过千言万语

要用 JUnit 5 和 Vert.x 创建一个测试，我们只需定义一个类:

https://gist.github.com/jponge/1b289aeb6b4e82d26cfc10e836b38c19

使测试类受到包保护是 JUnit 5 的一个常见习语。

***@ExtendWith*** 注释允许使用 Vert.x 扩展(一分钟后会有更多介绍！).事实上，一个测试可以使用几个扩展，与 JUnit 4 中的 runners 相反。 ***@DisplayName*** 注释是可选的，但是它给出了一个人类可读的测试描述。最后，你可以使用表情符号！

回到我们计数滴答的例子，下面是我们如何在***SampleVerticleTest***中将它写成一个方法:

https://gist.github.com/jponge/079120cd9cc5a4b4ec07e771b06e8805

***VertTestContext*** 类提供了运行 Vert.x 异步操作的上下文，它用于定义测试何时完成或失败。请记住，操作是在 JUnit runner 之外的其他线程上执行的。对 ***completeNow()*** 的调用立即将测试标记为成功。

Vert.x 扩展做了两件重要的事情:

1.  当一个测试方法有这些类型的参数时，它(可选地)注入 ***Vertx*** 和 ***VertxTestContext*** 的实例，并且
2.  它确保 JUnit 测试运行程序等待异步操作完成，即使测试方法执行已经退出。

如果需要，可以手动创建 ***Vertx*** 和 ***VertxTestContext*** 实例，但是它们为大多数测试用例提供了合理的默认值。同样重要的是要注意到***VertxTestContext***总是等待测试超时完成，以免永远阻塞测试的执行。超时延迟可以通过注释定制。

## 检查点

并不是每个异步测试执行都有一个单点完成:在很多情况下，您需要检查几行特定的代码已经被执行。

我们提供了一个*检查点*抽象。当所有创建的检查点都被标记后，测试就成功了。

回到我们之前的示例，我们可以使用一个必须标记 3 次的检查点来更简单地重写它:

https://gist.github.com/jponge/1f19d5d4d810867a87faaad4a436b0ac

## 集成测试(+其他好东西)

事件处理代码的 Vert.x 功能单元被称为一个*顶点*。简而言之，verticle 处理异步事件，并由事件循环来管理，而事件循环本身永久地绑定到线程上。

让我们考虑下面的垂直。它在端口 11981 上启动一个 HTTP 服务器，并用***“Yo！”*** 正文:

https://gist.github.com/jponge/2c2303667da58239acfebbcbd49b5e74

现在让我们为这个垂直领域编写一个集成测试。更具体地说，我们需要确保:

1.  在 ***Vertx*** 上下文中成功部署该垂直，并且
2.  我们需要发出 HTTP 请求并检查响应。

我们将这样做，并发出 10 个 HTTP 客户端请求。

该测试符合以下方法:

https://gist.github.com/jponge/be4b3e65da36fbe4d111b21f49a947a5

Vert.x 中的很多异步操作都需要一个***async result<T>***回调，其中 ***AsyncResult*** 要么包含一个成功时的值，要么包含一个失败时的异常。为了使测试代码更容易，避免出现*“if/else blocks dance”*，***VertxTestContext***提供了 ***成功*** 和 ***失败*** 的助手方法。

这里我们在两个地方使用了*successing*因为我们期望成功，并且我们传递了一个回调来处理 ***AsyncResult*** 值。

另一个要点是，我们的 JUnit 5 支持与测试断言库无关。您可以使用内置的 JUnit 断言 API，或者像我们的例子一样使用 [AssertJ](http://joel-costigliola.github.io/assertj/) 。你所要做的就是调用 ***用一个可以放置断言的 lambda 来验证*** 。lambda 内部抛出的任何异常都会导致测试因该异常而失败。

## 一个完整的例子

![](img/f1ca205c7f22ba4110abb31a4571b17d.png)

下面是一个完整的测试示例，包括 JUnit 5 的生命周期回调等特性和一个嵌套测试，该测试展示了如何使用定制的 ***Vertx*** 和 ***VertxTestContext*** 对象:

https://gist.github.com/jponge/3c04d422f0969da04d0283859fefd5e7

## 更进一步

*   [Vert.x 网站](http://vertx.io/)提供了对 vert . x 生态系统的全面描述。
*   [vert . x JUnit 5 模块](https://github.com/vert-x3/vertx-junit5)提供了完整的文档。
*   [Vert.x 示例库](https://github.com/vert-x3/vertx-examples)包含了本文给出的示例的完整源代码，通常是探索*“所有 vert . x 的东西”*的绝佳资源！