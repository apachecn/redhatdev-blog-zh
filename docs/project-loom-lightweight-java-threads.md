# Project Loom:轻量级 Java 线程

> 原文：<https://developers.redhat.com/blog/2019/06/19/project-loom-lightweight-java-threads>

构建响应性应用程序是一项永无止境的任务。随着功能强大的[多核](https://en.wikipedia.org/wiki/Multi-core_processor)CPU 的出现，更多的原始功率可供应用程序消耗。在 [Java](https://developers.redhat.com/topics/enterprise-java/) 、[线程](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html)用于让应用程序同时处理多个任务。开发人员在程序中启动一个 Java 线程，任务被分配给这个线程进行处理。线程可以完成各种任务，比如从文件中读取、向数据库中写入、接收用户输入等等。

在本文中，我们将解释更多关于线程的内容，并介绍 [Project Loom](https://wiki.openjdk.java.net/display/loom/Main) ，它在 Java 中支持高吞吐量和轻量级并发，有助于简化可伸缩软件的编写。

## 使用线程获得更好的可伸缩性

Java 让创建新线程变得如此容易，几乎所有时候程序最终都会创建比 CPU 能够并行调度的线程更多的线程。假设我们有一条两车道的道路(一个 CPU 的两个核心)，10 辆车想同时使用这条道路。自然，这是不可能的，但是想想这种情况目前是如何处理的。红绿灯是一种方式。交通灯允许一定数量的汽车驶入道路，并使交通有序地使用道路。

在计算机中，这是一个调度程序。调度程序将线程分配给 CPU 内核来执行它。在现代软件世界中，操作系统扮演着为 CPU 调度任务(或线程)的角色。

在 Java 中，每个线程都被 JVM 映射到一个操作系统线程(几乎所有的 JVM 都这样做)。由于线程数量超过了 CPU 内核，因此会分配大量 CPU 时间来调度内核上的线程。如果线程进入等待状态(例如，等待数据库调用响应)，该线程将被标记为*暂停*，并且一个单独的线程被分配给 CPU 资源。这被称为[上下文切换](https://en.wikipedia.org/wiki/Context_switch)(尽管这样做涉及更多)。此外，每个线程都分配有一些内存，操作系统只能处理有限数量的线程。

考虑一个应用程序，其中所有线程都在等待数据库响应。虽然应用程序计算机正在等待数据库，但是应用程序计算机上正在使用许多资源。随着 web 级应用程序的增加，这种线程模型可能会成为应用程序的主要瓶颈。

## 反应式编程

一种解决方案是利用[反应式编程](https://en.wikipedia.org/wiki/Reactive_programming)。简而言之，不是为每个并发任务(和阻塞任务)创建线程，而是一个专用线程(称为事件循环)在一个非反应式模型中查看分配给线程的所有任务，并在*相同的* CPU 核心上处理它们中的每一个。因此，如果一个 CPU 有四个核心，可能会有多个事件循环，但不会超过 CPU 核心的数量。这种方法解决了上下文切换的问题，但是在程序本身中引入了很多复杂性。这种类型的程序伸缩性也更好，这也是反应式编程最近变得非常流行的一个原因。 [Vert.x](http://vertx.io/docs/guide-for-java-devs/) 就是这样一个库，它帮助 Java 开发人员以一种反应式的方式编写代码。

你可以在这里和克莱门特·埃斯科菲耶的这本免费的[电子书中了解更多关于反应式编程的知识。](https://developers.redhat.com/promotions/building-reactive-microservices-in-java/)

## 复杂性最低的可扩展性

因此，每任务线程模型很容易实现，但不可伸缩。反应式编程更具可伸缩性，但实现有点复杂。表示程序复杂性与程序可伸缩性的简单图表如下所示:

[![](img/f3378c7e813328d71aba479737099860.png "Loom")](/sites/default/files/blog/2019/05/Loom.png)Program Complexity vs ScalabilityProgram complexity vs. scalability.">

我们需要的是上图中提到的最佳点(绿点)，在这里我们以最小的应用程序复杂性获得 web 规模。进入[项目织机](https://wiki.openjdk.java.net/display/loom/Main)。但是首先，让我们看看当前的每个线程一个任务模型是如何工作的。

## 当前每任务线程模型的工作原理

让我们看看它的实际效果。首先让我们编写一个简单的程序，一个 echo 服务器，它接受一个连接并为每个新连接分配一个新线程。让我们假设这个线程正在调用一个外部服务，它在几秒钟后发送响应。这模拟了线程的等待状态。因此，一个简单的 Echo 服务器将如下例所示。完整的源代码可以在[这里](https://github.com/masoodfaisal/loomexample)找到。

```
//start listening on a socket
ServerSocket server = new ServerSocket(5566);

while (true) {
    Socket client = server.accept();
    EchoHandler handler = new EchoHandler(client);
    //create a new thread for each new connection
    handler.start();
}
.
.
.
//extends Thread
class EchoHandler extends Thread {
public void run () {
   try {
     //make a call to the dummy downstream system. This calls will return after a couple of seconds
     //simulating a wait/block call.
     byte[] output = new java.net.URL("http://localhost:9090").openStream().readNBytes(5);
     //write something to the connection output stream
     writer.println("[echo] " + output);

```

当我运行这个程序并调用 100 次时，JVM 线程图显示一个峰值，如下所示(来自 jconsole 的输出)。我执行的生成调用的命令非常简单，它添加了 100 个 JVM 线程。

```
for i in {1..100}; do curl localhost:5566 & done
```

![](img/2177262ee69c656a790970e872a7cb0e.png)

## 潜龙计划

Project Loom 没有为每个 Java 线程分配一个操作系统线程(当前的 JVM 模型)，而是提供了额外的调度器，这些调度器在同一个操作系统线程上调度多个轻量级线程。这种方法提供了更好的使用(操作系统线程总是在工作而不是等待)和更少的上下文切换。

维基表示，Project Loom 支持“易于使用、高吞吐量的轻量级并发和 Java 平台上的新编程模型。”

项目织机的核心涉及延续和纤维。以下定义来自艾伦·贝特曼的精彩演讲，可在此[获取](http://cr.openjdk.java.net/~alanb/loom/Devoxx2018.pdf)。

### 纤维

> 纤程是轻量级的用户模式线程，由 Java 虚拟机调度，而不是由操作系统调度。光纤占用空间小，任务切换开销可以忽略不计。你可以拥有数百万个！

### 继续

> Continuation(确切地说:delimited continuation)是一个程序对象，表示可以暂停和恢复(也可能是克隆甚至序列化)的计算。

本质上，我们大多数人永远不会在应用程序代码中使用延续。我们大多数人会使用纤维来增强我们的代码。一个简单的定义是:

> 纤程=延续+调度程序

好吧，这似乎很有趣。任何新方法的挑战之一是它与现有代码的兼容性。Project Loom 团队在这方面做得非常好，Fiber 可以带 *Runnable* 接口。为了完整起见，请注意*延续*也实现了*可运行*。

因此，我们的 echo 服务器将发生如下变化。注意，改变的部分只是线程调度部分；线程内部的逻辑保持不变。完整的源代码可以在这里找到。

```
ServerSocket server = new ServerSocket(5566);
while (true) {

Socketclient=server.accept();

EchoHandlerLoomhandler=newEchoHandlerLoom(client);

 //Instead of running Thread.start() or similar Runnable logic, we can just pass the Runnable to the Fiber //scheduler and that's it

Fiber.schedule(handler);

}
```

**注:**这里[有项目织机的 JVM](https://wiki.openjdk.java.net/display/loom/Main#Main-DownloadandBuildfromSource)。我们需要从 Project Loom 分支构建 JVM，并开始将其用于 Java/C 程序。下面显示了一个示例:

```
java -version
openjdk version "13-internal" 2019-09-17
OpenJDK Runtime Environment (build 13-internal+0-adhoc.faisalmasood.loom)
OpenJDK 64-Bit Server VM (build 13-internal+0-adhoc.faisalmasood.loom, mixed mode)

```

有了这个新版本，线程看起来好多了(见下文)。默认情况下，纤程使用 [ForkJoinPool](https://docs.oracle.com/javase/10/docs/api/java/util/concurrent/ForkJoinPool.html) 调度器，尽管图表以不同的比例显示，但是您可以看到，与每个任务一个线程的模型相比，这里的 JVM 线程数量要少得多。这导致了我们在前面显示的图表中瞄准的绿色点。

![](img/e5476abdba48d0bd63bf6ead1dbc700c.png)

## 结论

Project Loom 带来的改进令人振奋。我们将线程数量减少了五分之一。Project Loom 允许我们用每个任务一个轻量级线程来编写高度可伸缩的代码。这简化了开发，因为您不需要使用反应式编程来编写可伸缩的代码。另一个好处是，许多遗留代码可以使用这种优化，而无需对代码库进行太多更改。我想说 Project Loom 带来了与 goroutines 类似的能力，并允许 Java 程序员编写互联网规模的应用程序，而无需被动编程。

*Last updated: June 26, 2019*