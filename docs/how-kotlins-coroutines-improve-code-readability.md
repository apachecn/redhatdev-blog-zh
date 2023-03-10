# Kotlin 的协同程序如何提高代码可读性

> 原文：<https://developers.redhat.com/blog/2018/12/03/how-kotlins-coroutines-improve-code-readability>

> 程序必须写给人们阅读，并且只是附带地给机器执行。— [艾贝尔森和苏斯曼](https://en.wikiquote.org/wiki/Programming_languages)

Kotlin 是一种新的实用语言，旨在解决现实世界中的问题。它基于 JVM，但是在 Kotlin 和 Java 之间有很多 T2 差异。Kotlin 是一种[空安全](https://kotlinlang.org/docs/reference/null-safety.html)和[简洁](https://kotlinlang.org/docs/reference/faq.html#what-advantages-does-kotlin-give-me-over-the-java-programming-language)语言，支持[函数式](https://kotlinlang.org/docs/reference/lambdas.html)编程。你可以在这里尝试用 Kotlin [编程。](https://try.kotlinlang.org)

使用传统的编程风格，同时避免为每个任务分配一个线程，协同程序提供了一种简单的方式来编写高度可伸缩的代码。

在本文中，我将重点关注代码的可读性，以及在我看来，协同程序如何提供一种比反应式方法更简洁的方法来编写代码。我已经用 [Project Reactor](https://projectreactor.io) 展示了反应式代码；然而，该示例可以扩展到任何反应库，例如， [RxJava](https://github.com/ReactiveX/RxJava) 和 [CompleteableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) 。请注意，基于协程的代码与使用反应式方法编写的代码一样可伸缩。对我来说，协程对开发者来说是一个双赢的局面。

你可以在这里阅读更多关于 Kotlin 协同程序的信息。非常令人兴奋的[项目 Loom](https://openjdk.java.net/projects/loom/) 将把轻量级线程模型引入 Java。这是一个类似于[戈朗套路](https://tour.golang.org/concurrency/1)的概念。

## 方法

我使用 Reactor 和协程方法实现了以下工作流。主要功能是`processOrder`，它执行以下操作:

*   该过程从并行调用`getOrderInfo`和`getShipmentInfo`开始。
*   上述两种方法完成后，流程调用`sendEmail`方法。

让我们将`processOrder`函数称为**进程**函数，将个体`getOrderInfo` *、*、`sendEmail`函数称为**业务**函数。该代码可在本[回购](https://github.com/masoodfaisal/coroutines-reactive-code-clarity)中获得。它展示了与被动方法相比，Kotlin couroutines 如何在不损失可伸缩性优势的情况下编写可读性更高的代码。

## 简单就是美

在本节中，您可以找到使用 Kotlin 的协同程序和反应式方法实现`getOrderInfo`和`getShipmentInfo`业务功能以及`processOrder`流程功能的代码。

#### 业务职能

以下是使用 Kotlin 方法的业务功能示例。请注意，这些功能只是用少得多的[非功能开销](https://medium.com/@elye.project/understanding-suspend-function-of-coroutines-de26b070c5ed)来表示业务逻辑:

```
fun getOrderInfo(orderId: String): String {
     return "Order Info $orderId"
}
```

```
fun getShipmentInfo(orderId: String): String {
     return "Shipped for order $orderId"
}
```

请注意，用 Kotlin 编写相同的函数时，会将代码的非功能方面与业务逻辑混在一起。

```
Mono<String> getOrderInfo(String orderId) {
     return Mono.just("Order Info " + orderId);
}
```

```
Mono<String> getShipmentInfo(StringorderId) {
     return Mono.just("Shipped for order "+ orderId);
}
```

#### 过程函数

在本节中，您可以找到使用 Kotlin 的协同程序和反应式方法实现`processOrder`流程功能的代码。注意，通过使用 Kotlin 的方法，代码可读性很高。代码记录了业务功能试图做的事情。

```
fun processOrder() = runBlocking {
  val orderId = "SN19876"
  val orderInfo = async { getOrderInfo(orderId) }
  val shipmentInfo = async { getShipmentInfo(orderId) }
  sendEmail(shipmentInfo.await(), orderInfo.await())
}
```

下面的代码片段显示了使用反应式方法实现的相同业务流程。下面的代码可读性较差，因为代码的非功能方面与业务流程的功能方面一起出现。

```
void processOrder() {
  String orderIdNumber = "SN19876";

  Mono.zip(getOrderInfo(orderIdNumber), getShipmentInfo(orderIdNumber))
      .flatMap(data -> sendEmail(data.getT1(), data.getT2()))
      .doOnSuccess(o -> System.out.println("Email sent " + o))
      .subscribe();
}
```

## 结论

从上面的例子可以明显看出，协程为编写可读性更好的代码提供了更好的选择。这篇博客中提到的反应式代码可以优化以减少更多的噪音，但总会有代码的非功能方面与业务方面混合在一起，现在我可以使用 Kotlin 的协同程序来避免这种情况。

Kotlin 是一种令人兴奋的新编程语言，尤其是如果你来自一个有 Java 背景的 T2 人。你可以通过参加这个 [coursera 课程](https://www.coursera.org/learn/kotlin-for-java-developers)开始你的 Kotlin 之旅。

此外，您可能会对这篇文章感兴趣:[交互式 Kotlin 应用](https://developers.redhat.com/blog/2017/12/07/inter-reactive-kotlin-applications/)。

*Last updated: December 9, 2018*