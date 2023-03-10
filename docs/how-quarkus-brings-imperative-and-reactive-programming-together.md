# Quarkus 如何将命令式和反应式编程结合在一起

> 原文：<https://developers.redhat.com/blog/2019/11/18/how-quarkus-brings-imperative-and-reactive-programming-together>

超音速亚原子 Java 奇点扩大了！

42 个版本、8 个月的社区参与和 177 名出色的贡献者促成了 [Quarkus 1.0](https://developers.redhat.com/topics/quarkus/) 的发布。这个版本是一个重要的里程碑，背后有很多很酷的特性。你可以在[发布公告](https://quarkus.io/blog/announcing-quarkus-1-0/)中了解更多。

基于这个惊人的消息，我们想深入研究 Quarkus 如何统一命令式和反应式编程模型及其反应式核心。我们将从简短的历史开始，然后深入探究这个双面反应式内核的组成，以及 Java 开发人员如何利用它。

[微服务](https://developers.redhat.com/topics/microservices/)、[事件驱动架构](https://developers.redhat.com/topics/event-driven/)、[无服务器](https://developers.redhat.com/topics/serverless-architecture/)功能正在兴起。最近，创建云原生架构变得更加容易；然而，挑战依然存在，尤其是对于 Java 开发人员来说。无服务器功能和微服务需要更快的启动时间，消耗更少的内存，最重要的是给开发者带来快乐。在这方面，Java 在最近几年做了一些改进(例如，容器的人体工程学增强，等等。).然而，要拥有一个可执行的容器原生 Java，并不容易。让我们首先来看看开发容器本地 Java 应用程序的一些固有问题。

让我们从一点历史开始。

![Threads CPUs Java and Containers](img/614f2c36939a4e2ed2cb16f95fad01ac.png)

## 线程和容器

从版本 8u131 开始，由于人体工程学的增强，Java 更加具有容器意识。所以现在，JVM 知道它所运行的内核数量，并可以相应地定制线程池——通常是 fork/join 池。这很好，但是假设我们有一个传统的 web 应用程序，它在 Tomcat、Jetty 等上使用 HTTP servlets 或类似的东西。实际上，这个应用程序为每个请求提供了一个线程，允许它在等待 IO 发生时阻塞这个线程，比如访问数据库、文件或其他服务。这种应用的规模取决于并发请求的数量，而不是可用内核的数量；这也意味着 Kubernetes 中对内核数量的配额或限制不会有很大帮助，最终会导致节流。

## 记忆衰竭

线程也会消耗内存。容器内的内存约束不一定有帮助。将这种情况分散到多个应用程序和线程中会导致更多的切换，在某些情况下还会导致性能下降。此外，如果一个应用程序使用传统的微服务框架，创建数据库连接，使用缓存，并且可能需要更多的内存，那么马上就需要查看 JVM 内存管理，以便它不会被杀死(例如，XX:+UseCGroupMemoryLimitForHeap)。尽管 JVM 可以理解 Java 9 中的 cgroups 并相应地调整内存，但是管理和调整内存仍然会变得非常复杂。

## 配额和限额

在 Java 11 中，我们现在支持 CPU 配额(例如，PreferContainerQuotaForCPUCount)。Kubernetes 还支持限额和配额。这可能是有意义的；但是，如果应用程序再次使用超过配额的线程，我们最终会根据内核进行调整，这对于传统的 Java 应用程序来说，每个请求使用一个线程是没有任何帮助的。

此外，如果我们使用配额和限制或者底层 Kubernetes 平台的横向扩展特性，问题不会自己解决；我们将在潜在问题上投入更多的能力，或者最终会过度投入资源。如果我们在公共云中的高负载上运行，我们最终肯定会使用不必要的资源。

## 什么能解决这个？

解决这些问题的直接方法是使用异步和非阻塞 IO 库和框架，如 Netty、 [Vert.x、](https://developers.redhat.com/blog/2019/10/21/eclipse-vert-x-3-8-1-update-for-red-hat-runtimes/)或 Akka。由于它们的反应性质，它们在容器中更有用。通过采用非阻塞 IO，同一个线程可以处理多个并发请求。当一个请求处理等待一些 IO 时，线程被释放，因此可以用来处理另一个请求。当最终接收到第一请求所需的 IO 响应时，第一请求的处理可以继续。使用同一个线程交叉请求处理大大减少了线程的数量以及处理负载的资源。

对于非阻塞 IO，内核数量成为基本设置，因为它定义了您可以并行运行的 IO 线程数量。如果使用得当，它可以有效地将负载分配到不同的内核上，用更少的资源处理更多的任务。

## 就这些吗？

而且，还有更多。反应式编程提高了资源利用率，但不是免费的。它要求应用程序代码包含非阻塞，并避免阻塞 IO 线程。这是一种不同的开发和执行模式。尽管有许多库可以帮助你做到这一点，但这仍然是一种思维方式的转变。

首先，您需要学习如何编写异步执行的代码，因为一旦您开始使用非阻塞 IOs，您就需要表达一旦收到响应将会发生什么。你不能再等待和阻挠了。为此，您可以传递回调、使用反应式编程或 continuation。但是，这还不是全部，你需要使用非阻塞的 IOs，因此可以访问非阻塞的服务器和客户端来获得你需要的一切。HTTP 是一个简单的例子，但是想想数据库访问、文件系统等等。

尽管端到端反应式提供了最佳效率，但这种转变可能难以理解。混合反应式代码和命令式代码的能力对于以下方面变得至关重要:

1.  有效使用热路径上的资源，以及
2.  为应用程序的其余部分提供更简单的代码风格。

## 进入夸库斯

这就是 Quarkus 的全部内容:在单一运行时中统一反应性和命令性。

Quarkus 以 Vert.x 和 Netty 为核心。此外，它还在顶层使用了一系列反应式框架和扩展来帮助开发人员。Quarkus 不仅仅是针对 HTTP 微服务，也是针对事件驱动架构。它的反应性质使它在处理信息时非常有效(例如阿帕奇·卡夫卡或 AMQP)。

这背后的秘密是对命令式代码和反应式代码使用单一的反应式引擎。

![](img/db6e0312e7ff55661184f3dda9fb52f7.png)

夸库斯做得非常出色。在命令式和反应式之间，显而易见的选择是拥有一个反应式核心。这有助于快速非阻塞代码处理几乎所有通过事件循环线程(IO 线程)进行的事情。但是，如果您正在创建一个典型的 REST 应用程序或客户端应用程序，Quarkus 也为您提供了命令式编程模型。例如，Quarkus HTTP 支持基于非阻塞和反应式引擎(Eclipse Vert.x 和 Netty)。应用程序接收的所有 HTTP 请求都由*事件循环* (IO 线程)处理，然后被路由到管理请求的代码。根据目的地，它可以调用工作线程(servlet、Jax-RS)上管理请求的代码，或者使用 IO was 线程(反应式路由)。

[![](img/ee67c0178ad2dc539a62987deec23a34.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/11/img_5dcca0ed0a68d.png)

对于消息传递连接器，使用非阻塞客户端并在 Vert.x 引擎上运行。因此，您可以有效地发送、接收和处理来自各种消息中间件的消息。

为了帮助你在 Quarkus 上开始使用 reactive，在 [Quarkus.io:](http://www.quarkus.io) 上有一些清晰的指南

*   [使用反应路线](https://quarkus.io/guides/reactive-routes-guide)
*   [反应式 SQL 客户端](https://quarkus.io/guides/reactive-sql-clients)
*   [使用 Apache Kafka 和反应式消息](https://quarkus.io/guides/kafka-guide)
*   [将 AMQP 用于反应式消息传递](https://quarkus.io/guides/amqp-guide)
*   [使用 Vertx API](https://quarkus.io/guides/using-vertx)

还有反应式的演示场景，可以在线尝试；你不需要电脑或 IDE，只需在你的浏览器中尝试一下。你可以在这里试用它们。

### 额外资源

*   [尝试夸夸其谈的四个理由](https://www.redhat.com/cms/managed-files/cl-4-reasons-try-quarkus-checklist-f19180cs-201909-en.pdf)
*   Quarkus 网址: [http://quarkus.io](http://quarkus.io)
*   Quarkus GitHub 项目:[https://github.com/quarkusio/quarkus](https://github.com/quarkusio/quarkus)
*   夸尔库斯推特:[https://twitter.com/QuarkusIO](https://twitter.com/QuarkusIO)
*   夸库聊天:[https://quarkusio.zulipchat.com/](https://quarkusio.zulipchat.com/)
*   夸库斯邮件列表:[https://groups.google.com/forum/#!forum/quarkus-dev](https://groups.google.com/forum/#!forum/quarkus-dev)

*Last updated: April 22, 2022*