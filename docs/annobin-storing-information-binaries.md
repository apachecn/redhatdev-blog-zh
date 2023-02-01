# 在二进制文件中存储额外的信息

> 原文：<https://developers.redhat.com/blog/2018/02/20/annobin-storing-information-binaries>

## 介绍

编译文件，通常称为二进制文件，是现代计算机系统的支柱。但是对于系统构建者和用户来说，除了这些文件的基本信息之外，通常很难找到更多的信息。Annobin 项目旨在回答以下问题:

*   这个二进制是如何构建的？
*   对二进制文件执行了什么测试？
*   二进制的来源是什么？

Annobin 项目是[水印规范](https://fedoraproject.org/wiki/Toolchain/Watermark)的一个实现，它详细说明了如何在二进制文件中记录额外的信息。该规范的一个重要特征是，它包括存储信息的地址范围。这使得有可能记录这样的事实，即二进制文件的一部分是用一组选项编译的，而另一部分是用另一组选项记录的。

## 它是如何工作的

信息以一系列 ELF 注释的形式存储在二进制文件中，并保存在一个特殊的部分。选择 ELF 注释是因为它们是定义良好的结构，任何操作 ELF 文件的工具都可以识别，并且当调试信息被删除时，它们不会从文件中被剥离。包含注释的部分也被标记为不可加载，因此它不会占用程序运行时映像中的任何空间。

水印规范是这样设计的，当二进制文件链接在一起时，注释可以被连接起来，并且它们将保持有效。该规范还包括一组合并注释的规则，如果这对用户来说是一个问题的话，可以减少注释的大小。

笔记可以由任何东西生成，尽管在 Annobin 项目中，它们是由 GNU 编译器集合(GCC)的插件创建的。该插件在启动时通过扫描 GCC 命令行和编译状态来记录大部分笔记。但是它也将自己插入到编译过程中，这样它就可以监视各个函数编译方式的变化，如果相关的话，它也可以记录这些变化。

为了从编译后的二进制文件中提取注释，使用了 readelf 程序。这将信息解码并以人类可读的形式显示出来。

## 如何使用它

要启用 Annobin 插件，请使用 GCC 命令行选项: *-fplugin=annobin*

如果 GCC 找不到插件，那么可能也需要添加-iplugindir 选项:*-iplugindir =<path/to/dir/containing/anno bin>*

注意:对于 Fedora 包维护者——如果使用标准的 rpm 构建宏，Annobin 插件会自动启用。

这应该是开始在二进制文件中记录信息所需的全部内容。为了查看插件是否工作，可以使用 readelf 程序来检查注释:

*readelf - notes - wide <文件>*

大多数二进制文件已经包含其他类型的笔记，因此为了找到由 Annobin 创建的笔记，请查找“Owner”字段以字母“GA”开头的笔记:

*所有者数据大小描述*
*GA$ <版本>3p 4 0x 000000010 OPEN 适用于 0x7da 到 0x838*
*GA$ <工具>gcc 7 . 2 . 1 2017 09 15 0x 00000000 OPEN 适用于 0x7da 到 0x838*

旧版本的 readelf 很难理解这些注释，因此输出可能如下所示:

*车主数据大小描述*
*GA $ 3p 4 0x 000000010 未知注类型:(0x 00000100)*
*GA $ gcc 7 . 2 . 1 2017 09 15 0x 00000000 未知注类型:(0x 0000100)*

Annobin 项目包括一些示例脚本，演示如何使用这些注释来执行各种检查。脚本记录在 Annobin 的 info 文件中，并且在脚本本身内部。下面是一个快速概述:

### built-by.sh

尝试确定哪个工具编译了二进制文件。如果可能的话，使用笔记，但也尝试几种其他方法。

### check-abi.sh

检查二进制文件，看它是否是用具有不同 ABI(因此可能不兼容)的目标文件构建的。

### hardened.sh

检查二进制文件，看它是否是用预期的一组强化选项构建的。

这些只是例子。其他脚本可以编写，其他笔记可以记录在二进制文件中。

## 如何构建它

可以从这里获得压缩的压缩文件:*https://nickc.fedorapeople.org/annobin-X.X.tar.xz*

其中 X.X 是最新的版本号(当前为 3.4)。

或者，最新的源代码可以在 Annobin git 库 git://[sourceware.org/git/annobin.git](http://sourceware.org/git/annobin.git)中找到。Annobin 也作为一个预构建的 rpm 存在于 Fedora 发行版中(从 Fedora 27 开始),可以用命令安装: *dnf install annobin*

源代码分为几个子目录:

*   插件 GCC 插件的源代码。
*   脚本-示例脚本。
*   测试——插件和脚本的测试套件。
*   文档-文档。
*   配置源文件所必需的配置文件。

只有插件实际需要搭建，通常的“*配置；make* 序列应该足够了。尽管这个插件有几个依赖项，但是唯一特别的是它需要由支持插件并提供它们需要的头文件的 GCC 版本来构建。

## 如何延伸

水印规范被设计成可扩展的。任何人或任何工具都可以添加任意注释。它们可以在创建二进制文件时添加，也可以在以后添加。最简单的方法是用要添加的注释创建一个汇编文件，然后将它汇编成一个目标文件。然后，该文件可以包含在二进制文件的最后一个链接中，或者通过使用 objcopy 程序添加到二进制文件中(使用其 *- merge-notes* 选项)。

汇编程序源文件中的一个注释可能如下所示:

*。section . GNU . build . attributes*
DC . l。名字结束。Lname_start #名称字段长度
*.dc.l 0 #描述字段长度*
*. DC . l 0x 100 # type = OPEN*
*。lname _ start:*T14*。asciz " GA $<your-text-here>" # name field*
*。Lname_end:*

或者，如果便笺需要涵盖特定的地址范围:

*。section . GNU . build . attributes*
DC . l。名字结束。Lname_start #名称字段长度
*.dc.l 16 #描述字段长度*
*. DC . l 0x 100 # type = OPEN*
*。lname _ start:*T14*。asciz " GA $<your-text-here>" # name field*
*。lname _ end:*T20*。quad start_symbol #描述字段*
*。四边形结束符号*

## 未来的步骤

Annobin 项目仍在开发中。未来计划包括:

*   增加了汇编程序插入自己的注释的能力。这将允许为非 GCC 编译的文件记录注释(例如，汇编源文件或用 LLVM 编译的文件)。
*   增加了记录源代码散列的能力。在编译期间，每个输入文件(头文件和源代码)都被散列(使用 SHA-256？)及其名称和哈希值存储在编译后的二进制文件中。然后，消费者可以使用存储的哈希值来验证他们拥有的源代码是否与用于编译二进制文件的源代码相同。

### 链接:

*   水印规范-[https://fedoraproject.org/wiki/Toolchain/Watermark](https://fedoraproject.org/wiki/Toolchain/Watermark)
*   Annobin git 库:git://[sourceware.org/git/annobin.git](http://sourceware.org/git/annobin.git)

*Last updated: February 27, 2018*