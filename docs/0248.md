# Debuginfo 不仅仅用于调试程序

> 原文：<https://developers.redhat.com/blog/2021/01/07/debuginfo-is-not-just-for-debugging-programs>

在 Red Hat 的很长一段时间里，rpm 中的所有可执行文件都是在启用 debuginfo 的情况下构建的。虽然这种做法使支持人员能够更容易地调查使用 GDB 和崩溃等工具报告的问题，但最终的 debuginfo 还有其他重要的非调试用途。

Debuginfo 因其最初的用途而得名。随着时间的推移，使用调试所需的相同信息的其他应用程序(如应用程序二进制接口[ABI]符合性检查、数据结构布局分析和性能监控)已经开发出来。最好将 debuginfo 视为编译器生成的可执行文件和开发人员编写的源代码之间的映射信息。它帮助人们获得关键信息，以更好地理解实际运行他们系统的可执行代码，并提供一种双重检查编译器和开发人员工作的方法。

## 应用程序二进制接口(ABI)检查

目前，在 Linux 上，很少有应用程序是单片独立的二进制文件。大多数二进制文件大量使用共享库来利用其他项目的工作，并减少可执行文件的大小。然而，为了使这种实践可行，由库提供的应用程序二进制接口(ABI)需要稳定。如果一个 Linux 发行版更新了一个应用程序使用的共享库，结果这个应用程序突然停止工作，那就太可怕了。像 [libabigail](https://sourceware.org/libabigail/) 这样的工具比较共享库的不同版本，以检测 ABI 的任何变化。

这些 ABI 检查工具使用 debuginfo 来确定哪些函数可用、函数的返回类型、传递给函数的参数以及这些不同类型的数据布局。debuginfo 是一种机器可读的机制，用于验证 ABI 没有改变。没有 debuginfo，这些检查是不可行的。

对于像 [Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux) 和 [Fedora](https://getfedora.org/) 这样的发行版，这些 [ABI 检查](https://developers.redhat.com/blog/2017/02/28/abi-change-analysis-of-fedora-packages/)是必不可少的。ABI 检查是将软件包的较新的候选版本与以前的版本进行比较，以确保 ABI 兼容性。这确保了依赖这些软件包的其他软件将继续运行。

## 数据结构布局分析

编译器可以在数据结构中插入填充，以确保数据结构不违反目标计算机体系结构的对齐约束。这种填充浪费内存。当开发人员创建数据结构时，他们可能不知道对齐约束。此外，对于某些编译器选项和其他目标处理器，可能会使用不同的对齐约束。

dwarves 包中的 pahole 等工具读取描述数据结构的 debuginfo，并确定在何处添加了填充。即使在拥有千兆字节 RAM 的机器上，人们也非常希望[更有效地组织数据结构，以避免浪费空间](https://developers.redhat.com/blog/2016/06/01/how-to-avoid-wasting-megabytes-of-memory-a-few-bytes-at-a-time/)。

## 性能监控

人们可能希望对程序在生产中的操作提供反馈。这些信息可以提供程序如何正常运行的基线。它还可以洞察程序的哪些区域消耗了太多时间，可以提高效率。通过 [perf](https://fedoramagazine.org/performance-profiling-perf/) 对程序计数器进行周期性采样是一种估计程序各个区域所用时间的常用方法。然而，仅仅拥有来自程序堆栈的程序地址和字节的原始样本对开发人员来说并没有什么用处。开发人员希望堆栈回溯，这样他们就可以生成[火焰图](http://www.brendangregg.com/flamegraphs.html)，并能够确定他们需要修改源代码的哪些区域，以提高软件未来版本的性能。

## 将 debuginfo 信息放在手边

debuginfo 的这些非调试用途可以捕获 ABI 中的意外变化、数据结构中意外浪费的空间以及代码中意外低效的部分，从而帮助提高代码质量。debuginfo 确实会占用空间，但是有一些方法可以将这些信息从二进制文件中剥离出来，放在其他地方，这样默认的二进制文件就不会比不使用 debuginfo 编译的二进制文件大。因此，最好默认启用 debuginfo 来编译代码。

*Last updated: January 6, 2021*