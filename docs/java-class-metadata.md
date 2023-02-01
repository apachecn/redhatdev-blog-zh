# Java 类元数据:用户指南

> 原文：<https://developers.redhat.com/blog/2018/02/14/java-class-metadata>

上周我在[免费 Java 室](https://fosdem.org/2018/schedule/track/free_java/)的 [FOSDEM 2018](https://fosdem.org/2018/) 上做了一个关于 Java 类元数据的[演讲。在我的演讲中，我解释道:](https://fosdem.org/2018/schedule/event/class_metadata/)

1.  什么是 Java 类元数据
2.  为什么了解它会有帮助
3.  如何度量它并减少元数据对 Java 应用程序的影响

“太久了——没读”的总结是:

1.  它是 Java 在运行时为了动态加载、链接、JIT 编译和执行 Java 代码而保留的加载类库模型。
2.  编写代码时所做的不同设计选择会显著增加或减少 Java 需要保留的元数据数量。
3.  JVM 可以为建模每个加载的类的单个结构提供元数据存储成本的明细，允许您权衡和比较备选设计的成本。

FOSDEM 团队已经将演讲的[视频放到了网上。它被限制在 25 分钟内，这足以让您开始，但不足以解释如何进行真正精细的分析和调优。正如我在演讲开始时所承诺的，我现在已经发表了这篇演讲的](https://www.youtube.com/watch?v=jsJtZdYhQuE)*和* [原文 4 篇](https://github.com/adinn/fosdem2018)。这些文章介绍了这个主题，并详细解释了如何测量和分析 JVM 统计数据。包括基于 Wildfly 代码库的示例分析，以及列出所有可用统计数据以及如何解释它们的建议的完整参考资料。

请检查一下！

**资源:**

*   [观看演讲视频](https://www.youtube.com/watch?v=jsJtZdYhQuE)
*   [观看在 FOSDEM 上展示的幻灯片](https://github.com/adinn/fosdem2018/blob/master/FOSDEM2018_Metadata.pdf)
*   [阅读演讲所依据的四篇文章](https://github.com/adinn/fosdem2018)