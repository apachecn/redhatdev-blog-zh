# 2017 年夏季 GNU 工具链更新

> 原文：<https://developers.redhat.com/blog/2017/07/31/summer-2017-gnu-toolchain-update>

GNU 工具链是由 GNU 项目开发的编程工具的集合。这些工具通常打包在一起，因为它们通常用于开发软件应用程序、操作系统和嵌入式系统的低级软件。

这个博客是涵盖组成这个工具链的组件的最新变化和改进的常规系列的一部分。然而，除了发布新版本之外，这里描述的特性是工具中软件开发的前沿。这确实意味着它们进入生产版本可能还需要一段时间，它们可能还没有完全发挥作用。但是任何对试验感兴趣的人都可以构建自己的工具链副本，然后进行试验。

## Binutils

2.29 版本已经发布。

除了本博客中已经详细介绍的先前的更改之外，此版本还包含:

*   在使用虚拟内存管理器的系统上，支持将分区放入特殊的内存区域。这类似于链接器脚本中的 MEMORY 命令，只不过它只在没有内存管理单元的系统上工作。
*   在新的系统中，可以将部分标记为需要特殊类型的特殊存储器。链接器将所有具有相同需求的部分收集在一起，并将它们放入一个特殊标记的段中。然后，加载程序可以检测该段的需求，并确保使用了正确的内存类型。
*   支持 WebAssembly 文件格式并转换为 wasm32 ELF 格式。
*   PowerPC 汇编程序现在检查指令中使用了正确的寄存器类。
*   ARM 汇编器现在支持 ARMv8-R 架构和 Cortex-R52 处理器。
*   链接器现在支持 ELF GNU 程序属性。这些是为加载程序准备的运行时注释，告诉它关于正在初始化的二进制文件的更多信息。
*   链接器包含对英特尔间接分支跟踪(IBT)增强的支持。这是一项旨在帮助对抗恶意代码的技术，这些恶意代码滥用堆栈来强制程序执行不需要的行为。更多信息请见:
    *https://software . Intel . com/sites/default/files/managed/4d/2a/control-flow-enforcement-technology-preview . pdf*
*   现在，使用新的链接器选项- force-group-allocation 或通过将 FORCE_GROUP_ALLOCATION 放入链接器脚本中，可以在部分链接时解析节组(删除组并像普通节一样放置组成员)。
*   MIPS 端口现在支持:
    +micro MIPS 扩展物理寻址(XPA)指令。
    +ISA 第 5 版。
    +想象力 interAptiv MR2 处理器。
    +MIPS 16 e 2 便于组装和拆卸。
*   SPARC 端口现在支持 SPARC M8 处理器，该处理器实现了甲骨文 SPARC 架构 2017。
*   Objdump 的 *-行号*选项现在可以通过新的 *-内联*选项进行扩充，这样内联函数将显示它们的嵌套信息。
*   Objcopy 现在有一个选项“ *- merge-notes* ”，通过合并和删除冗余条目来减少二进制文件中的注释大小。
*   AVR 汇编程序支持 *__gcc_isr* 伪指令。当 GCC 想要创建一个中断处理程序的序言或尾声时，就会产生这个指令。然后，汇编程序确保生成可能的最佳代码。

同时在主线 binutils 来源:

*   汇编程序现在支持 DWARF 调试行信息中的位置视图。这是帮助改进编译器提供给调试器的源代码位置信息的项目的一部分:

https://developers.redhat.com/blog/2017/07/11/statement-frontier-notes-and-location-views/#more-437095

## 基因组数据库

8.0 版本已经发布。此版本包含:

*   支持 C++右值引用。
*   Python 脚本增强:
    +启动、停止和访问正在运行的 btrace 记录的新功能。
*   GDB 命令解释器:
    +用户命令现在可以接受无限数量的参数。
    +命令 *eval* 现在扩展了用户定义的参数。
*   DWARF 版本 5 支持
*   GDB/MI 增强:
    +新的*-文件-列表-共享-库*命令来列出程序中的共享库。
    +New*-target-flash-erase*命令，擦除闪存。
*   支持原生 FreeBSD/mips (mips*-*-freebsd)
*   支持 Synopsys ARC 和 FreeBSD/mips 目标。

