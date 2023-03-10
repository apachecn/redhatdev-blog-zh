# 谢南多厄 GC 在 JDK 13，第 3 部分:架构和操作系统

> 原文：<https://developers.redhat.com/blog/2019/07/01/shenandoah-gc-in-jdk-13-part-3-architectures-and-operating-systems>

在这个系列中，我一直在报道 [JDK 13](https://developers.redhat.com/products/openjdk/overview/) 即将到来的[谢南多厄 GC](https://developers.redhat.com/videos/youtube/N0JTvyCxiv8/) 的新进展。在[第 1 部分](https://developers.redhat.com/blog/?p=602377)中，我查看了加载参考屏障的开关，在[第 2 部分](https://developers.redhat.com/blog/?p=606477)中，我查看了为每个对象消除一个额外单词的计划。在本文中，我将研究 Shenandoah GC 将使用的新架构和新操作系统。

## Solaris

BellSoft [最近贡献了一个变化](https://mail.openjdk.java.net/pipermail/shenandoah-dev/2019-May/009556.html)，允许 Shenandoah 在 Solaris 上构建和运行。Shenandoah 本身没有特定于操作系统的代码；因此，移植到新的操作系统相对容易。在这种情况下，它主要相当于一批让 Solaris 编译器满意的修复，比如删除枚举中的尾随逗号。

我们遇到的一个值得注意的问题是 Solaris 10。与更高版本的 Solaris 所做的(以及基本上所有其他相关操作系统所做的)相反，Solaris 10 将用户内存映射到更高的地址范围(例如，从 0xff 开始的地址...而不是 0x7f)。其他操作系统将地址空间的上半部分保留给内核内存。

这种方法与 Shenandoah 的任务队列的优化相冲突，后者会在假设它在较高的地址范围内有一些空闲空间的情况下对指针进行编码。很容易通过构建时标志来禁用，阿列克谢·希皮列夫做到了。[修复](http://hg.openjdk.java.net/jdk/jdk/rev/f2f11d7f7f4e)完全在 Shenandoah GC 内部，不影响堆中 Java 引用的表示。通过这一更改，Shenandoah 可以在 Solaris 10 和更高版本(也可能更旧，但我们没有尝试过)上构建和运行。这不仅对那些希望 Shenandoah 在 Solaris 上运行的人有意思，对我们也有意思，因为它需要额外的清洁来使非主线工具链满意。

对 Solaris 支持的更改已经出现在 JDK 13 开发仓库中，并且已经反向移植到谢南多厄的 JDK 11 和 JDK 8 反向移植仓库中。

## x86_32

Shenandoah 在很久以前曾经以“被动”模式支持 x86_32。这种模式只依赖 stop-the-world GC 来避免实现壁垒(基本上就是一直运行退化 GC)。这是一个有趣的模式，可以看到通过非常小的微服务规模的虚拟机使用非提交和更细的本机指针可以获得的内存占用量。这种模式在集成上游之前就被放弃了，因为许多 Shenandoah 测试期望所有的启发式/模式都能正常工作，并且拥有基本的 x86_32 支持会破坏 tier1 测试。所以，我们关闭了它。

今天，由于[加载引用障碍](https://developers.redhat.com/blog/?p=602377)和[消除了单独的转发指针 slo](https://developers.redhat.com/blog/?p=606477) t，我们已经显著简化了运行时接口，并且我们可以在此基础上构建完全并发的 x86_32。这种方法允许我们在 Shenandoah 代码中保持 32 位的清洁度(在这次改变之前，我们已经修复了>的 5 个 bug！)，并且它作为 Shenandoah 可以在 32 位平台上实现的概念证明。在额外节省空间很重要的场景中，比如在容器或嵌入式系统中，这是很有趣的。LRB +无转发指针+ 32 位支持的组合为我们提供了目前使用 Shenandoah 可能达到的最低内存占用范围。

对 x86_32 位支持的更改已经完成，并准备集成到 JDK 13 中。然而，他们目前正在等待消除转发指针的变化，这反过来又在等待一个讨厌的 C2 错误修复。该计划是在加载参考障碍和消除转发指针变化被反向移植之后，稍后将其反向移植到谢南多厄 JDK 11 和 JDK 8 反向移植。

## 其他架构和操作系统

随着对 OS 和架构支持的这两个增加，Shenandoah 将很快在四个操作系统上可用(例如，已知的构建和运行): Linux、Windows、MacOS 和 Solaris，加上三个架构:x86_64、arm64 和 x86_32。鉴于 Shenandoah 的设计没有特定于操作系统的代码，也没有过于复杂的特定于架构的代码，我们可能会在未来的版本中加入更多的操作系统或架构(如果有人觉得实现起来足够有趣的话)。

和往常一样，如果你不想等待发布，你可以拥有一切并帮助解决问题:查看 Shenandoah GC Wiki 。

### 阅读更多

[JDK 谢南多厄 GC 13，第 1 部分:荷载参考屏障](https://developers.redhat.com/blog/?p=602377)

[JDK 13 中的 Shenandoah GC，第 2 部分:删除前向指针单词](https://developers.redhat.com/blog/?p=606477)

*Last updated: June 27, 2019*