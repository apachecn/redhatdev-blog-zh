# 如何使用 Linux perf 工具统计软件事件

> 原文：<https://developers.redhat.com/blog/2019/04/23/how-to-use-the-linux-perf-tool-to-count-software-events>

Linux perf 工具最初是为了允许访问性能监控硬件而编写的，该硬件对硬件事件进行计数，例如执行的指令、处理器周期和高速缓存未命中。然而，它也可以用来统计软件事件，这对于测量系统软件的某个部分执行的频率是很有用的。

最近，有人在 Red Hat 询问是否有一种方法可以统计系统中正在执行的系统调用。内核有一个预定义的软件跟踪点`raw_syscalls:sys_enter`，它收集准确的信息。每次进行系统调用时，它都会计数。要使用跟踪点事件，需要以 root 用户身份运行`perf`命令。

以下代码将给出每秒(`-I 1000`)系统调用(`-e raw_syscalls:sys_enter`)的系统范围计数(`-a`选项):

```
# perf stat -a -e raw_syscalls:sys_enter -I 1000
#           time             counts unit events
     1.000640941              1,250      raw_syscalls:sys_enter                                      
     2.001183785              1,901      raw_syscalls:sys_enter                                      
     3.001601593              1,922      raw_syscalls:sys_enter   

```

`raw_syscalls:sys_enter`跟踪点只是内核中预定义的一个跟踪点事件。要列出其他 1000 多个预定义的跟踪点事件，请以 root 用户身份运行以下命令:

```
# perf list tracepoint

List of pre-defined events (to be used in -e):

  block:block_bio_backmerge                          [Tracepoint event]
  block:block_bio_bounce                             [Tracepoint event]
  block:block_bio_complete                           [Tracepoint event]
  block:block_bio_frontmerge                         [Tracepoint event]
  block:block_bio_queue                              [Tracepoint event]
  block:block_bio_remap                              [Tracepoint event]
  block:block_dirty_buffer                           [Tracepoint event]
  block:block_getrq                                  [Tracepoint event]
  block:block_plug                                   [Tracepoint event]
  ...

```

您可能希望为内核中还没有跟踪点的任意函数设置一个计数器。没问题。您可以定义自己的探测点，然后在`perf stat`命令中使用它们来监控执行高开销操作的函数。例如，清除 2MB 的大页面的延迟大约是清除传统 4KB 页面的 500 倍。这些延迟可能是显而易见的，您可能想知道何时会出现大量延迟。

以下设置了 perf 可访问的`clear_huge_page`功能中的探测点:

```
# perf probe --add clear_huge_page
Added new event:
  probe:clear_huge_page (on clear_huge_page)

You can now use it in all perf tools, such as:

	perf record -e probe:clear_huge_page -aR sleep 1

```

下面提供了每 10 秒(10，000 毫秒)的计数:

```
# perf stat -a -e probe:clear_huge_page -I 10000
#           time             counts unit events
    10.000241215                 73      probe:clear_huge_page                                       
    20.001129381                  4      probe:clear_huge_page                                       
    30.001567364                  3      probe:clear_huge_page                                       
    40.002202895                  2      probe:clear_huge_page                                       
    50.003554968                  1      probe:clear_huge_page                                       
    50.316752807                  0      probe:clear_huge_page
    ...

```

当不再需要`clear_huge_page`功能的探测点时，可以如下图所示将其移除。

```
# perf probe --del=probe:clear_huge_page
Removed event: probe:clear_huge_page

```

perf 探测点也可以放在用户空间的可执行文件中。您可能需要在启用 debug info(GCC 的`-g`选项)的情况下编译代码，或者安装 debuginfo RPMs 以允许 perf 找到函数的位置。要对 glibc 库中的`malloc`函数进行探测，需要用`--exec`选项指定可执行文件。

```
# perf probe --exec=/lib64/libc-2.17.so --add malloc
Added new event:
  probe_libc:malloc    (on malloc in /usr/lib64/libc-2.17.so)

You can now use it in all perf tools, such as:

	perf record -e probe_libc:malloc -aR sleep 1

```

使用`probe_libc:malloc`，您可以获得每 10 秒钟发生的`malloc`呼叫的数量。下面是最初闲置 20 秒的机器的输出。20 秒后，并行内核构建开始，调用 malloc 的次数急剧增加。

```
# perf stat -a -e probe_libc:malloc -I 10000
#           time             counts unit events
    10.000900150                  2      probe_libc:malloc                                           
    20.001803180                  0      probe_libc:malloc                                           
    30.002286255          1,829,385      probe_libc:malloc                                           
    40.002442647         12,553,306      probe_libc:malloc                                           
    50.002578104         15,579,692      probe_libc:malloc
    ...

```

一旦使用完用户空间探测器，就可以将其删除:

```
# perf probe --exec=/lib64/libc-2.17.so --del malloc
Removed event: probe_libc:malloc

```

将`perf stat`与软件探测点结合使用可以帮助您回答一些代码执行频率的问题。有关设置软件探测点的更多信息，请查看`perf-probe`手册页。

*Last updated: October 12, 2022*