# GNU 工具链更新-2018 年春季

> 原文：<https://developers.redhat.com/blog/2018/03/26/gnu-toolchain-update-2018>

GNU 工具链是由 GNU 项目开发的编程工具的集合。这些工具通常打包在一起，因为它们通常用于开发软件应用程序、操作系统和嵌入式系统的低级软件。

这篇博客是一个系列(见:【2017 年秋季更新)的一部分，涵盖了组成这个工具链的组件的最新变化和改进。除了新版本的发布，这里描述的特性是工具中软件开发的前沿。这意味着它们进入生产版本可能还需要一段时间，它们可能还没有完全发挥作用。但是任何对试验感兴趣的人都可以构建自己的工具链副本，然后进行试验。

## GLIBC

2.27 版本现在出来了；这个版本中有许多新特性和错误修复。一些亮点包括:

*   该库现在可以编译，支持构建静态 PIE 可执行文件。
*   abort 函数立即终止进程，不刷新 stdio 流。以前的 glibc 版本用于刷新流，导致死锁和进一步的数据损坏。
*   增加了对内存保护键的支持。 *< sys/mman.h >* 头现在声明了函数 *pkey_alloc，pkey_free，pkey_mprotect，pkey_set，pkey_get* 。
*   添加了 *copy_file_range* 功能。
*   增加了对运行在 Linux 上的 RISC-V ISA 的支持。

详情见:
[https://sourceware.org/ml/libc-announce/2018/msg00000.html](https://sourceware.org/ml/libc-announce/2018/msg00000.html)

## BINUTILS

2.30 版本现在出来了；此版本中有几项新功能，包括:

*   汇编程序支持 DWARF 调试行信息中的位置视图。
*   bfd 链接器现在有一个 *-z 分离代码*命令行选项来生成一个单独的代码 PT_LOAD 段。
*   bfd 链接器现在也有一个*-z defs*命令行选项，作为 *-z defs* 选项的逆选项。
*   黄金链接器现在有一个 *-z text-unlikely-segment* 选项，将所有. text.unlikely 部分移动到一个单独的段。
*   DWARF 转储工具(readelf，objdump)现在支持显示和跟踪链接到单独的调试信息文件的选项。

详情见:
[https://sourceware.org/ml/binutils/2018-01/msg00381.html](https://sourceware.org/ml/binutils/2018-01/msg00381.html)

## 基因组数据库

8.1 版本现已发布，包括以下新功能:

*   默认情况下，C++函数上的断点现在设置在所有作用域上(“通配符”匹配)。
*   改进的防锈支持；特别是，现在可以在调试 Rust 代码时检查 Trait 对象。
*   *启用*和*禁用*命令现在接受一系列断点位置。
*   新的 *rbreak* 命令通过正则表达式模式插入一些断点。

详情见:
[https://www.gnu.org/software/gdb/news/](https://www.gnu.org/software/gdb/news/)

## 纽利布

newlib C 库的 3.0.0 版本现在出来了；该版本包括:

*   代码和文档中删除了 K&R 支持。
*   64 位 time_t 支持。
*   增加了 RISC-V 平台支持。
*   新的 expf、exp2f、logf 和 powf 实现。
*   新的长双复数数学例程。

## （同 groundcontrolcenter）地面控制中心

与此同时，GCC 正迅速接近第 8 版的分支日期。尽管如此，在过去的几个月里还是增加了几个新功能:

*   控制流保护
    新选项 *-fcf-protection=[full]* 启用或禁用分支和返回指令的检测，以确保它们的目的地有效。目前，x86 GNU/Linux target 提供了基于英特尔控制流执行技术(CET)的实现。x86 的指令插入由特定于目标的选项控制:
    *-mcet、-mibt* 和 *-mshstk。*
*   堆栈冲突保护
    新选项 *-fstack-clash-protection* 生成代码来防止堆栈冲突风格的攻击。启用此选项时，编译器一次只会分配一页堆栈空间，并且在分配后会立即访问每一页。
*   可修补函数入口点
    新选项*-fpatchable-Function-Entry = N[，M]* 在每个函数的开始处生成 *N* nop 指令，函数入口点在第*M’*个 nop 之前。nop 指令保留了额外的空间，只要代码段是可写的，这些空间就可以用来在运行时插入任何所需的指令。空间的大小可以通过 nop 的数量间接控制。
*   支持新的 C 和 C++标准
    *-STD =*命令行选项现在接受 *c17、c++17* 和 *c++2a* 作为标准名称。这些标准对应于 ISO C17 标准、2017 年 ISO C++标准以及暂定于 2020 年发布的 ISO C++标准的下一个修订版。
*   新的警告选项
    *   *-Wmissing-attributes:*
        当函数声明缺少一个或多个相关函数声明所用的属性时发出警告，这些属性的缺少可能会对生成代码的正确性或效率产生不利影响。使用-Wall 自动启用。
    *   -Wstringop-truncation:
        警告对有界字符串操作函数的调用，如 *strncat、strncpy* 和 *stpncpy* ，这些函数可能会截断复制的字符串或保持目标不变。
    *   *-Wcast-align=strict:*
        每当指针被抛出使得目标所需的对齐增加时发出警告。例如，如果一个*字符** 被转换为一个*整数** ，则发出警告。
    *   *-Wcast-function-type:*
        当函数指针被强制转换为不兼容的函数指针时发出警告。在涉及具有可变参数列表的函数类型的强制转换中，只考虑所提供的初始参数的类型。
    *   *-Wpacked-not-aligned:*
        如果在打包结构或联合中具有显式指定对齐方式的结构字段未对齐，则发出警告。
*   改进的调试位置支持
    添加了几个新的命令行选项，以便能够生成更好的调试信息:
    *   *-gas-loc-support:*
        通知编译器汇编器支持*。loc* 指令，它可以使用它们来生成 DWARF 行号表。这通常是可取的，因为汇编程序生成的行号表比编译器自己生成的要紧凑得多。
    *   *-gas-loc view-support:*
        通知编译器，汇编程序支持视图赋值和复位断言检入。loc 指令。这使得汇编程序生成的行号表更好。
    *   *-gvariable-location-views:*
        将用行号表中隐含的渐进视图号增加可变位置列表。这使得调试信息使用者能够检查程序某些点的状态，即使在该点没有与相应的源位置相关联的指令。
    *   *-ginline-points:*
        会让编译器为内联函数生成扩展调试信息。位置视图跟踪标记被插入到内联的入口点，以便可以计算地址和视图编号，并在调试信息中输出。
*   新的清理选项
    *   *-fsanitize = pointer-compare:*
        与指针操作数一起使用时，启用比较操作的指令插入。
    *   *-fsanitize = pointer-subtract:*
        使用指针操作数启用减法运算的指令插入。
    *   -*fsanitize =指针溢出:*
        使用指针操作数启用加法运算的指令插入。
    *   *-fsanitize=builtin:*
        启用所选内置函数的参数检测。
    *   -fsanitize-coverage = trace-CMP:
        支持数据流引导的模糊代码检测。

你可能还想看看由[大卫·马尔科姆撰写的 GCC 8](https://developers.redhat.com/blog/author/rhdmalcolm/) 中的[可用性改进。](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)

目前就这些。我们会在夏天回来看更多！

*Last updated: March 30, 2018*