# GCC 的推荐编译器和链接器标志

> 原文：<https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc>

你知道当你编译你的 C 或 C++程序时， [GCC](https://gcc.gnu.org/) 不会默认启用所有的异常吗？你知道为了获得与 GNU/Linux 发行版相同的安全强化级别，你需要指定哪些构建标志吗(比如 [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/) 和 [Fedora](https://www.redhat.com/en/technologies/linux-platforms/articles/relationship-between-fedora-and-rhel) )？本文介绍了一个推荐的构建标志列表。

Red Hat Enterprise Linux 和 Fedora 中基于 GNU 的工具链(由 GCC 程序如`gcc`、`g++`和 [Binutils](https://www.gnu.org/software/binutils/) 程序如`as`和`ld`组成)在构建标志方面非常接近上游默认值。由于历史原因，GCC 和 Binutils 上游项目在默认情况下不支持优化或任何安全强化。虽然在从源代码构建 GCC 和 Binutils 时，默认设置的某些方面可以更改，但我们在 RPM 构建中提供的工具链不会这样做。我们只将体系结构选择与发行版要求的最低体系结构级别保持一致。

因此，开发人员需要注意构建标志，并根据他们的项目需求来管理它们，以实现优化、警告和错误检测级别以及安全强化。

在创建发行版(如 Fedora 和 Red Hat Enterprise Linux)的构建过程中，必须注入编译器和链接器标志，如下所述。当您将这些发行版中的一个与附带的编译器一起使用时，会重新创建这个环境，需要指定一个广泛的标志列表。由于工具链和内核的限制，推荐的标志因发行版本而异。下表列出了推荐的构建标志(如`gcc`和`g++`编译器驱动程序所示)，以及适用于哪个版本的 Red Hat Enterprise Linux 和 Fedora 的简要说明:

*关于红帽企业版 Linux 的 GCC 版本的说明—GCC 4.8 包含在红帽企业版 Linux 7 中。GCC 4.4 包含在 Red Hat Enterprise Linux 6 中。您可以使用红帽开发工具集(DTS) 轻松安装 GCC 8 或更高版本。2018 年 11 月发布的[DTS 8](https://developers.redhat.com/blog/2018/11/13/gcc-8-2-ga-rhel/)包含了 GCC 8.2。 [RHEL 8 Beta](https://developers.redhat.com/rhel8) 使用 GCC 8 作为默认编译器。*

| **【标志】t1㎡** | **目的** | **适用的 Red Hat Enterprise Linux 版本** | **适用的 Fedora 版本** |
| --- | --- | --- | --- |
| `-D_FORTIFY_SOURCE=2` | 运行时缓冲区溢出检测 | 全部 | 全部 |
| `-D_GLIBCXX_ASSERTIONS` | C++字符串和容器的运行时边界检查 | 全部(但在没有 DTS 6 或更高版本的情况下无效) | 全部 |
| `-fasynchronous-unwind-tables` | 增加回溯的可靠性 | 所有(适用于 aarch64、i386、s390、s390x 和 x86_64) | 全部(适用于 aarch64、i386、s390x、x86_64) |
| `-fexceptions` | 启用基于表的线程取消 | 全部 | 全部 |
| `-fpie -Wl,-pie` | 可执行文件的完全 ASLR | 7 和更高版本(对于可执行文件) | 全部(对于可执行文件) |
| `-fpic -shared` | 共享库没有文本重定位 | 全部(对于共享库) | 全部(对于共享库) |
| `-fplugin=annobin` | 生成用于强化质量控制的数据 | 将来的 | Fedora 28 和更高版本 |
| `-fstack-clash-protection` | 提高堆栈溢出检测的可靠性 | 未来(7.5 之后) | 27 及以后(除 armhfp 外) |
| `-fstack-protector`或`-fstack-protector-all` | 堆垛粉碎保护器 | 仅 6 个 | 不适用的 |
| `-fstack-protector-strong` | 同样地 | 7 及更高版本 | 全部 |
| `-g` | 生成调试信息 | 全部 | 全部 |
| `-grecord-gcc-switches` | 在调试信息中存储编译器标志 | 全部 | 全部 |
| `-mcet -fcf-protection` | 控制流完整性保护 | 将来的 | 28 及更高版本(仅限 x86) |
| `-O2` | 推荐的优化 | 全部 | 全部 |
| `-pipe` | 避免临时文件，加速构建 | 全部 | 全部 |
| `-Wall` | 推荐的编译器警告 | 全部 | 全部 |
| `-Werror=format-security` | 拒绝潜在不安全的格式字符串参数 | 全部 | 全部 |
| `-Werror=implicit-function-declaration` | 拒绝缺失的函数原型 | 全部(仅限 C 语言) | 全部(仅限 C 语言) |
| `-Wl,-z,defs` | 检测并拒绝下划线 | 全部 | 全部 |
| `-Wl,-z,now` | 禁用惰性绑定 | 7 及更高版本 | 全部 |
| `-Wl,-z,relro` | 重定位后的只读段 | 6 及更高版本 | 全部 |

该表没有列出用于管理可执行堆栈或`.bss`部分的标志，假设这些历史特性现在已经被淘汰。

**编译器标志**的文档可以在 [GCC 手册](https://gcc.gnu.org/onlinedocs/gcc/Invoking-GCC.html)中找到。那些标志(以`-Wl`开始)被传递给链接器，并在 ld 的[文档中描述。](https://sourceware.org/binutils/docs/ld/Options.html)

对于某些标志，需要进行额外的解释:

*   `-D_GLIBCXX_ASSERTIONS`启用额外的 **C++标准库加固**。它是用 libstdc++实现的，在 [libstdc++文档](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_macros.html)中有描述。与具有完全调试支持的 C++容器不同，它的使用不会导致 ABI 的变化。
*   许多**调试和性能工具**需要`-fasynchronous-unwind-tables`才能在大多数架构上工作(由于堆栈管理的架构差异，armhfp、ppc、ppc64、ppc64le 不需要这些表)。尽管它在 aarch64 上是必需的，但默认情况下上游 GCC 并不启用它。默认情况下，Red Hat Enterprise Linux 和 Fedora 的编译器带有一个补丁来启用它。
*   对于多线程 C 和 C++代码的**硬化，推荐使用`-fexceptions`。**如果没有它，线程取消处理程序的实现(由`pthread_cleanup_push`引入)在堆栈上使用一个完全无保护的函数指针。这个函数指针可以简化对基于堆栈的缓冲区溢出的利用，即使所讨论的线程从未被取消。
*   `-fstack-clash-protection`防止基于**重叠堆和栈**的攻击。这是 GCC 8 中新增的一个编译器标志，在 Red Hat Enterprise Linux 7.5 和 Fedora 26(以及两者的更高版本)中已经被反向移植到系统编译器中。我们期望这个编译器特性在 Red Hat Enterprise Linux 7.6 中达到成熟。这个标志的 GCC 实现有两种风格:通用的和特定于架构的。通用版本与旧的`-fstack-check`标志(不推荐使用)有许多相同的问题。对于 Red Hat Enterprise Linux 支持的体系结构，可以使用改进的特定于体系结构的版本。这包括 aarch64，上游 GCC 仅提供有问题的通用支持(截至 2018 年 2 月中旬)。Fedora armhfp 架构也缺乏上游和下游支持，因此不能在那里使用标志。
*   `-fstack-protector-strong`完全取代早期的**电池组保护器**选项。它只检测具有可寻址局部变量或使用`alloca`的函数。其他函数不会受到直接堆栈缓冲区溢出的影响，也不会被检测。这大大降低了堆栈保护程序对性能和代码大小的影响。
*   要启用主程序(可执行程序)的**地址空间布局随机化(ASLR)** ，必须使用`-fpie -Wl,-pie`。然而，尽管以这种方式生成的代码是位置独立的，但它使用了一些不能在共享库中使用的重定位(动态共享对象)。为此，使用`-fpic`，并与`-shared`链接(以避免在支持位置相关共享库的架构上的文本重定位)。动态共享对象总是独立于位置的，因此支持 ASLR。此外，Red Hat Enterprise Linux 6 中的内核在某些情况下对 PIE 二进制文件使用了不合适的地址空间布局( [bug 1410097](https://bugzilla.redhat.com/show_bug.cgi?id=1410097) )，这可能会严重干扰调试(以及其他事情)。这就是为什么不建议在 Red Hat Enterprise Linux 6 上构建 PIE 二进制文件。
*   `-fplugin=annobin`启用 [annobin 编译器插件](https://developers.redhat.com/blog/2018/02/20/annobin-storing-information-binaries/)，它捕获额外的元数据以允许确定在构建期间使用了哪些编译器标志。Annobin 目前只在 Fedora 上可用，它作为 Fedora 28 构建标志的一部分自动[启用，在那里显示为`-specs=/usr/lib/rpm/redhat/redhat-annobin-cc1`。](https://fedoraproject.org/wiki/Changes/Annobin)
*   为了生成我们推荐的**调试信息**(使用`-g`)，甚至是优化的生产构建。只有部分可用的调试信息(由于优化)肯定比什么都没有要好。使用 GCC，生成调试信息不会改变代码生成。在分发二进制文件之前，可以使用像`eu-strip`这样的工具来分离调试信息(这在 RPM 构建过程中会自动发生)。
*   `-grecord-gcc-switches`捕获编译器标志，这有助于确定在整个编译过程中是否使用了预期的编译器标志。
*   `-mcet -fcf-protection`支持未来英特尔 CPU 中的**控制流执行技术(CET)** 特性。这涉及到额外的 nop 的产生，这些 nop 被当前的 CPU 所忽略。建议您现在启用此标志，以检测由它们引起的任何问题(例如，与动态工具框架的交互，或性能问题)。
*   对于许多应用程序来说，`-O2`是一个很好的选择，因为`-O3`引入的额外内联和循环展开会增加指令缓存的占用，最终会降低性能。`-D_FORTIFY_SOURCE=2`还要求`-O2`或更高。
*   默认情况下，GCC 允许代码调用**未声明的函数**，将它们视为返回`int`。`-Werror=implicit-function-declaration`把这样的调用变成错误。这避免了难以追踪的运行时错误，因为默认的`int`返回类型与许多平台上的`bool`或指针不兼容。对于 C++，不需要这个选项，因为 C++编译器拒绝调用未声明的函数。
*   需要`-Wl,-z,defs`来检测**下划线**，这是在调用链接编辑器生成另一个共享库时，由于共享库参数缺失而导致的现象。这会产生一个带有不完整 ELF 依赖信息的共享库(以缺少`DT_NEEDED`标签的形式)，并且产生的共享对象可能与使用符号版本控制的库的未来版本(例如 glibc)不向前兼容，因为其中缺少符号版本控制信息。
*   不建议在 Red Hat Enterprise Linux 6 上使用`-Wl,-z,now`(也称为`BIND_NOW`)，因为动态链接器以错误的顺序处理非惰性重定位([错误 1398716](https://bugzilla.redhat.com/show_bug.cgi?id=1398716) )，导致 IFUNC 解析器失败。IFUNC 解析器的交互仍然是一个开放的问题，即使是在以后的版本中，但是`-Wl,-z,defs`将会发现涉及下划线的有问题的情况。

在 RPM 构建中，这些标志中的一些是使用`-specs=/usr/lib/rpm/redhat/redhat-hardened-cc1`和`-specs=/usr/lib/rpm/redhat/redhat-hardened-ld`注入的，因为 GCC 规范中的选项选择机制允许自动删除 PIC 构建(用于动态链接)中与 PIE 相关的标志(用于静态链接)。由于历史原因，`-Wl,-z,now`包含在`-specs=/usr/lib/rpm/redhat/redhat-hardened-ld`中，不在命令行中，所以它不会直接出现在构建日志中。

### 在 RPM 构建期间注入标志

RPM 规范文件需要在`%build`部分注入构建标志，作为构建工具调用的一部分。

最新版本的`redhat-rpm-config`包[记录了如何获得发行版编译器和链接器标志](https://src.fedoraproject.org/rpms/redhat-rpm-config/blob/master/f/buildflags.md)。注意，链接指向 Fedora 包的最新版本。对于较旧的发行版，仅支持以下获取标志的方法:

*   运行`./configure`的`%{configure}` RPM 宏，也设置`CFLAGS`和`LDFLAGS`宏。
*   `%{optflags}` RPM 宏和`$RPM_OPT_FLAGS`环境变量，为 C 和 C++编译器提供编译器标志。
*   `$RPM_LD_FLAGS`环境变量，提供链接器标志。

请注意，Red Hat Enterprise Linux 7 和更早版本并不支持所有软件包的完全硬化版本，因此有必要指定:

```
%global _hardened_build 1

```

```
in the RPM spec file to enable the full set of hardening flags. The optional hardening comprises ASLR for executables (PIE) and non-lazy binding/BIND_NOW. For technical reasons, the recommended linker flag -Wl,-z,defs is not used either.
```

### 要考虑的其他标志

*   `-fwrapv`告诉编译器，应用程序假设**有符号整数溢出**具有通常的取模行为(例如，就像在 Java 中一样)。默认情况下，整数溢出被视为未定义，这有助于某些循环优化。这可能会导致遗留代码出现问题，即使对于 C/C++，遗留代码也会采用 Java 行为。
*   `-fno-strict-aliasing`指示编译器对如何使用指针以及哪些**指针可以指向相同的数据(别名)**做出更少的假设。这可能是编译遗留代码所必需的。
*   `-flto`和其他各种标志可用于开启**链接时间优化(LTO)** 。这可以提高性能并减少代码，但可能会干扰调试。它还可能揭示源代码中的一致性问题，这些问题以前被单独的编译所隐藏。
*   在某些情况下，`-Os`(针对小代码进行优化)可能会比`-O2`产生更快的代码，这是因为减少了指令缓存压力。
*   对于某些应用，`-O3`或`-O2 -free-loop-vectorize`将提供显著的速度提升。默认情况下，GCC 不执行**循环向量化**。注意`-O3`会改变代码对 ELF 符号插入的反应方式，所以这个选项不完全兼容 ABI。总的来说，我们仍然认为`-O2`是默认的选择。
*   为了与遗留应用程序(尤其是 i686 上的应用程序)兼容，可能需要使用`-mstackrealign`，遗留应用程序在调用用最新 GCC 版本编译的库函数之前不保留**堆栈对齐**。

### 有问题的标志

有些标志使用相当频繁，但会引起问题。以下是其中几个例子:

*   会产生非常令人惊讶的后果，因为通常适用于浮点运算的许多恒等式不再适用。这种影响可以扩展到没有使用该选项编译的代码。
*   `-mpreferred-stack-boundary`和`-mincoming-stack-boundary`改变了 ABI，可能会破坏与其他代码和未来库升级的互操作性。
*   `-O0`可以改善调试体验，虽然它禁用了所有优化，但它也消除了任何依赖于优化的硬化(如 source fortification/`-D_FORTIFY_SOURCE=2`)。
*   同样，**杀毒选项** ( `-fsanitize=address`等等)可能是很好的调试工具，但是当它们用于跨多个操作系统版本长期使用的生产版本时，可能会产生无法预料的后果。例如，地址杀毒拦截器禁用了 ABI 与未来库版本的兼容性。

### Red Hat 开发人员工具集(DTS)的标志

[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/)为红帽企业版 Linux 提供最新稳定的 GNU 工具链。截至 2018 年 11 月，DTS 包含 GCC 8.2。参见本文[通过`yum`在红帽企业版 Linux 上安装 GCC 8](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/) 。

DTS 2.0 和更高版本中提供了`-fstack-protector-strong`标志。DTS 6 及以后版本支持`-D_GLIBCXX_ASSERTIONS`。DTS 7.1 支持`-fstack-clash-protection`标志。其他特定于版本的限制是由于 DTS 没有增强的系统组件(如 glibc 或内核)造成的，因此这些限制也适用于 DTS 版本。

### 语言标准版本

到目前为止，讨论的标志主要影响代码生成和调试信息。C 和 C++语言特有的一个重要问题是选择:

*   Red Hat Enterprise Linux 7 和更早版本中的系统编译器默认为 C 的 C90 和 C++的 C++98，带有许多 GNU 扩展，其中一些扩展成为了后来的标准版本。
*   红帽企业 Linux 7 系统编译器基于 GCC 4.8，支持 C 的`-std=gnu11`选项和 C++的`-std=gnu++11`选项。然而，C11 和 C++11 支持都是实验性的。
*   开发人员工具集(DTS)为较新版本的标准提供了广泛的支持，使用混合链接模型确保了与系统 libstdc++库的兼容性。
*   Fedora 27 系统编译器默认为 C11 和 C++14(在未来的 Fedora 版本中会再次改变)。

一般情况下，建议使用工具链支持的最新标准版本，对于 Red Hat Enterprise Linux 系统编译器是 C99 ( `-std=gnu99`)和 C++98(默认启用)。对于开发人员工具集，应该使用更新的默认值。标准中的一些变化不具有完美的向后兼容性。因此，可能需要移植工作来使用新标准的设置。

请注意，即使是最新版本的 GNU 工具链也不支持一些可选的 C 特性(如 C11 线程或附录 K 及其`_s`函数)，C++支持也在不断发展，尤其是对于最近或即将推出的 C++标准版本。

* * *

**参见**:

*   [如何在红帽企业版 Linux](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/) 上安装 GCC 8——使用`yum`安装 GCC 8 以及 Clang/LLVM 6。【2018 年 11 月开发者工具集 8 发布包括 GCC 8.2、GDB 8.2 以及 SystemTap、Valgrind、OProfile 和其他工具的更新版本
*   阅读自 Red Hat Enterprise Linux 6 和 7 中发布的 4.x 版本以来 GNU 编译器集合的改进。
    *   [GCC 8 中的可用性改进](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)
    *   [-GCC 7 中的 wimplict-fall through](https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/)
    *   [GCC 6 即将推出的特性](https://developers.redhat.com/blog/2016/02/23/upcoming-features-in-gcc-6/)
    *   [关于 GCC 5-开发者工具集你需要知道的 5 件事](https://developers.redhat.com/blog/2015/10/16/5-things-need-know-gcc-5-developer-toolset-beta/)

*Last updated: March 7, 2019*