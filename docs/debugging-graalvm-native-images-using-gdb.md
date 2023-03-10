# 使用 gdb 调试 GraalVM 本机映像

> 原文：<https://developers.redhat.com/blog/2020/06/25/debugging-graalvm-native-images-using-gdb>

GraalVM 项目除了其他功能之外，还包括一个名为 GraalVM 本地映像的组件。GraalVM 原生映像支持将 [Java](https://developers.redhat.com/topics/enterprise-java/) 应用交付为收缩包装、自包含、独立的可执行文件，通常称为 Java 原生映像。与在 JVM 上以传统方式运行相同的应用程序相比，本机映像通常占用空间更小，启动时间更快。对于短期运行的应用程序或小型的基于[容器](https://developers.redhat.com/topics/containers/)的服务来说，这通常是一个优势。对于长时间运行的程序来说，代价通常是较低的峰值[性能](https://developers.redhat.com/blog/category/performance/)，对于具有大量常驻数据的程序来说，代价是较高的垃圾收集开销和延迟。

我们对 GraalVM 原生映像特别感兴趣，它是基于 [Quarkus](https://quarkus.io/) 的应用程序的备选后端交付选项。Java 团队努力确保 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 与 GraalVM 原生映像很好地集成。在这个过程中，他们发现一个重要的可用性问题是调试交付的本机映像的能力。

当然，这主要不是一个发展问题。在开发和测试期间，调试应用程序的大部分艰苦工作都可以在 JVM 上完成。但是，如果部署的本机映像行为不同，总会有该怎么办的问题。虽然这种情况不应该发生，但由于应用程序中的错误或配置构建时的问题，这种情况可能会发生。在极少数情况下，由于在将应用程序代码或 JDK 运行时代码编译成本机代码时，本机编译过程会引入差异，因此部署的本机映像可能会有不同的行为。

调试之所以困难，是因为生成的本机映像是高度优化的代码，只有最少的符号信息。许多原始的 Java 方法代码是内联的，因此很难将特定的指令与它们的原始 Java 方法联系起来，更不用说 Java 源文件中的特定代码行了。这至少和在最高优化级别编译的没有任何符号信息的 C 或 C++程序一样难调试。

当然，C/C++编译器通过将 debuginfo 嵌入到二进制文件中来解决这个问题。这些信息准确地告诉调试器如何解释生成的代码，并将其与原始源代码联系起来。因此，我最近决定通过向 Linux 映像添加 DWARF 调试信息来解决 GraalVM 本机映像问题。这足以使用标准的 Linux 调试器 gdb 进行有效的源代码级调试。用于调试 Windows 二进制文件的类似解决方案正在开发中。

调试器现在知道已经编译成映像的 Java 类和方法。断点可以放在方法入口或方法中的特定行上。调试器可以将本机映像中的机器指令与 Java 文件中的特定源代码行相关联。它可以停止并逐行通过方法代码，并显示调用堆栈。调试器甚至知道指令是否属于内联方法，并在您单步执行代码时，在外部编译方法的源代码行和内联源代码之间来回切换。

这个调试功能现在已经出现在最新的 GraalVM 版本中。我最近上传了一个[的视频演示](https://www.youtube.com/watch?v=JqV-NFWupLA)到[夸尔库斯频道](https://www.youtube.com/channel/UCaW8QG_QoIk_FnjLgr5eOqg/videos)，更精确地解释了已经实施的内容。它包括一个基于小 Java 程序的演示，展示了如何构建一个包含调试信息的本地映像，并在 gdb 中运行/调试它。我希望你喜欢这个演示，并欢迎对可用性或调试时遇到的任何错误的反馈。

*Last updated: January 24, 2022*