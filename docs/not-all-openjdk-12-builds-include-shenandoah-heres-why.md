# 并非所有 OpenJDK 12 版本都包含 Shenandoah:原因如下

> 原文：<https://developers.redhat.com/blog/2019/04/19/not-all-openjdk-12-builds-include-shenandoah-heres-why>

[OpenJDK 12 现在出了](https://openjdk.java.net/projects/jdk/12/)，有了新的特性。这些是:

189: Shenandoah:低暂停时间垃圾收集器(实验性)
230:微基准测试套件
325:开关表达式(预览)
334: JVM 常量 API
340:一个 AArch64 端口，而不是两个
341:默认 CDS 档案
344:可接受的 G1 混合收集
346:立即从 G1 返回未使用的提交内存

当我从 OpenJDK 12 项目页面链接到[开源构建页面](https://jdk.java.net/12/)时，我看到了可下载的二进制文件。我下载并安装 Linux 二进制文件，然后看看特性列表上的第一项 Shenandoah 是否有效:

```
$ ./jdk-12/bin/java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -cp ~ Hello
Error occurred during initialization of VM
Option -XX:+UseShenandoahGC not supported
```

哦！这是怎么回事？

一点历史: [Shenandoah](https://wiki.openjdk.java.net/display/shenandoah/Main) ，一个高性能低暂停时间垃圾收集器，是 Red Hat 领导的项目。当我们第一次提议将 Shenandoah 贡献给 OpenJDK 时，Oracle 明确表示[他们不想支持它](https://bugs.openjdk.java.net/browse/JDK-8215030)。这很公平:OpenJDK 是自由软件，所以你不必支持任何你不想要的东西。我们告诉 Oracle，我们将与他们合作设计一个真正干净的可插拔垃圾收集器接口，允许任何人轻松地选择将垃圾收集器包含在他们的构建中。我们一起做到了，谢南多阿进入了 JDK 12 号。

很明显，甲骨文公司选择不建造雪兰多。他们排除它并没有做任何严格意义上的错事，但是我觉得有些事情不对劲。Oracle 不支持这些版本——您需要它们的商业二进制文件来获得支持——那么为什么要排除 Shenandoah 呢？可能只是因为他们使用了标准的构建脚本来构建他们的开源二进制文件。然而，在一个相当轻功能的 OpenJDK 版本中，我发现开源构建排除了一个最重要的贡献是很奇怪的。我非常感谢 Oracle 提供 GPL 许可的 OpenJDK 构建，但我希望他们能构建所有的构建。

然而，并非一切都没了。如果你想在 Red Hat Enterprise Linux 或 Fedora 上尝试 Shenandoah，从 JDK 8 开始，我们的所有版本都支持它；只需使用软件安装工具，或者，例如，

```
 $ yum install java-11-openjdk-devel
```

如果你想尝试 JDK 12，Fedora 和 Red Hat Enterprise Linux(通过 [EPEL](https://fedoraproject.org/wiki/EPEL) )的软件包将很快推出。对于其他系统， [AdoptOpenJDK](https://adoptopenjdk.net/) 提供了 JDK 12 个二进制，都支持 Shenandoah。

*Last updated: May 1, 2019*