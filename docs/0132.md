# GDB 开发者的 GNU 调试器教程，第 1 部分:调试器入门

> 原文：<https://developers.redhat.com/blog/2021/04/30/the-gdb-developers-gnu-debugger-tutorial-part-1-getting-started-with-the-debugger>

本文是演示如何有效地使用 [GNU 调试器(GDB)](https://www.gnu.org/software/gdb/) 来调试 [C 和 C++](/topics/c) 中的应用程序的系列文章的第一篇。如果您使用 GDB 的经验有限或者没有经验，本系列将教您如何更有效地调试代码。如果你已经是一个使用 GDB 的经验丰富的专业人士，也许你会发现一些你以前没有见过的东西。

除了为许多 GDB 命令提供开发人员提示和技巧，未来的文章还将涵盖调试优化代码、离线调试(核心文件)和基于服务器的会话(*又名* `gdbserver`，用于容器调试)等主题。

## 为什么是另一个 GDB 教程？

网上的大多数 GDB 教程只不过是对基本的`list`、`break`、`print`和`run`命令的介绍。新 GDB 用户不妨读(或唱)一下[官方的 GDB 歌曲](https://www.gnu.org/music/gdb-song.html)！

本系列中的每一篇文章都将从开发 GDB 的人的角度来关注使用 GDB 的一个方面，而不是简单地演示一些有用的命令。我每天都使用 GDB，这些技巧和窍门是我(和许多其他高级 GDB 用户和开发人员)用来简化我们的调试会话的。

因为这是该系列的第一篇文章，请允许我按照 GDB 歌曲的建议，从头开始:如何管理 GDB。

## 编译程序任选项

让我来解决一个显而易见的问题:为了获得最佳的调试体验，在没有优化和调试信息的情况下构建应用程序。这是一个微不足道的建议，但 freenode.net GDB 公共 IRC 频道(#gdb)经常看到这些问题，它们值得一提。

**TL；DR** :如果可以避免的话，不要用优化来调试应用。请关注未来关于优化的文章。

如果您不知道“幕后”可能会发生什么，优化会导致 GDB 以令人惊讶的方式运行在开发周期中，我总是使用 C 编译器选项`-O0`(即字母 *O* 后跟数字零)来构建可执行文件。

我也总是让工具链发出调试信息。这是通过`-g`选项完成的。指定确切的调试格式不再是必要的(或可取的)；DWARF 多年来一直是 GNU/Linux 上默认的调试信息格式。所以不要理会使用`-ggdb`或`-gdwarf-2`的建议。

值得添加的一个特定选项是`-g3`，它告诉编译器包含应用程序中使用的宏(`#define FOO ...`)的调试信息。这些宏可以像程序中的其他符号一样在 GDB 使用。

简而言之，为了获得最佳的调试体验，在编译代码时使用`-g3 -O0`。一些环境(比如那些使用 GNU 自动工具的环境)设置控制编译器输出的环境变量(`CFLAGS`和`CXXFLAGS`)。检查这些标志以确保编译器的调用启用了您想要的调试环境。

有关`-g`和`-O`对调试体验的影响的更多信息，请参见 Alexander Oliva 的论文 [GCC gOlogy:研究优化对调试的影响](https://www.fsfla.org/~lxoliva/#gOlogy)。

## 启动脚本

在我们实际使用 GDB 之前，必须说明一下 GDB 是如何启动的，它执行什么脚本文件。启动时，GDB 将执行许多系统和用户脚本文件中包含的命令。这些文件的位置和执行顺序如下:

1.  `/etc/gdbinit`(不在 FSF GNU GDB 上):在很多 GNU/Linux 发行版中，包括 Fedora 和[红帽企业 Linux](/products/rhel/overview) ，GDB 首先寻找系统默认初始化文件，并执行其中包含的命令。在基于 Red Hat 的系统上，这个文件执行安装在`/etc/gdbinit.d`中的任何脚本文件(包括 [Python](/blog/category/python/) 脚本)。
2.  `$HOME/.gdbinit` : GDB 将从主目录中读取用户的全局初始化脚本(如果该文件存在的话)。
3.  最后，GDB 将在当前目录中寻找一个启动脚本。请将此视为特定于应用程序的定制文件，您可以在其中添加每个项目的用户定义命令、漂亮的打印机和其他定制。

所有这些启动文件都包含要执行的 GDB 命令，但它们也可能包含 Python 脚本，只要它们以`python`命令开头，例如`python print('Hello from python!')`。

我的`.gdbinit`其实挺简单的。它最重要的几行启用了命令历史，这样 GDB 就可以记住上一次会话中执行的给定数量的命令。这类似于 shell 的历史机制和`.bash_history`。整个文件是:

```
set pagination off
set history save on
set history expansion on
```

第一行关闭了 GDB 的内置分页功能。下一行允许保存历史(默认情况下保存到`~/.gdb_history`)，最后一行允许使用感叹号(！)性格。这个选项通常是禁用的，因为感叹号在 c #中也是一个逻辑运算符。

为了防止 GDB 读取初始化文件，给它一个`--nx`命令行选项。

## 在 GDB 获得帮助

有几种方法可以获得使用 GDB 的帮助，包括广泛的——如果枯燥的话——[文档](https://sourceware.org/gdb/documentation/)解释每个小开关、旋钮和功能。

### GDB 社区资源

该社区在两个地方为用户提供帮助:

*   通过电子邮件: [GDB 邮件列表](https://sourceware.org/mailman/listinfo/gdb/)
*   通过 IRC: #gdb on [libera.chat](https://libera.chat/)

然而，因为这篇文章是关于*使用* GDB，用户获得命令帮助的最简单的方法是使用 GDB 的内置帮助系统，下面将讨论。

### 访问帮助系统

通过`help`和`apropos`命令访问 GDB 的内置帮助系统。不知道如何使用`printf`命令？问 GDB:

```
(gdb) help printf
Formatted printing, like the C "printf" function.
Usage: printf "format string", ARG1, ARG2, ARG3, ..., ARGN
This supports most C printf format specifications, like %s, %d, etc.
(gdb)
```

`help`接受任何 GDB 命令或选项的名称，并输出该命令或选项的用法信息。

像所有 GDB 命令一样，`help`命令支持制表符结束。这可能是确定许多命令接受什么类型的参数的最有用的方法。例如，输入`help show ar`并按 tab 键将提示您完成:

```
(gdb) help show ar
architecture   args         arm
(gdb) help show ar
```

GDB 让您在命令提示符下准备好接受进一步细化的输入。将`g`添加到命令中，后跟一个制表符，将完成到`help show args`:

```
(gdb) help show args
Show argument list to give program being debugged when it is started.
Follow this command with any number of args, to be passed to the program.
(gdb)
```

不知道您要查找的命令的确切名称？使用`apropos`命令在帮助系统中搜索特定术语。你可以把它看作是对内置帮助的一种模仿。

现在你知道了如何以及在哪里可以找到帮助，我们准备好开始 GDB 了。

## 从 GDB 开始

不出所料，GDB 接受大量命令行选项来改变其行为，但启动 GDB 最基本的方式是在命令行上将应用程序的名称传递给 GDB:

```
$ gdb myprogram
GNU gdb (GDB) Red Hat Enterprise Linux 9.2-2.el8
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /home/blog/myprogram...
(gdb) 

```

GDB 启动，打印出一些版本信息(显示了 GCC 工具集 10)，加载程序及其调试信息，并显示版权和帮助消息，以命令提示符`(gdb)`结束。GDB 现在准备接受意见。

### 避免消息:-q 或- quiet 选项

我已经看过 GDB 的启动信息几千次了，所以我用`-q`选项抑制(或“安静”)它:

```
$ gdb -q myprogram
Reading symbols from /home/blog/myprogram...
(gdb) 

```

那就更不用读了。如果你真的是 GDB 的新手，你可能会发现完整的启动消息非常有用或令人宽慰，但是过一会儿，你也会在你的 shell 中将`gdb`别名为`gdb -q`。如果您确实需要隐藏的信息，使用`-v`命令行选项或`show version`命令。

### 传递参数:- args 选项

程序通常需要命令行参数。GDB 提供了多种方式将这些传递给你的程序(或者用 GDB 的说法是“劣等的”)。两种最有用的方法是通过`run`命令或者在启动时通过`--args`命令行选项传递应用程序参数。如果您的应用程序通常以`myprogram 1 2 3 4`启动，只需在前面加上`gdb -q --args`，GDB 就会记住您的应用程序应该如何运行:

```
$ gdb -q --args myprogram 1 2 3 4
Reading symbols from /home/blog/myprogram...
(gdb) show args
Argument list to give program being debugged when it is started is "1 2 3 4".
(gdb) run
Starting program: /home/blog/myprogram 1 2 3 4
[Inferior 1 (process 1596525) exited normally]
$

```

### 附加到正在运行的进程:- pid 选项

如果一个应用程序已经在运行，却被“卡住”了，你可能想从内部找出原因。只要给 GDB 你的应用程序的进程 ID 和`--pid`:

```
$ sleep 100000 &
[1] 1591979
$ gdb -q --pid 1591979
Attaching to process 1591979
Reading symbols from /usr/bin/sleep...
Reading symbols from .gnu_debugdata for /usr/bin/sleep...
(No debugging symbols found in .gnu_debugdata for /usr/bin/sleep)
Reading symbols from /lib64/libc.so.6...
Reading symbols from /usr/lib/debug/usr/lib64/libc-2.31.so.debug...
Reading symbols from /lib64/ld-linux-x86-64.so.2...
Reading symbols from /usr/lib/debug/usr/lib64/ld-2.31.so.debug...
0x00007fc421d5ef98 in __GI___clock_nanosleep (requested_time=requested_time@entry=0, remaining=remaining@entry=0x0)
    at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:28
28	  return SYSCALL_CANCEL (nanosleep, requested_time, remaining)
(gdb) 

```

使用此选项，GDB 会自动加载具有内部版本 ID 信息的程序的符号，例如发行版提供的软件包，并中断程序，以便您可以与它进行交互。在以后的文章中可以找到更多关于 GDB 如何以及在哪里找到调试信息的信息。

### 跟进失败:核心选项

如果您的进程中止并转储了核心文件，使用`--core`选项告诉 GDB 加载核心文件。如果核心文件包含中止进程的构建 ID，GDB 会自动加载该二进制文件及其调试信息(如果可以的话)。然而，大多数开发人员需要通过以下选项将可执行文件传递给 GDB:

```
$ ./abort-me
Aborted (core dumped)
$ gdb -q abort-me --core core.2127239
Reading symbols from abort-me...
[New LWP 2127239]
Core was generated by `./abort-me'.
Program terminated with signal SIGABRT, Aborted.
#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:50
50	  return ret;
(gdb)

```

**提示**:找不到核心文件？在使用 systemd 的 GNU/Linux 系统上，检查`ulimit -c`以查看 shell 是否阻止程序创建核心文件。如果值是`unlimited`，使用`coredumpctl`查找核心文件。或者，运行`sysctl -w kernel.core_pattern=core`来配置 systemd 输出名为`core.` *`PID`* 的核心文件，正如我在前面的例子中所做的那样。

### 加速命令执行:- ex、- iex、- x 和- batch 选项

我经常在 shell 中反复运行 GDB 命令来测试问题或运行脚本。这些命令行选项有助于实现这一点。大多数用户会使用(多个)`--ex`参数来指定启动时运行的命令，以重新创建调试会话，例如`gdb -ex "break` *`some_function`* `if` *`arg1`* `== nullptr" -ex r` *`myprogram`*。

*   `--ex` *`CMD`* 加载程序(和调试信息)后，运行 GDB 命令 *`CMD`* 。`--iex`执行相同的操作，但在加载指定程序之前执行 *`CMD`。*
*   `-x` *`FILE`* 在程序加载和`--ex`命令执行后，执行来自 *`FILE`* 的 GDB 命令。如果我需要大量的`--ex`参数来重现一个特定的调试会话，我会经常使用这个选项。
*   使 GDB 在第一个命令提示符下立即退出；即在所有命令或脚本运行之后。注意`--batch`将比`-q`隐藏更多的输出，以便于在脚本中使用 GDB:

```
$ # All commands complete without error
$ gdb -batch -x hello.gdb myprogram
Reading symbols from myprogram...
hello
$ echo $?
0
$ # Command raises an exception
$ gdb -batch -ex "set foo bar"
No symbol "foo" in current context.
$ echo $?
1
$ # Demonstrate the order of script execution
$ gdb -x hello.gdb -iex 'echo before\n' -ex 'echo after\n' simple
GNU gdb (GDB) Red Hat Enterprise Linux 9.2-2.el8
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
before
Reading symbols from simple...
hello
after
(gdb) 

```

## 接下来

在本文中，我分享了关于 GDB 如何启动、读取脚本(以及何时读取脚本)的细节，以及高级 GDB 用户常用的几个启动选项。

本系列的下一篇文章将稍微绕一下，解释什么是调试信息，如何检查它，GDB 在哪里寻找它，以及如何在发行版提供的包中安装它。

你有关于 GDB 脚本或启动的建议或提示，或者关于如何使用 GDB 的未来主题的建议吗？请对这篇文章发表评论，并与我们分享您的想法。

*Last updated: October 14, 2022*