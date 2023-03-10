# Red Hat 开发工具集 8.1 测试版现已推出

> 原文：<https://developers.redhat.com/blog/2019/04/17/red-hat-developer-toolset-8-1-beta-now-available>

[Red Hat Developer Toolset](https://developers.redhat.com/videos/youtube/_CHQVnkk70E/) 用最新的、稳定的 GCC 版本来扩充 Red Hat Enterprise Linux，这些版本与原始的基础版本一起安装。此版本的 Red Hat 开发工具集 8.1 Beta 包括以下新组件:

*   GCC 8.2.1
*   GDB 8.2
*   binutils 2.30
*   埃尔夫蒂斯 0.176
*   Valgrind 3.14.0

对于 AMD64 和 Intel 64 架构，Red Hat Enterprise Linux 6 和 Red Hat Enterprise Linux 7 支持此测试版。它还支持 Red Hat Enterprise Linux 7 上的以下架构:64 位 ARM、IBM POWER()的 little-endian 和 little-endian 变体以及 IBM Z。有关每个更新组件的更多信息，请参见下文。

## Red Hat 开发工具集 8.1 Beta 组件详细信息

GCC 8.2.1 对 DTS 8 中包含的版本提供了许多错误修复:`yum install devtoolset-81`

GDB 8.2 在 DTS 8 版本的基础上提供了许多增强功能。它现在可以访问 IBM POWER Systems 体系结构的 POWER8 处理器的这些附加寄存器:

`PPR`、`DSCR`、`TAR`、`EBB/PMU`寄存器和`HTM`寄存器。

binutils 2.30 对 DTS 8 中包含的版本进行了大量修改。以前，使用 GOLD 链接器是为 IBM POWER Systems 体系结构试验性启用的。由于链接器功能不全，它在 Red Hat Developer Toolset 8.1 Beta 中已被禁用。

elfutils 0.176 在 DTS 8 版本的基础上提供了几个增强功能，包括:

*   与多个 cv 相关的各种错误已被修复。
*   用`dwelf_elf_begin()`函数扩展了`libdw`库，这是处理压缩文件的`elf_begin()`的变体。
*   `eu-readelf`工具现在可以识别并打印带有`--notes`或`-n`选项的 GNU 属性注释和 GNU 构建属性 ELF 注释。
*   一个新的`--reloc-debug-sections-only`选项已经被添加到`eu-strip`工具中，以解决调试部分之间的所有琐碎的重新定位，而不需要任何其他剥离。在某些情况下，此功能仅与`ET_REL`文件相关。

Valgrind 3.14.0 在 DTS 8 版本的基础上提供了许多增强功能，包括:

*   增加了对 IBM Z 体系结构的 z13 指令集的支持。
*   对 IBM POWER Systems 体系结构的向量指令的支持得到了改进。

## 资源

有关此版本的更多详细信息，请查看用户文档:

*   [DTS 8.1 测试版用户指南](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/8-beta/html-single/user_guide)
*   [DTS 8.1 测试版发行说明](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/8-beta/html-single/8.1_release_notes/index)
*   [DTS 8.1 测试版组件的完整列表](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/8/html-single/user_guide/index#tabl-Red_Hat_Developer_Toolset-About)

*Last updated: May 1, 2019*