# 为 x86-64-v2 微体系结构级别构建 Red Hat Enterprise Linux 9

> 原文：<https://developers.redhat.com/blog/2021/01/05/building-red-hat-enterprise-linux-9-for-the-x86-64-v2-microarchitecture-level>

构建 Linux 发行版时，最重要的早期决策之一是支持硬件的范围。发行版的默认编译器标志对于硬件平台兼容性非常重要。使用较新 CPU 指令的程序可能无法在较旧的 CPU 上运行。在本文中，我将讨论一种构建 x86-64 版本的 Red Hat Enterprise Linux(RHEL)9 的新方法，并分享 Red Hat 对此版本的建议。

## x86-64 微体系结构级别的背景

GNU C 库(glibc)提供了一种加载优化库的方法，这些优化库使用了并非所有系统都具备的附加硬件特性。最初，除了通常安装在`/usr/lib64`目录中的缺省(后备)实现之外，这种机制被设计为支持一两个可选的库实现。然而，库查找机制[中涉及的 power-set 构造与当前具有一长串可选 CPU 特性](https://sourceware.org/pipermail/libc-alpha/2020-May/113757.html)的平台不太匹配。我们尤其在 x86 架构上看到这一点，多年来已经添加了许多可选特性(参见维基百科文章中关于 CPUID 指令的以获得列表)。过多的选择不仅给动态链接器也给程序员带来了问题。直到最近，关于在优化的库中采用什么样的 CPU 特性还没有什么指导。 [GCC 和 glibc 在特性集的定义上有分歧](https://sourceware.org/bugzilla/show_bug.cgi?id=24080)，而 [glibc 选择机制是特定于厂商的](https://sourceware.org/bugzilla/show_bug.cgi?id=23249)。

2020 年夏天，AMD、英特尔、Red Hat 和 SUSE [合作](https://lists.llvm.org/pipermail/llvm-dev/2020-July/143289.html)在 x86-64 基准之上定义了三个 x86-64 微架构级别。这三种微体系结构大致根据硬件发布日期将 CPU 特性组合在一起:

*   **x86-64-v2** 带来了对矢量指令的支持(尤其是),最高支持 SIMD 流扩展 4.2 (SSE4.2)和补充 SIMD 流扩展 3 (SSSE3)、POPCNT 指令(用于数据分析和某些数据结构中的位调整)以及 CMPXCHG16B(用于并发算法的双字比较和交换指令)。
*   **x86-64-v3** 增加了 AVX2、MOVBE(用于大端数据访问)和附加位操作指令的向量指令。
*   **x86-64-v4** 包括一些 AVX-512 变体的矢量指令。

我们已经在 x86-64 psABI 附录中详细记录了这三个级别[。即将发布的 GCC 版本 11 和 LLVM 版本 12 将在`-march=`参数中支持它们。用一种新的机制(没有动力装置结构)来增强`glibc`动态装载机的补丁已经被整合到`glibc`中，命名为`glibc-hwcaps`。这些变化预计将成为即将到来的【2.33 版本的一部分。](https://gitlab.com/x86-psABIs/x86-64-ABI)

## RHEL 9 的建筑考虑

从历史上看，x86_64 Red Hat Enterprise Linux 用户空间的构建符合最初的 AMD K8 基线减去 AMD 特定的 3Dnow！零件。该决定支持并包括最新版本的 Red Hat Enterprise Linux 8。然而，由于内核驱动程序的删除，旧的硬件(比如带有第一代 Opteron CPUs 的系统)不太可能以任何有用的方式运行 Red Hat Enterprise Linux。当运行较旧的硬件时，也有显著的功率需求。

到目前为止，我们已经能够通过像 [IFUNCs](https://sourceware.org/glibc/wiki/GNU_IFUNC) 、[功能多版本](https://gcc.gnu.org/onlinedocs/gcc/Function-Multiversioning.html)这样的机制来利用新的 CPU 特性，或者通过`dlopen`加载替代实现，这可以通过正在进行的`glibc-hwcaps`工作来自动化。这些方法中的每一种都只适用于特别指定的代码块。发行版的其余部分仍然没有采用额外的 CPU 特性，所以 CPU 的这些部分基本上是休眠的。

作为定义 x86-64 微体系结构级别的一个受欢迎的副作用，我们现在有了一种方便的语言来讨论 Linux 发行版的体系结构基线:我们可以坚持到底，使用最初的 K8 基线，或者我们可以应用后面三个级别中的一个。

## 关于 RHEL 的建议 9

我们认为 x86-64-v2 是 Red Hat Enterprise Linux 9 的合适选择。我们的建议基于以下观察:

*   尽管有主机支持，但虚拟机模型会人为屏蔽 x86-64-v2 CPU 特性，这相对容易调整。
*   下一个级别 x86-64-v3 不可用，因为我们打算为 x86-64 架构构建一个统一的发行版。
*   2020 年发布的[新服务器级 CPU 不实现 AVX 指令集。](https://www.intel.com/content/www/us/en/products/processors/atom/p-series/atom-p-5900-processor-brief.html)
*   AVX 指令在某些软件实现中也不可用(尽管`valgrind`工具支持它们)。缺少仿真支持可能会限制开发人员开发 Red Hat Enterprise Linux 9。

和以前的 Red Hat Enterprise Linux 版本一样，我们将通过 IFUNCs 和函数多版本化继续支持其他 CPU 特性(超越 x86-64-v2)。我们也可以使用`glibc-hwcaps`机制来加载库的优化版本。像往常一样，在 Red Hat Enterprise Linux 9 发布之前，这些计划可能会随时发生变化。

目前，这些变化并不适用于除了 Fedora ELN 以外的 Fedora。

## 对其他架构的支持

Red Hat 与其硬件合作伙伴一起定期审查所有架构的架构基线。对于 IBM POWER 和 IBM Z，我们已经为每个主要版本更新了基线。例如，Red Hat Enterprise Linux 8 需要 POWER little-endian (ppc64le)和——对于 s390x 架构——z13 或更高版本的 CPU。

## 结论

本文描述了指导 Red Hat 为 Red Hat Enterprise Linux 9 选择 x86-64 微体系结构级别的标准。我们的建议 x86-64-v2 将支持额外的矢量指令(高达 SSE4.2 和 SSSE 3)、用于数据分析的 POPCNT 指令以及用于并发算法的 CMPXCHG16B 指令。

*Last updated: January 29, 2021*