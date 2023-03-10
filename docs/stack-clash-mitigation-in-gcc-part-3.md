# GCC 中的堆栈冲突缓解，第 3 部分

> 原文：<https://developers.redhat.com/blog/2020/05/22/stack-clash-mitigation-in-gcc-part-3>

在之前的帖子中，[*GCC 中的栈冲突缓解—背景*](https://developers.redhat.com/blog/2017/09/25/stack-clash-mitigation-gcc-background/) 和[*GCC 中的栈冲突缓解:Why -fstack-check 不是答案*](https://developers.redhat.com/blog/2019/04/30/stack-clash-mitigation-in-gcc-why-fstack-check-is-not-the-answer/) 中，我满怀希望地展示了[栈冲突攻击](https://www.qualys.com/2017/06/19/stack-clash/stack-clash.txt)是如何构造的，以及为什么 GCC 现有的`-fstack-check`机制不足以保护。

那么，我们该怎么办呢？很明显，我们想要类似于`-fstack-check`的东西，但是没有基本的问题。输入一个新选项:`-fstack-clash-protection`。

防止堆栈冲突攻击的代码生成的主要原则是:

*   任何单个分配都不能大于一页。编译器必须将大型请求转换成一系列页面大小或更小的请求。
*   在分配页面时，发出指令来探测它们。(姑且称这些*显式探针*。)
*   一系列没有插入探针的子页分配总共不能分配超过一页。

围绕这些原则的简单实现可能效率很低，但是这个选项为构建安全、高性能的实现提供了基础。

## 提高性能的隐式探测

代码中自然发生的堆栈访问是一个*隐式探测*。隐式探测意味着没有额外的成本；因此，使用隐式探测比使用显式探测更有利。例如，由于目标体系结构的行为、应用程序二进制接口(ABI)的要求，或者通过分析程序中现有的内存引用，可能会发生隐式探测。

例如，许多处理器上的 call 指令将返回地址推送到堆栈上。因此，如果堆栈在堆栈保护中，call 指令将出错。这是在*sp 的隐式探测。一些应用程序二进制接口要求*sp 总是包含一个反向链指针(指向下一个外部堆栈框架的指针)。因此，每次堆栈分配都需要自动更新*sp。同样，这是对*sp 的隐式探测。

我们还可以分析生成的代码。例如，在一个目标上，调用者为被调用者分配空间来保存寄存器。因此，在被调用者中，寄存器 save to *(sp + 48)是在*(sp + 48)的隐式探测。在其他目标上，被调用者经常在函数入口将寄存器对推到堆栈上。这些推送是*sp 的隐式探测。

事实证明，利用上面提到的隐式探测可以显著减少显式探测的数量。如果我们将 glibc 作为 x86 和 PPC 上的一个例子，我们会发现 glibc 中只有不到 2%的函数需要在它们的序言中进行显式探测。例如，如果一个函数在这些体系结构上分配的堆栈空间少于一页，那么就不需要显式探测。

## 当前状态

从 RHEL 7.5 开始，Red Hat 的工程师为所有[Red Hat Enterprise Linux](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)(RHEL)目标实现了`-fstack-clash-protection`。RHEL 7.5 只为 glibc 启用了`-fstack-clash-protection`。从 [RHEL 8](https://developers.redhat.com/rhel8/) 开始，整个发行版用`-fstack-clash-protection`编译，用`annobin` / `annocheck`来验证发行版用正确的标志编译。

Fedora 27 和更高版本默认为所有使用标准默认编译选项的包启用`-fstack-clash-protection`(注意没有对 32 位 ARM 目标的`-fstack-clash-protection`支持)。

GCC 8 包括对英特尔、IBM Power、IBM Z 系列和 ARM 的 aarch64 目标的`-fstack-clash-protection`支持。

LLVM 11 将包括 Serge Guelton 编写的针对 Intel 64 和 AMD64 的堆栈冲突保护。

## 测试

Red Hat 的工程师编写了各种测试来验证静态和动态堆栈利用率的分析。自从引入 GCC 以来，Red Hat 的工程师还为所有针对`-fstack-clash-protection`报告的错误编写了回归测试。这些测试作为 GCC 标准回归测试过程的一部分运行。如果要在当前不支持的目标上实现堆栈冲突缓解，大多数测试都具有足够的可移植性，可以在其他目标上使用。

Red Hat 的工程师还实现了一个扫描器，可以检查可重定位对象、可执行文件和动态共享对象。扫描器在一个指令窗口中查找违反上述关键原则的情况，并将可疑代码通知开发人员。Red Hat 已经使用扫描仪扫描关键库和对象(对扫描仪报告为潜在易受攻击的所有序列进行人工验证)。事实证明，这种做法在验证 Fedora 27 一直在使用`-fstack-clash-protection`以及验证 ARM 工程师对 aarch64 实现的改进无效时特别有用。

*Last updated: May 19, 2020*