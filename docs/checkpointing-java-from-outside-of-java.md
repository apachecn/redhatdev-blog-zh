# 从 Java 外部对 Java 进行检查点操作

> 原文：<https://developers.redhat.com/blog/2020/10/15/checkpointing-java-from-outside-of-java>

当 [OpenJDK](https://developers.redhat.com/products/openjdk/overview) 的 Java 虚拟机(JVM)运行一个 [Java](https://developers.redhat.com/topics/enterprise-java) 应用程序时，它会在启动主类之前加载十几个类。它运行一个方法几百次，然后调用该方法上的优化编译器。这种准备是 Java“一次编写，随处运行”能力的重要组成部分，但这是以长启动时间为代价的。

我们一直在研究一种新的方法，它允许你加载你的类，预热你的实时(JIT)编译器，然后检查你的应用程序。稍后，您可以恢复应用程序，使其快速运行。有了这些改变，我们已经看到需要几秒钟启动的应用程序在几毫秒内就预热了。

在本文中，您将学习如何从 [Linux 命令行](https://developers.redhat.com/topics/linux)检查和恢复正在运行的 Java 程序。在不久的另一篇文章中，我将介绍一个 Java 本地接口(JNI)库，它允许您从 Java 代码内部检查点和恢复 Java 程序。

## 在 Java 代码中使用检查点

JNI [检查点恢复库](https://www.jfokus.se/jfokus19-preso/Checkpointing-Java.pdf)基于用户空间 (CRIU)中的 [Linux 检查点/恢复，我们将在本文的例子中使用它。CRIU 可以节省你的启动时间，但它提供了更多的可能性。](http://www.criu.org)

如果你有一个长时间运行的程序，你可以定期检查它。然后，如果出现故障，可以从最后一个检查点重新启动应用程序。如果失败是由于一个 bug，那么您可以快速重现它。如果失败是由外部因素引起的，您可以从停止的地方继续，而不会丢失任何工作。

作为另一个例子，假设您想在程序中的几个点进行堆转储，但是停止遍历堆会干扰执行。插入检查点使您可以运行程序直到完成，然后返回并重新开始进行堆转储。这样，您可以在感兴趣的时间点看到内存布局，但程序执行的顺序与原始程序非常相似。

听起来不错吧？我们来看一个例子。

## 从 Java 外部设置检查点

在这个例子中，您将学习如何从命令行检查和恢复一个正在运行的 Java 程序。首先，假设我们正在运行一个名为 Scooby 的 Java 程序。

从一号终端输入:

```
% setsid java -XX:-UsePerfData -XX:+UseSerialGC Scooby

```

在另一个终端的另一个目录中，输入:

```
% sudo criu dump -t <pid> --shell-job -o dump.log

```

您现在可以做一个`ps`并看到您的 Java 程序不再运行。您可以查看目录并查看大量图像文件。您还可以查看`dump.log`来了解 CRIU 为检查您的代码所做的一切。

现在，从您转储图像的目录中，执行以下操作:

```
% sudo criu restore --shell-job -d -vvv -o restore.log

```

您应该会看到您的 Java 程序再次运行。您可以检查`restore.log`来查看恢复做了什么。您会注意到，默认情况下，CRIU 将 JVM 恢复到相同的进程 ID (PID)。如果您想要多次恢复同一个映像，可以使用虚拟 PID:

```
% sudo unshare -p -m -f bash
# mount -t proc none /proc/
# criu restore --shell-job

```

在同一个目录的另一个窗口中，您可以:

```
% sudo unshare -p -m -f bash
# mount -t proc none /proc/
# criu restore  --shell-job

```

## 结论

检查点有一些问题。目前，在使用检查点时，您需要关闭`perf`和并行垃圾收集。如果您有一个`/var/lib/sss/pipes/nss`文件，您将不得不删除它。您还需要 root 权限来运行`restore`操作，因为您需要能够选择特定的 PID。CRIU 团队目前正在研究这个问题。

请继续关注我的下一篇文章，我将向您展示如何使用 JNI 检查点恢复库从 Java 内部检查 Java。

*Last updated: February 4, 2021*