# 跳转线程优化简介

> 原文：<https://developers.redhat.com/blog/2019/03/13/intro-jump-threading-optimizations>

作为 GCC 10 的 [GCC 开发者](https://developers.redhat.com/blog/2019/03/08/usability-improvements-in-gcc-9/) ' [按需范围工作](https://gcc.gnu.org/wiki/AndrewMacLeod/Ranger)的一部分，我一直在改进后向跳转线程，使它能够线程化依赖于范围的路径。这反过来让我关注跳转线程，这是我多年来一直小心翼翼避免使用的编译器的一部分。如果你和我一样，对编译器优化很好奇，但是不知道跳转线程，也许你会对这个简短的介绍感兴趣。

在最高级别，跳转线程的主要目标是减少程序控制流图中不同路径上动态执行的跳转次数。由于条件的减少，这通常会导致性能的提高，从而实现进一步的优化。通常，对于通过跳转线程消除的每个运行时分支，会消除两到三个其他运行时指令。

控制流的简化也大大降低了`-Wuninitialized`等警告的误报率。来自`-Wuninitialized`的误报通常会发生，因为存在通过控制流图的路径，这些路径在运行时不会发生，但会保留在代码的内部表示中。

GCC 开发人员发现`-Wuninitialized`的误报和错过的优化机会之间有很强的相关性。因此，GCC 开发人员对 `-Wuninitialized`的任何误报都非常感兴趣。

经典的跳转线程示例是一个简单的跳转到跳转优化。例如，它可以转换以下内容:

```
  if (a > 5)
    goto j;
  stuff ();
  stuff ();
j:
  goto somewhere;

```

到下面更优化的序列中:

```
  if (a > 5)
    goto somewhere;
  stuff ();
  stuff ();
j:
  goto somewhere;

```

但是，跳转线程也可以线程化两个已知重叠的部分条件:

```
void foo(int a, int b, int c)
{
  if (a && b)
    foo ();
  if (b || c)
    bar ();
}
```

以上转化为:

```
void foo(int a, int b, int c)
{
  if (a && b) {
    foo ();
    goto skip;
  }
  if (b || c) {
skip:
    bar ();
  }
}

```

一个更有趣的序列是跳转线程复制块以避免分支。考虑上面的一个稍微调整的版本:

```
void foo(int a, int b, int c)
{
  if (a && b)
    foo ();
  tweak ();
  if (b || c)
  bar ();
}

```

编译器很难线程化上面的代码，除非它复制了`tweak()`，使得结果代码更大:

```
void foo(int a, int b, int c)
{
  if (a && b) {
    foo ();
    tweak ();
    goto skip;
  }
  tweak ();
  if (b || c) {
skip:
    bar ();
  }
}

```

由于代码复制，编译器能够在不改变语义的情况下连接两个重叠的条件。顺便说一下，这是跳转线程的最终目标:避免昂贵的条件分支，即使这可能以更多代码为代价。

为了追求更快的运行速度，GCC 对于它愿意复制的指令或基本块的数量是有限制的。各种编译调整迫使跳转线程考虑更长的序列。其中一个选项是`--param max-fsm-paths-insns=500`，它使线程化器线程化序列，每个序列可能复制多达 500 条指令(而不是默认的 100 条)。还有`--param max-fsm-thread-length`，它类似地扩展了 threader 的最大值，但是使用了基本块长度而不是指令长度。和所有的`--param`选项一样，把它们用于自娱自乐和巧妙的派对把戏，因为它们可能会在没有通知的情况下发生变化。

默认情况下，`-O2`及以上版本启用跳转线程，但不幸的是，它与各种值范围传播(VRP)通道交织在一起，没有独立的方法将其关闭。欺骗性的`-fno-thread-jumps`标志只在低级 RTL 优化器中关闭跳转线程，在典型的编译中只处理极少量的跳转线程。让`-fno-thread-jumps`适用于整个编译器中的所有跳转线程，以及从整体上将 VRP 从跳转线程中分离出来，都在我们的任务列表中。

如果你想看看跳转线程的运行，用`-fdump-tree-all-details -O2`编译一个足够复杂的程序，看看`*.c*{ethread, thread1, thread2, thread3, thread4}`和 VRP 转储`(*.c*{vrp1, vrp2})`。你应该会看到类似`Threaded jump 3 --> 4 to 7`的东西。

尽情享受吧！

## 也阅读

*   [了解 GCC 警告](https://developers.redhat.com/blog/2019/03/13/understanding-gcc-warnings/)

## 面向 C/C++开发人员的更多文章

*   GCC 9 的可用性改进(GCC 9 计划在 Fedora 30 中推出)
*   GCC 8 中的可用性改进(GCC 8 现在可用于 Red Hat Enterprise Linux 6、7 和 8 测试版。)
*   [如何在红帽企业版 Linux 7 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/#more-568577)
*   [GCC 的推荐编译器和链接器标志](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)
*   [Clang/LLVM 入门](https://developers.redhat.com/blog/2017/11/01/getting-started-llvm-toolset/)
*   [用 GCC 8 检测字符串截断](https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8/)
*   [通过 GCC 7 进行的隐式下降检测](https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/)
*   [使用 GCC 7 进行内存错误检测](https://developers.redhat.com/blog/2017/02/22/memory-error-detection-using-gcc/)
*   [用 GCC 插件诊断函数指针安全缺陷](https://developers.redhat.com/blog/2017/03/17/diagnosing-function-pointer-security-flaws-with-a-gcc-plugin/)
*   [更好地使用 C11 原子——第一部分](https://developers.redhat.com/blog/2016/01/14/toward-a-better-use-of-c11-atomics-part-1/)

*Last updated: March 11, 2019*