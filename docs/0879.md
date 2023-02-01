# Quarkus:为什么要编译成 native？

> 原文：<https://developers.redhat.com/blog/2019/03/29/quarkus-why-compile-to-native>

Quarkus 是 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 的原生产品，为了实现这一目标，我们已经花费了大量时间在多个不同领域开展工作，例如 Java 虚拟机(JVM)和各种框架优化。而且，还有很多工作要做。引起开发人员兴趣的一个领域是 Quarkus 的全面和无缝的方法，从您的 [Java](https://developers.redhat.com/topics/enterprise-java/) 代码生成特定于操作系统的(也称为本机)可执行文件，就像您对 C 和 C++等语言所做的那样，我们相信它们通常会在构建-测试-部署周期的末尾使用。

虽然原生编译很重要，但正如我们将在后面讨论的，Quarkus 与普通 OpenJDK Hotspot 配合得非常好，这要归功于我们对整个堆栈所做的显著性能改进。Quarkus 提供的本机可执行方面是可选的，如果您不想要它或者您的应用程序不需要它，那么您可以忽略它。事实上，即使在使用原生映像时，Quarkus 仍然严重依赖 [OpenJDK](https://developers.redhat.com/products/openjdk/overview/) 。由于 Hotspot 丰富的动态代码执行能力，广受欢迎的开发模式能够交付近乎即时的变更测试周期。此外， [GraalVM](https://www.graalvm.org/) 在内部使用 OpenJDK 的类库和 HotSpot 来生成一个原生映像。

尽管如此，仍然有一个问题:如果其他优化如此之好，为什么还要进行原生编译呢？这是我们将在这里更深入探讨的问题。

让我们从显而易见的开始: [JBoss](http://www.jboss.org/) 和 Red Hat 在了解如何优化 JVM、堆栈和框架方面有着悠久的历史记录，包括:

*   第一个在云上运行的应用服务器 [Red Hat OpenShift](https://www.redhat.com/en/blog/openshift-is-the-first-paas-to-support-java-ee-6-full-profile-in-the-cloud)
*   第一个运行在[插件计算机](https://developers.redhat.com/index.php/videos/youtube/9c-g7UxT0nk//)上的应用服务器。
*   第一个运行在树莓平台上的应用服务器。
*   在 Android 上运行我们的许多[项目。](http://www.jboss.org/iot/android/)

正如这里所展示的，我们多年来一直致力于在云和受限设备(即物联网)上运行 Java，一直在寻找如何从 JVM 中挤出下一个性能或内存优化。我们和其他人长期以来一直在关注编译 Java 的潜力，无论是通过 [GCJ](https://gcc.gnu.org/wiki/GCJ) 、[艾维恩](https://readytalk.github.io/avian/)、 [Excelsior JET](https://www.excelsiorjet.com/) ，甚至是[达尔维克](https://source.android.com/devices/tech/dalvik)，因为我们了解其中的权衡(例如，缺乏“随处运行一次”的构建与更少的磁盘空间或更快的启动时间)。

为什么这些权衡很重要？答案是，对于一些重要的场景，它完全有意义:

*   考虑无服务器/事件驱动的环境，我们需要[实时启动服务](https://mikhail.io/2018/08/serverless-cold-start-war/)(软或硬)来对事件做出反应。与持续服务不同，冷启动成本会延长对请求的响应时间。今天，JVM 仍然需要“一段时间”才能启动，尽管在某些情况下，用硬件解决问题会有所帮助，但 1 秒和 5 毫秒之间的差别可能是生死攸关的。是的，我们可以玩诸如热备份 JVM 之类的把戏(例如，我们已经用 OpenWhisk 的[虚拟端口](https://github.com/projectodd/kwsk)做到了这一点)，但这并不能保证当请求增加时，你会有足够的 JVM 等待处理请求，当你要为那些并不总是被使用的进程付费时，这听起来也不是一个省钱的好方法。
*   然后是我们经常听到的多租户方面。尽管 JVM 已经发展到可以复制许多操作系统的功能，但它仍然没有我们在 Linux 等操作系统中认为理所当然的那种进程隔离，并且一个线程的故障可能会导致整个 JVM 瘫痪。许多人通过在单个 JVM 中只运行单个用户的应用程序来解决这个问题。这样做是合理的:因此，失败的单位是 JVM，一个用户的失败不一定会影响任何其他用户。然而，为每个用户运行一个 JVM 经常会带来大规模的问题。
*   密度对云原生应用也很重要。拥抱 [12 因素](https://12factor.net/)应用、微服务和 Kubernetes 导致每个应用有许多 JVM。虽然您获得了灵活性和健壮性，但是每个服务的基本内存占用成本开始增加，即使该成本的一部分并不是绝对必要的。静态编译的可执行文件能够受益于封闭世界的优化，例如细粒度的死代码消除，其中只有服务实际使用的框架部分(包括 JDK 本身)包含在结果映像中。通过将应用程序定制为本机友好的，Quarkus 可以用于在主机上密集打包许多服务实例，而不会损害安全性。

仅仅因为这些原因，本地可执行选项为我们和我们社区中的其他人提供了一个有效的解决方案。还有另一个不太技术性的原因，也同样重要:在过去的几年里，许多 Java 开发人员和公司已经放弃了它，转而支持新的语言，通常是因为他们认为 JVM、堆栈和框架臃肿、缓慢等等。

试图将一把现有的锤子强行安装到新钉子上并不总是最好的方法，有时最好退一步考虑一种新工具。如果 Quarkus 让人们停下来重新思考，那么对于整个 Java 生态系统来说是一件好事。Quarkus 对如何交付更高效的应用程序采取了创新的观点，使 Java 与以前被认为是禁忌的应用程序架构(如无服务器)相关联。此外，通过它的扩展能力，我们希望看到一个 Quarkus Java 扩展生态系统，它提供了一个大的框架集，可以与您的应用程序一起开箱即用地进行本地编译。

*Last updated: September 3, 2019*