# JDK 飞行记录器对 GraalVM 本机映像的支持:迄今为止的历程

> 原文：<https://developers.redhat.com/articles/2021/07/23/jdk-flight-recorder-support-graalvm-native-image-journey-so-far>

在过去的一年里，Oracle 和 Red Hat 的工程师们共同努力，将 [JDK 飞行记录器](/blog/2020/08/25/get-started-with-jdk-flight-recorder-in-openjdk-8u) (JFR)的支持带到了 [GraalVM 原生映像](https://www.graalvm.org/reference-manual/native-image/)。原型拉取请求最初是由[红帽](https://github.com/oracle/graal/pull/3070)和[甲骨文](https://github.com/oracle/graal/pull/3155)在 2020 年末和 2021 年初提出的。然后，工作继续在[共享存储库](https://github.com/native-image-jfr/graal/tree/jfr)中进行，计划贡献一个新的包含各方工作的拉请求。2021 年 6 月，我们将[第一个拉请求](https://github.com/oracle/graal/pull/3444)与 OpenJDK 11 upstream 合并，该请求引入了在 GraalVM 上支持 JDK 飞行记录器的基础设施。它在 GraalVM 的 21.2 版本中可用。这篇文章分享了这个故事背后的一些细节:什么是原生图像，我们为什么要添加 JDK 飞行记录器，我们面临的一些技术挑战，以及我们下一步打算做什么。

## GraalVM 本机映像的性能分析

GraalVM 是一个高性能的运行时，它支持生成 [Java](/topics/enterprise-java) 应用程序的[本机映像](https://www.graalvm.org/reference-manual/native-image/)。这个特性获取 Java 类文件，并为目标平台生成一个二进制可执行文件。最终的本机可执行文件包含一个用 Java 编写的运行时系统，称为 SubstrateVM，它包括内存管理和线程调度等组件。这类似于 [OpenJDK](/products/openjdk) 如何使用 HotSpot 虚拟机(VM)来执行 Java 应用。本机可执行文件通过迭代的指向分析和堆填充生成，随后是[提前编译](https://dl.acm.org/action/cookieAbsent)。与运行在传统 JVM(如 HotSpot)上的 Java 应用程序相比，GraalVM 生成的本机可执行文件的启动时间明显更快。

有望获得更好性能的二进制可执行文件仍然需要一个支持工具生态系统来进行监控和性能分析。但是，目前，OpenJDK 上通常用于 Java 应用程序的大多数监控功能对于 GraalVM 上的 GraalVM 本机映像都不可用。如果没有工具来检查应用程序执行和运行时执行层，诊断应用程序性能问题是很困难的。这种忽略对于在生产中部署的应用程序来说至关重要，无论它们是在[容器](/topics/containers)中还是在裸机上。

## 为什么是 JDK 飞行记录器？

JDK 飞行记录器是 Hotspot VM 中的一个分析系统，它通过基于事件的日志记录系统提供 JVM 内部数据和定制的、特定于应用程序的数据。JDK 飞行记录器旨在以最小的性能开销收集生产环境中的关键数据。传统上，输出是 JFR 文件(.jfr)，它可以被像 [JDK 任务控制](https://github.com/openjdk/jmc)这样的工具读取。在 JDK 14 和更高版本中，数据也可以通过 Java API 进行流式传输。

让 JDK 飞行记录器支持 GraalVM 本机映像将为用户提供一个强大的性能分析工具，类似于 HotSpot VM 体验。理想情况下，使用 JDK 飞行记录器在 OpenJDK 上分析其 Java 应用程序的开发人员将会有相同的使用它作为本机可执行文件的体验。用于原生映像的 JFR 将是 SubstrateVM 中的一个分析系统，它提供 SubstrateVM 内部数据和定制的特定于应用程序的数据，具有与 HotSpot 中相同的 JFR 文件和流功能。

GraalVM 原生映像的 JFR 将提供 SubstrateVM 系统的内部数据，模仿 HotSpot 中的许多数据类型。这包括有关垃圾收集操作、线程状态、监视器状态、异常、安全点、对象分配、文件或套接字 I/O 事件等信息。

### 使用内部和自定义事件进行分析

热点中的 JDK 飞行记录器和其他剖析工具帮助我们回答了各种各样的问题。GraalVM 原生映像中的 JDK 飞行记录器旨在做同样的事情。这些问题包括:

*   垃圾收集器消耗了多少执行时间？
*   在 safepoint，stop-the-world 操作中花费了多少时间？
*   执行流程中是否存在瓶颈，它们在哪里？
*   我的申请中有哪些热门方法？
*   在 I/O 处理上花费了多少时间？
*   像死锁和等待锁的操作这样的同步问题怎么办？

除了内部事件之外，JDK 飞行记录器还有一个 API，供开发人员将他们自己的事件添加到系统中。这使用户能够添加更多数据进行分析，从而受益于现有的低开销、面向生产的 JFR 基础架构。与单独的、定制的、事件或度量发射代理相比，定制事件可以利用运行时常数(类名、方法名、字符串等)的公共池来降低总体输出成本。

### 解决性能问题

事件本身不足以诊断性能问题。幸运的是，现有的 JFR 格式让我们能够快速集成可视化和分析工具，如 JDK 任务控制。我们可以一起使用这些工具来解决本地图像应用程序中的性能问题。SubstrateVM 中与 HotSpot 中的事件一对一匹配的任何事件都将被理解，而无需对现有分析工具进行任何更改。

**注意**:HotSpot 的 JDK 飞行记录器有大量的资源，我在文章末尾列出了其中一些。请看看，了解更多关于 JDK 飞行记录器的一般信息，以及它为解决性能问题带来的好处。

## 实现 JDK 飞行记录器对 GraalVM 本机映像的支持

在我们的工作之前，没有 JDK 飞行记录器对 GraalVM 本机映像的支持，我们在实现这种支持时遇到了许多挑战。虽然 Java API 是可访问的，但是没有管理 JDK 飞行记录器系统的机制。此外，直接使用 JDK 飞行记录器 Java API 的现有代码，例如，[来开始记录](https://github.com/jiekang/sandbox/blob/java-jfr-app/src/main/java/com/redhat/jkang/Main.java)，失败，并出现[未满足链接错误](https://gist.github.com/jiekang/ae928a7bbe4dafeb05b5e7a97d0174d1)。失败的原因是 JDK 飞行记录器本地 API 不存在。出现这个错误是因为在 HotSpot 库`libjvm.so`中找到的本地 JDK 飞行记录器代码没有包含在 SubstrateVM 系统中。不幸的是，本机代码不适合包含在内，因为它与 HotSpot 内部有很深的联系。

为了将 JDK 飞行记录器添加到 GraalVM 本机映像中，我们完全用 SubstrateVM 中的 Java 代码在 HotSpot VM 中重新实现了 JDK 飞行记录器的本机基础结构。然而，我们无法将用 C++编写的 HotSpot 代码一对一地翻译成用 Java 编写的 SubstrateVM。本节解释了一些原因。

### 阶级转变

首先，HotSpot 中的 event 类[名义上是空的](https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/jdk.jfr/share/classes/jdk/jfr/Event.java#L100)，类转换在运行时延迟完成，以便为启用的事件的方法添加实现。这些方法的主体为空，并被标记为 final，以防止扩展。查看 [EventInstrumentation](https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/jdk.jfr/share/classes/jdk/jfr/internal/EventInstrumentation.java#L328) 中的代码，看看事件转换后的实现是什么样子。

SubstrateVM 中需要转换的事件类来发出事件，但对于本机映像，类转换无法在运行时完成。OpenJDK JDK 飞行记录器通过 Java 虚拟机工具接口(JVMTI)将布尔标志[重新转换为仪器类。默认值为 true。为了解决这个问题，我们可以将正在运行的用于 SubstrateVM 分析的 JVM 的值设置为 false，例如通过](https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/hotspot/share/jfr/recorder/service/jfrOptionSet.cpp#L249)

```
 $ mx native-image -J-XX:FlightRecorderOptions=retransform=false ... 
```

SubstrateVM 看到的结果事件类将包含转换后的实现，就好像事件被启用并且 JDK 飞行记录器正在运行一样。

另一个解决方案是通过使用 [jdk.jfr.internal.JVM](https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/jdk.jfr/share/classes/jdk/jfr/internal/JVM.java#L206) 中的内部 API `retransformClasses`，在编译之前[转换事件类。这个变通方法也依赖于 HotSpot JDK 飞行记录器的实现细节，但是它允许对转换哪些类进行更细粒度的控制。](https://gist.github.com/jiekang/d3c0a7e11b689f57950ec009c5f8fff4#file-jfreventsubstitution-java-L80)

### 不间断安全点

默认情况下，本机代码(HotSpot)不能被 safepoint 中断，而 Java 代码(SubstrateVM)可以。幸运的是，SubstrateVM 有[不间断注释](https://github.com/oracle/graal/blob/master/substratevm/src/com.oracle.svm.core/src/com/oracle/svm/core/annotate/Uninterruptible.java),可以用来标记不会因为安全点而中断的方法块。将所有“本机”部分标记为不可中断是不理想的，因为性能会受到这些部分的影响。同样，HotSpot 中的一些 JDK 飞行记录器代码路径被注释为“ [can safepoint here](https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/hotspot/share/jfr/recorder/checkpoint/jfrCheckpointManager.cpp#L403) ”(并被适当标记以允许安全指向)，因此我们仍然需要仔细关注 SubstrateVM 实现，以确保保持一致性。

使用`Uninterruptible`注释也有一个限制:不允许对象分配。当在 Java(一种面向对象的编程语言)中实现熟悉的代码模式时，这被证明是很成问题的。甚至日志记录也很困难，因为任何分配的`toString`方法在不可中断的方法调用中都是不可用的。这使得调试比正常情况下更痛苦，尤其是在处理并发问题时，简单的调试器策略看不到与被寻找的问题相同的执行流。

此外，SubstrateVM 的一些关键部分，如垃圾收集器，具有被标记为不可中断的方法。我们必须能够在这些位置触发事件发射基础设施，因此其组件也需要不间断。这要求它们不分配 Java 对象。幸运的是，SubstrateVM 提供了`malloc`或释放内存的方法，这些方法[不由垃圾收集](https://github.com/oracle/graal/blob/master/sdk/src/org.graalvm.nativeimage/src/org/graalvm/nativeimage/UnmanagedMemory.java#L57)管理。但是，这些实例没有扩展`Object`，所以任何现有的 Java API，比如`ArrayList`，都不能用来操作数据。如果需要的话，这些数据结构必须重新实现。

### JFR 记录器线程

写数据的一些关键工作交给了 HotSpot 中的一个线程，JFR `Recorder`线程，它本身被排除在 JDK 飞行记录器系统之外。这种排斥不能在 SubstrateVM 中模仿，因为任何线程都可以被用来执行虚拟机操作，如垃圾收集。垃圾收集器会发出相关的 JDK 飞行记录器事件，排除会导致此类操作中的事件数据丢失。

甚至有可能为了垃圾收集操作而暂停数据块的关闭(其中事件被写入磁盘，epoch 被转换，元数据和常量池被发出)。然后，此操作可以将垃圾收集事件写入缓冲区。这里的任务需要进行分析，并适当地标记为不可中断的，以保持这些垃圾收集事件及其常量池数据的一致性。我在前面描述的关闭块的写操作需要小心地写成一个安全点操作，具有不可中断的部分，以考虑占用垃圾收集的可能性。

## 结论

JDK 飞行记录器是 HotSpot VM 的一个重要特性，它为用户提供了大量关于 JVM 执行的数据，并且开销很低，适合在生产环境中连续运行。当使用 SubstrateVM 运行本机映像应用程序时，开发人员可以使用相同的工具，这将非常有用。

JFR 基础设施的初始合并已经完成，但在系统能够提供由 GraalVM 生成的本地可执行文件视图(类似于 HotSpot 的视图)之前，还有很长的路要走。接下来的工作是为垃圾收集、线程、异常和 SubstrateVM 中其他有用的位置添加事件。当前没有为 SubstrateVM 实现 [RJMX API](https://docs.oracle.com/cd/E15289_01/JRJVD/com/jrockit/mc/rjmx/package-summary.html) ，所以没有 API 来远程管理 JDK 飞行记录器。与此同时，JFR 系统仍在 OpenJDK 和 HotSpot 中得到显著改进和增强。我们需要考虑影响 SubstrateVM 实现中 API 的主要变化，以正确支持不同的底层 OpenJDK 版本。此时，代码库仅针对 OpenJDK 11。

现在，您可以通过在构建时将标志`-H:+AllowVMInspection`传递给`native-image`进程来试用最新的 GraalVM 和测试 JDK 飞行记录器。完成这些之后，您可以在运行时向应用程序二进制文件添加诸如`-XX:+FlightRecorder -XX:StartFlightRecording="filename=recording.jfr"`之类的标志。

除了 GraalVM， [Mandrel](/blog/2021/04/14/mandrel-a-specialized-distribution-of-graalvm-for-quarkus) ，为 [Quarkus](/products/quarkus/) 的社区分布，自然会继承 JFR 的支持。随着我们不断改进和增强 GraalVM 本机映像的 JDK 飞行记录器，请关注更多关于在 Quarkus 和 Mandrel 环境中使用本机映像的 JFR 的公告和文章。

## 了解更多关于 JDK 飞行记录器

以下资源可用于了解有关 JDK 飞行记录器的更多信息，并使用它来解决性能问题:

*   【Java 任务控制和飞行记录器中间件应用监控简介 (Mario Torre 和 Marcus Hirt，FOSDEM 2019)
*   [开始使用 OpenJDK 8u 中的 JDK 飞行记录器](/blog/2020/08/25/get-started-with-jdk-flight-recorder-in-openjdk-8u) (Mario Torre，Red Hat 开发人员)
*   使用 Java 飞行记录器与 JDK 11 号
*   [使用 JFR](https://docs.oracle.com/javase/10/troubleshoot/troubleshoot-performance-issues-using-jfr.htm#JSTGD299) 解决性能问题( *Java 平台，标准版故障排除指南*，Oracle)
*   [JDK 飞行记录仪——藏在 OpenJDK](https://bell-sw.com/announcements/2020/06/24/Java-Flight-Recorder-a-gem-hidden-in-OpenJDK/) 中的宝石(贝尔软件)
*   [JDK 11——JDK 飞行记录仪简介](https://youtu.be/7z_R2Aq-Fl8)(Markus grnlund，甲骨文 YouTube)
*   [使用自定义 JDK 飞行记录器事件监控 REST APIs】(贡纳·莫林)](https://www.morling.dev/blog/rest-api-monitoring-with-custom-jdk-flight-recorder-events/)
*   [集装箱介绍 JFR: JDK 集装箱飞行记录仪](/blog/2021/01/25/introduction-to-containerjfr-jdk-flight-recorder-for-containers) (Andrew Azores，红帽开发者)

## 承认

作者感谢 Alina Yurenko 和 Oracle 花时间阅读本文。

*Last updated: January 5, 2023*