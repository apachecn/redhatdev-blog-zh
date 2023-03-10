# NOP 空间中的指令编码指南

> 原文：<https://developers.redhat.com/blog/2019/07/29/guidelines-for-instruction-encoding-in-the-nop-space>

越来越多的 CPU 使用在前几代 CPU 上作为 nop(无操作指令)执行的指令来实现新功能。这给操作系统构建者带来了一些挑战，尤其是在遗留软件支持领域。

一个可以被某些 CPU 完全忽略的指令看起来并没有什么用处，但事实证明它有相当多的应用，包括:

*   性能提示，例如标记作为互斥锁和解锁操作一部分的原子加载和存储。
*   可选的数组边界检查，由 MPX 英特尔公司实施。
*   安全加固，比如在加载时验证内存是只读的(不可写的)，使得注入 C++ vtables 或字节码更加困难。
*   指令流中的标记，用于控制流完整性验证(如英特尔 CET)，或作为动态检测工具(如 Valgrind)的提示。

在 NOP 空间中使用指令编码意味着操作系统和应用程序开发人员可以交付一组二进制文件。这些二进制文件可以在旧的和新的 CPU 上运行。在旧的 CPU 上，由于它们的非操作性质，它们提供的指令和信息被完全忽略。能够识别指令的新 CPU 可以运行相同的二进制文件，这样就可以从额外的 CPU 特性中获益。有一些相关的注意事项，以便在实践中顺利工作。

## 不同实现之间的 NOP 支持可能有所不同

大多数指令集有多个独立的实现。即使只有一家 CPU 供应商提供真正的芯片，像 Fedora 和 Red Hat Enterprise Linux 这样的发行版至少要处理三种实现:实际的硬件、QEMU 中的仿真支持以及 [Valgrind](http://valgrind.org/) 指令解码器和编译器。根据现有的实现之一，如果新指令应该在 NOP 空间中，但是不在，结果可能是不可预测的。通常，这些尚未实现的 NOP 会导致非法的指令陷阱，这使得在 NOP 空间中使用指令的理由完全无效。

例如，英特尔 x86 架构最初只支持有限长度的 NOP 指令。对更长 NOP 的支持最早出现在英特尔奔腾 Pro CPU 中，但是[长 NOP 支持并不存在于所有具有类似特性集](https://sourceware.org/bugzilla/show_bug.cgi?id=6957)的 CPU 中，这导致了互操作性问题。使用长 nop 的二进制文件因 CPU 上不支持它们的非法指令陷阱而崩溃。

## 改变 NOP 行为需要是可选的

作为额外的安全措施，NOP 空间中的指令不应该在仅仅升级 CPU 之后立即变为活动的。这有两个原因:一个工具链(编译器/汇编器/链接器组合)可能偶然产生了这些 nop(例如，CPU 供应商没有有效跟踪的非主流工具链)。或者，可能存在故意使用 NOP 指令的现有二进制文件。因为 nop 到目前为止一直被忽略，不忽略它们可能会暴露新的错误(不正确的结果或崩溃)。这些错误可能是真正的工具链错误，也可能是应用程序开发人员对新功能的简单误用，他们没有机会测试他们的代码。

对于想要升级硬件的用户来说，这两种情况都非常具有挑战性。软件通常比硬件寿命长。从源代码重建受影响的二进制文件(或者根据更新的库重新链接它们)并不总是可能的。在这种情况下，NOP 空间中的新指令会无限期地延迟硬件更新。

在过去，虚拟化足以解决这一难题:虚拟机管理程序可以提供一个禁用这些 NOP 空间指令的 CPU 模型，即使物理 CPU 支持它们。当然，这仍然需要 CPU 对分割配置的某种程度的支持:对 hypervisor 本身和一些来宾的新指令支持，但对其他来宾没有支持(真正的 NOP 操作)。(通常，系统管理员不希望为了单个来宾而禁用虚拟机管理程序上的安全强化功能。)

在基于容器的世界中，这种粗略的控制似乎是不够的。在具有高安全性需求(或多个租户)的环境中，容器主机和某些特权容器将被期望以 NOP-space 指令提供的完全强化来运行。然而，仍然希望提供与最终用户提供的容器映像的完全兼容性，这些容器映像受到上述任一或两种错误类别的影响。引导时配置设置通常过于粗糙而没有用。(我们最近在 Linux 上的`vsyscall`页面支持的上下文中遇到了类似的安全性强化和与旧容器映像的兼容性之间的权衡。)

### 如何让新行为变得可选

实现这种选择性支持的低级软件接口可以采取各种形式。

1.  使用 ELF 标记来启用新的 CPU 功能。内核中的 ELF 加载器(或 glibc 动态加载器)可以检查现有的二进制文件，并验证所有文件都具有所需的支持级别。如果用户空间中的动态加载器负责打开支持，那么与下一个选项有重叠。
2.  兼容的二进制文件可以明确选择新的行为，使用系统调用(或用于`prctl`或`arch_prctl`系统调用的新的子命令)，或通过设置特殊的 CPU 寄存器(这就是`libmpx`如何打开英特尔 MPX)。此设置仅适用于当前进程或线程。(这里什么有意义取决于所讨论的特性。)当使用`execve`用新的进程映像替换进程映像时，对新指令的支持将恢复到默认状态(禁用)。
3.  对于容器用例，有一个系统调用并使支持状态可以跨`execve`继承可能会很有趣。这意味着容器中的整个流程树将使用新的 CPU 特性。可能通过检查与容器映像相关联的元数据并将该信息通过系统调用传递给内核，由容器引擎来确保映像与其兼容。

涉及内核、管理程序、CPU 以及固件的具体实现可以采取多种形式。关键的一点是，一个运行的操作系统内核必须能够跨其独立的进程支持不同的功能激活状态。一旦 CPU 和固件支持这种能力，内核和虚拟机管理程序就可以协作实现每个进程的模型。如果它只是一个引导时选项，无论是每个客户机还是(更糟糕的)每个物理主机，在基于容器的环境中启用该特性要困难得多。

## 与动态仪器工具(如 Valgrind)协调

Valgrind 不仅模拟 NOP，还可以使用 [`valgrind.h`](https://sourceware.org/git/gitweb.cgi?p=valgrind.git;a=blob;f=include/valgrind.h;h=HEAD) 头文件嵌入特殊的类似 NOP 的指令序列。当在真实的 CPU 上运行时，这些指令(大部分)被忽略，但是当在仿真下运行时，它们为 Valgrind 提供了有用的信息。例如，当 Valgrind 假定内存是已定义的(基于之前的程序动作)时，它们可以显式地将内存标记为未定义。

新的 NOP 空间指令不应该与 Valgrind 使用的标记指令冲突。

## 结论

NOP 空间中的指令是提供新的性能和安全特性的一种有吸引力的方式。然而，需要注意避免与 NOP 指令的现有使用冲突。而且，如果出现这种冲突，最终用户将需要一种方法来解决它们，这意味着他们使用的操作系统将需要能够在单个进程级别启用和禁用对新 NOP 空间指令的支持。

*Last updated: July 26, 2019*