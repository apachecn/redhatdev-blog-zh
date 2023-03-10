# 分析并降低 SystemTap 的脚本启动成本

> 原文：<https://developers.redhat.com/blog/2018/10/30/analyzing-reducing-systemtaps-startup-cost>

SystemTap 是一个用于调查系统问题的强大工具，但是对于一些 SystemTap 检测脚本来说，启动时间太长了。本文是系列文章的第 1 部分，描述了如何分析和减少 SystemTap 的脚本启动成本。

我们可以使用 SystemTap 来研究这个问题，并提供一些关于 SystemTap 用来将 SystemTap 脚本转换为插装的每一遍所需时间的硬数据。SystemTap 有一组探测点，标记从 0 到 5 的路径的开始和结束:

*   pass0:解析命令行参数
*   密码 1:分析脚本
*   第二关:细化
*   第三步:翻译成 C 语言
*   pass4:将 C 代码编译成内核模块
*   步骤 5:运行检测

本系列文章:

*   第 1 部分:[分析并降低 SystemTap 的脚本启动成本](https://developers.redhat.com/blog/2018/10/30/analyzing-reducing-systemtaps-startup-cost/)
*   第 2 部分:[使用 syscall_any tapset](https://developers.redhat.com/blog/2018/11/08/systemtap-reduced-startup-syscalls/) 减少 SystemTap 监控脚本的启动开销

## 分析启动成本

因为我们对脚本实际收集数据所需的时间感兴趣，所以我们不打算在 pass5 上收集数据。一遍的开始和结束的每个探测点包括描述会话的 C++结构的参数。会话结构包括关于脚本名称的信息，这有助于识别哪些脚本具有较高的启动开销。开发 SystemTap 示例脚本 [stap_time.stp](https://sourceware.org/systemtap/examples/#apps/stap_time.stp) 就是为了记录这些信息。因为[红帽企业版 Linux](https://developers.redhat.com/products/rhel/download/) (RHEL) 7 存储 C++字符串的方式存在差异，所以该脚本被略微修改如下:

```
#! /usr/bin/env stap 
# Copyright (C) 2018 Red Hat, Inc.
# Written by William Cohen <wcohen@redhat.com>
#
# Provide concise plottable information about phases 0-4 of systemtap

global start, pass0, pass1, pass2, pass3, pass4

function print_head() {printf("")}
function print_tail() {printf("")}

probe begin { print_head() }
probe end { print_tail() }

probe stap.pass0 { start[session] = gettimeofday_ms() }
probe stap.pass0.end { pass0[session] = gettimeofday_ms() }
probe stap.pass1.end { pass1[session] = gettimeofday_ms() }
probe stap.pass2.end { pass2[session] = gettimeofday_ms() }
probe stap.pass3.end { pass3[session] = gettimeofday_ms() }

probe stap.pass4.end {
pass4[session] = gettimeofday_ms()
// Dig through C++ string private fields to find the systemtap script name
cmdline_script = @cast(session, "struct systemtap_session")->cmdline_script->_M_dataplus->_M_p
if (strlen(user_string2(cmdline_script, "<unavailable>"))){
  script = "<cmdline_script>"
} else {
  script_file = @cast(session, "struct systemtap_session")->script_file->_M_dataplus->_M_p
  script = user_string2(script_file, "<unavailable>")
}

// print out data
printf("%s %d %d %d %d %d\n",
script,
pass0[session] - start[session],
pass1[session] - pass0[session],
pass2[session] - pass1[session],
pass3[session] - pass2[session],
pass4[session] - pass3[session])
// delete entries
delete pass0[session]
delete pass1[session]
delete pass2[session]
delete pass3[session]
delete pass4[session]
}

```

当脚本运行时，每次 SystemTap 运行并完成 pass4 时，它都会打印出一行，显示脚本名称，后跟每次 pass (0 到 4)所需的时间(以毫秒为单位)，如下所示:

```
also_ran.stp 71 1721 1440 80 2245
```

因为 SystemTap 脚本是 SystemTap `testsuite`的一部分，所以通过在终端窗口中运行以下命令来收集所有示例的数据变得非常容易:

```
$ stap stap_time > syscall_any.dat
```

然后，在另一个终端窗口的`/usr/share/systemtap/testsuite`中，以 root 用户身份运行以下命令:

```
make installcheck RUNTESTFLAGS="--debug systemtap.examples/check.exp"
```

一旦`testsuite`完成，`syscall_any.dat`文件包含关于示例脚本的每个阶段花费多长时间完成的数据。Gnuplot 可以用下面的`syscall_any.gp`由这些数据组成:

```
set terminal png

set output 'syscall_any.png'
set key left top invert

set title 'Systemtap Examples Times with syscall\_any tapset'
set grid y
set style data histograms
set style histogram rowstacked
set ytics 10 nomirror
set style fill solid 1
set ylabel "Time (ms)"
set ytics 10000
set xlabel "Example script"
set xrange [0:]
set yrange [0:]

plot 'syscall_any.dat' using 2 t "Pass 0 (cmdline parsing)", '' using 3 t "Pass 1 (script parsing)", '' using 4 t "Pass 2 (elaboration)", '' using 5 t "Pass 3 (code generation)", '' using 6 t "Pass 4 (module compilation)",
```

现在我们有了一个条形图，总结了 SystemTap 将各种示例脚本转换成实际工具所用的时间。该图如下所示:

[![Bar graph showing how long SystemTap took to convert example script to instrumentation with syscall_any tapset](img/9c889947e607271eade9275df688964c.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/syscall_any.png)

首先引人注目的是峰值 243 ( `strace.stp`)正在逼近 90 秒的开销开始。`strace.stp`脚本检测每一个`syscall`的进入和退出，从而产生大量的探针和这些探针的相关代码。`errsnoop.stp`在 195–196 处的峰值也有同样的原因。

203 处的峰值是由于`ltrace.stp`在被监控的用户空间应用程序所使用的共享库中插入了许多函数。这为 545 个探测点创建了仪器。

图表右侧的峰值 300 和 301 是针对`qemu_count.stp`的，是通过使用 SystemTap 通配符来探测所有 QEMU 探测点而产生的。在进行测量的 x86_64 RHEL 机器上，由于对机器上安装的许多不同处理器(如 SPARC、MIPS 和 PowerPC 处理器)进行了 QEMU 仿真，因此有超过 10，000 个探测点被检测。

`hw_watch_addr.stp`和`hw_watch_sym.stp`的峰值为 122 到 125，而`ioctl_handler.stp`和`latencytap.stp`的峰值为 264 到 267，这并不常见，因为他们在代码生成(pass3)上花费的时间比大多数脚本都多。这是因为这些脚本是使用 SystemTap `--all-modules`选项生成的，该选项将更多的调试信息提取到检测中，以允许脚本将模块的地址映射回函数名。

## 降低启动成本

减少示例脚本的启动开销是一项正在进行的工作，但是在过去的一个月中，针对每个`syscall`的几个脚本已经有了一些改进。新发布的 SystemTap 4.0 中提供了`syscall_any`和`syscall_any.return`探测点，这将在即将发布的 Red Hat 开发者文章中讨论。

下面是移除了实现`syscall_any`的补丁的示例脚本时间图。下图中的峰值分别位于 17 ( `eventcount.stp`)、40 ( `stopwatches.stp`)、111 ( `syscallbypid-nd.stp`)、250 ( `thread-business.stp`)、255 ( `container_check.stp`)和 256 ( `errno.stp`)处，它们不在使用`syscall_any` tapset 的上图中。

[![Bar graph showing how long SystemTap took to convert example scripts to instrumentation without syscall_any tapset](img/8bb6a0c56d8fcd1ff4c0576a9993c0c2.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/no_syscall_any.png)

这个简单的示例脚本启动时间图表为我们提供了各种示例脚本的开销的快速指示，并提供了一些可视化的线索，表明哪些阶段的检测生成成本更高。

## 额外资源

参见[我关于红帽开发者的其他文章](https://developers.redhat.com/blog/author/wcohen2013/):

*   [用 SystemTap 使代码的操作更加透明和明显](https://developers.redhat.com/blog/2018/05/14/making-the-operation-of-code-more-transparent-and-obvious/)
*   [使用动态追踪工具，卢克](https://developers.redhat.com/blog/2018/05/11/use-the-dynamic-tracing-tools-luke/)
*   [了解应用程序需要哪些功能才能在容器中成功运行](https://developers.redhat.com/blog/2017/02/16/find-what-capabilities-an-application-requires-to-successful-run-in-a-container/)
*   [如何避免每次浪费几个字节的内存](https://developers.redhat.com/blog/2016/06/01/how-to-avoid-wasting-megabytes-of-memory-a-few-bytes-at-a-time/)
*   [提高处理器利用率的指令级多线程](https://developers.redhat.com/blog/2016/05/04/instruction-level-multithreading-to-improve-processor-utilization/)
*   [“不要过河”:光速下的线程安全和内存访问](https://developers.redhat.com/blog/2016/04/06/dont-cross-the-streams-thread-safety-and-memory-accesses-at-the-speed-of-light-2/)

Red Hat Developer 还有很多关于 [SystemTap](https://developers.redhat.com/blog/tag/systemtap/) 和[性能](https://developers.redhat.com/blog/category/performance/)的文章。

*Last updated: October 25, 2021*