关于每一项的完整列表和更多细节，请参见发布源中的 gdb/NEWS 文件。

同时，在开发源中添加了以下新功能:

*   在 Unix 系统上，GDBserver 现在在低级命令行参数中进行 globbing 扩展和变量替换。
*   新命令
    +*set debug separate-debug-file*
    +*show debug separate-debug-file*
    这些控制关于单独调试文件搜索的调试输出的显示。

## （同 groundcontrolcenter）地面控制中心

7.1 版本已经发布。这个版本中的大部分增强和新特性已经在这个博客的早期版本中报道过了，但是有几个新的东西在截止日期之前发布了:

*   添加了新的循环拆分优化过程。包含在迭代空间的一侧总是为真而在另一侧总是为假的条件的某些循环被分成两个循环，使得两个新循环中的每一个仅在迭代空间的一侧迭代，并且不需要在循环内部检查该条件。
*   PowerPC 端口对 ISA 3.0 的支持得到了增强，默认情况下可以生成更多的新指令，并提供更多的内置函数来为其他新指令生成代码。

同时在发展来源方面:

*   增加了对 SPARC M8 处理器的支持。
*   添加了开关 *'-mfix-ut700* 和 *-mfix-gr712rc* 选项，以解决 LEON3FT SPARC 处理器中的一个错误。
*   还增加了几个新的警告选项:
    +选项'*-wmultisationmacros*'警告不安全的多语句宏，这些宏看起来受到一个子句的保护，但实际上只有第一个语句受到保护。比如:
    *# define DOIT x++；y++*
    *if(c)DOIT；*
    +选项“ *-Wsizeof-pointer-div* ”警告当相关对象是指针而不是数组时，将指针大小除以元素大小的两个大小的表达式的可疑除法。比如:
    *char * ptr；*
    *int a = sizeof(ptr)/sizeof(ptr[0])；*

## GLIBC

2.26 版本的工作仍在继续。添加的新功能包括:

*   malloc 中添加了一个每线程缓存。对缓存的访问不需要锁，因此显著加快了分配和释放少量内存的快速路径。重新填充一个空的缓存需要锁定底层的竞技场，但是性能的提高仍然是显著的。
*   Unicode 10.0.0 支持:字符编码、字符类型信息和音译表都更新为 Unicode 10.0.0 标准。
*   对 DNS 存根解析器的改进:
    +GNU C 库现在将检测/etc/resolv.conf 何时被修改，并重新加载更改后的配置。GNU C 库现在支持任意数量的搜索域(使用/etc/resolv.conf 中的“search”指令配置)。
    +当“旋转”(RES_ROTATE)解析器选项激活时，GNU C 库现在会从配置中随机选择一个名称服务器作为起点。
*   默认情况下，可调参数功能现在是启用的。这允许用户使用 GLIBC 可调参数环境变量来调整 GNU C 库的行为。
*   新函数 reallocarray，它将分配的块(如 realloc)的大小调整为两个大小的乘积，保证在乘法中整数溢出时干净地失败。
*   特定于 Linux 的系统调用 preadv2 和 pwritev2 的新包装器。它们分别是 preadv 和 pwritev 的扩展版本，带有一个附加的 flags 参数。支持的标志集取决于运行的内核；全面支持目前需要内核 4.7 或更高版本。
*   posix_spawnattr_setflags 现在支持 POSIX_SPAWN_SETSID 标志，为衍生的进程创建一个新的会话 ID。
*   errno.h 现在可以在所有支持的操作系统上安全地使用 C 预处理汇编语言。在这种情况下，它将只定义 Exxxx 常量，作为扩展到整数文字的预处理宏。
*   在 ia64、powerpc64le、x86-32 和 x86-64 上，数学库现在实现了 ISO/IEC/IEEE 60559:2011(IEEE 754-2008)和 ISO/IEC TS 18661-3:2015 定义的 128 位浮点。

这次到此为止。更多在秋天...

* * *

**在 [RHEL 6 号或 7 号](https://developers.redhat.com/products/rhel/download/?intcmp=7016000000124eKAAQ)上用原生 GCC 构建你的第一个应用。**

*Last updated: January 3, 2023*