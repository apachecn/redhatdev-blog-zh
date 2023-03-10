# JDK 谢南多厄 GC 13，第 1 部分:荷载参考屏障

> 原文：<https://developers.redhat.com/blog/2019/06/27/shenandoah-gc-in-jdk-13-part-1-load-reference-barriers>

在这一系列文章中，我将介绍即将在 [JDK 13](https://developers.redhat.com/products/openjdk/overview/) 举行的 [Shenandoah GC](https://developers.redhat.com/videos/youtube/N0JTvyCxiv8/) 的一些新进展。尽管用户无法直接看到，但最重要的变化可能是 Shenandoah 的障碍模型转换为加载参考障碍。这一变化解决了对谢南多厄的一个主要批评点——昂贵的原始阅读障碍。在这里，我将进一步解释这个变化意味着什么。

Shenandoah(以及其他收集器)使用屏障来确保堆的一致性。更具体地说，Shenandoah GC 使用屏障来确保我们所说的“空间不变性”。这意味着当 Shenandoah 进行收集时，它将对象从所谓的“from-space”复制到“to-space”，并且是在 Java 线程运行时(并发地)进行的。

因此，在 JVM 中可能有任何对象的两个副本。为了保持堆的一致性，我们需要确保:

*   写入到空间拷贝+读取可以从两个拷贝进行，受内存模型约束=弱空间不变量，或者
*   写入和读取总是发生在到空间副本=强到空间不变量中。

我们确保这一点的方法是，每当发生读取和写入时，采用相应类型的屏障。考虑这个伪代码:

```
void example(Foo foo) {
  Bar b1 = foo.bar;             // Read
  while (..) {
    Baz baz = b1.baz;           // Read
    b1.x = makeSomeValue(baz);  // Write
}

```

使用 Shenandoah 壁垒，它看起来像这样(JVM+GC 在幕后会做什么):

```
void example(Foo foo) {
  Bar b1 = readBarrier(foo).bar;             // Read
  while (..) {
    Baz baz = readBarrier(b1).baz;           // Read
    X value = makeSomeValue(baz);
    writeBarrier(b1).x = readBarrier(value); // Write
}
```

换句话说，无论我们从哪里读取一个对象，我们首先通过一个读屏障解析该对象，无论我们从哪里写入一个对象，我们都可能将该对象复制到 to-space。这里就不赘述了；这么说吧，这两种操作都有些成本。

还要注意，我们需要在 write 的值上设置一个读屏障，以确保我们只在堆引用更新时将空间引用写入字段(这是 Shenandoah 的旧屏障模型的另一个麻烦)。

因为这些障碍是一件昂贵的事情，我们非常努力地优化它们。一个重要的优化是将障碍物吊出环路。在这个例子中，我们看到 b1 被定义在循环之外，但只在循环内部使用。我们也可以在循环外设置障碍，一次，而不是在循环内多次:

```
void example(Foo foo) {
  Bar b1 = readBarrier(foo).bar;  // Read
  Bar b1' = readBarrier(b1);
  Bar b1'' = writeBarrier(b1);
  while (..) {
    Baz baz = b1'.baz;            // Read
    X value = makeSomeValue(baz);
    b1''.x = readBarrier(value);  // Write
}
```

而且，因为写屏障比读屏障更强，所以我们可以将两者折叠起来:

```
void example(Foo foo) {
  Bar b1 = readBarrier(foo).bar; // Read
  Bar b1' = writeBarrier(b1);
  while (..) {
    Baz baz = b1'.baz;           // Read
    X value = makeSomeValue(baz);
    b1'.x = readBarrier(value);  // Write
}
```

这一切都很好，工作得相当好，但也很麻烦，因为优化过程非常复杂。任何对象的一个空间和两个空间副本都可以在任何时候在 JVM 中浮动，这是令人头痛和复杂的主要原因。例如，我们需要额外的屏障来比较对象，以防我们将一个对象与其自身的不同副本进行比较。需要为任何读取或写入插入读取屏障和写入屏障，包括非常频繁的原始读取或写入。

那么，为什么不对此进行优化，并在从内存中加载对象时确保空间不变性呢？这就是负载参考屏障的用武之地。它们的工作方式很像我们以前的写屏障，但并不用于使用场所(从对象读取或向对象存储时)。相反，当加载对象时(在它们的定义站点)，它们被更早地使用:

```
void example(Foo foo) {
  Bar b1' = loadReferenceBarrier(foo.bar);
  while (..) {
    Baz baz = loadReferenceBarrier(b1'.baz); // Read
    X value = makeSomeValue(baz);
    b1'.x = value;                           // Write
}
```

你可以看到代码基本上和之前一样——在我们优化之后——除了我们还不需要优化任何东西。此外，store-value 的读屏障也消失了，因为我们现在知道(由于强大的空间不变性),无论 makeSomeValue()做了什么，如果需要的话，它一定已经使用了 load-reference-barrier。新的负载参考屏障与我们之前的写入屏障几乎完全相同。

这种屏障模型有很多优点(对于我们 GC 开发人员来说):

*   强不变量意味着更容易推断 GC 和对象的状态。
*   更简单的屏障界面。事实上，我们在 JDK11 之后添加到 GC barrier 接口的很多东西现在都没有用上:不需要原语上的屏障，不需要对象平等屏障，等等。
*   优化就容易多了(见上)。屏障自然地被放置在最不热的位置:它们的定义位置，而不是它们最热的位置:它们的使用位置，然后试图从那里优化它们(并不总是成功)。
*   不再需要物体就等于障碍。
*   不再需要“解析”屏障(一种有点奇怪的屏障，主要用于内部函数和类似读或写操作的地方)。
*   所有的障碍现在都是有条件的，这为以后的进一步优化提供了机会。
*   我们可以重新启用一些优化，比如之前需要禁用的快速 JNI 获取器，因为它们不能很好地处理可能的来自空间的引用。

对于用户来说，这种变化大多是不可见的，但底线是它提高了 Shenandoah 的整体性能。它还为其他改进开辟了道路，比如取消转发指针，我将在后续文章中讨论这一点。

负载参考障碍[于 2019 年 4 月整合到 JDK 13 开发库](https://bugs.openjdk.java.net/browse/JDK-8221766)中。我们将很快开始将其反向移植到谢南多厄的 JDK 11 号和 JDK 8 号。如果你不想等待，你已经可以拥有它了:查看[的 Shenandoah GC Wiki](https://wiki.openjdk.java.net/display/shenandoah/Main) 了解详情。

### 阅读更多

[JDK 13 中的 Shenandoah GC，第 2 部分:删除前向指针单词](https://developers.redhat.com/blog/?p=606477)

[JDK 2013 年 Shenandoah GC，第 3 部分:架构和操作系统](https://developers.redhat.com/blog/?p=606497)

*Last updated: July 1, 2019*