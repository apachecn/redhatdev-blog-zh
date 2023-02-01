# SystemTap 的 BPF 后端引入了跟踪点支持

> 原文：<https://developers.redhat.com/blog/2018/04/23/systemtaps-bpf-backend-tracepoint-support>

这篇博客是 SystemTap 的 BPF (Berkeley Packet Filter)后端 stapbpf 系列的第三篇。在第一篇文章[介绍 STAP pbf——SystemTap 的新 bpf 后端](https://developers.redhat.com/blog/2017/12/13/introducing-stapbpf-systemtaps-new-bpf-backend/)中，我解释了什么是 BPF 以及它给 SystemTap 带来了什么特性。在第二篇文章中，[什么是 bpf 地图，它们在 STAP pbf 中是如何使用的](https://developers.redhat.com/blog/2017/12/15/bpf-maps-used-stapbpf/)，我研究了 BPF 地图，BPF 的关键组件之一，以及它们在 STAP pbf 实施中的作用。

在本文中，我将介绍 stapbpf 最近增加的对跟踪点探测的支持。跟踪点是 Linux 内核中静态插入的钩子，用户定义的探测器可以附加在其上。跟踪点可以在 Linux 内核的不同位置找到，包括性能关键的子系统，比如调度程序。因此，跟踪点探测器必须快速终止，以避免显著的性能损失或这些子系统中的异常行为。BPF 没有循环和 4k 指令的限制意味着它足以完成这项任务。

## 将跟踪点探测器与 stapbpf 一起使用

SystemTap 使用户可以很容易地编写 BPF 程序并将其附加到跟踪点。SystemTap 的高级脚本语言提供了一种与内核的 BPF 工具进行交互的简单方法。以下示例脚本将探测器附加到`mm_filemap_add_to_page_cache`跟踪点。它跟踪每个进程添加了多少页面，以及哪个进程添加了最多的页面。一旦跟踪点探测器运行了 30 秒钟，计时器探测器(也是一个 BPF 程序)就会触发，添加最多页面的进程会连同它添加的页面数量一起打印出来。然后通过`exit()`终止探测。

```
$ cat example.stp
global faults[250]
global max = -1

probe kernel.trace("mm_filemap_add_to_page_cache")
{
  faults[pid()]++

  if (max == -1 || faults[pid()] > faults[max])
    max = pid()
}

probe timer.s(30)
{
  if (max != -1)
    printf("Pid %d added %d pages\n", max, faults[max])
  else
    printf("No page cache adds detected\n")

  exit()
}

```

要使用 stapbpf 运行这个脚本，只需使用`stap --bpf`:

```
# stap --bpf example.stp
Pid 5099 added 5894 pages

```

## stapbpf 的优势

SystemTap 的脚本语言方便地抽象出各种低层次的 BPF 细节，这些细节可能与用户的查询无关，并可能使他们的调查复杂化，或者至少恶化与 BPF 工具相关的学习曲线。声明一个包含 250 个键-值对(`global fault[250]`)的 hashmap 并检查它是否包含特定的键(`pid() in fault`)这样的操作在 SystemTap 中表达起来非常简单。如果使用其他 BPF 工具，那么执行这些操作可能需要更多的冗长性或 BPF 内部的额外知识，例如各种类型的 BPF 映射，以及应该使用哪个内核提供的 BPF 助手函数来正确地访问该映射。

Stapbpf 还能够为不同于它当前运行的系统(主机)的内核版本创建跟踪点探测器。这对于目标机器需要最少的工具或者必须为尚未加载到内核中的模块编译探测器的情况非常有用。为了交叉编译探测器，stapbpf 直接从目标机器的内核头文件中获取跟踪点信息。

为此，stapbpf 使用了一种有趣的技术，该技术改编自 SystemTap 的默认(可加载内核模块)运行时。包含跟踪点定义的内核头文件与 SystemTap 创建的附加头文件一起编译。这些特定于 SystemTap 的头修改跟踪点定义宏，以便在结果模块中找到的调试信息包含构造探测器所需的信息。特别是，stapbpf 需要跟踪点参数的大小及其在探测执行期间的位置，以便可以正确地访问这些参数。SystemTap 也使用这种技术通过`@cast`操作符实现类型转换。这允许 stapbpf 的用户将 void 指针上下文变量转换为可以取消引用的类型:

```
probe kernel.trace("hrtimer_init")
{
  state = @cast($hrtimer, "hrtimer", "kernel<linux/hrtimer.h>")->state
  printf("hrtimer state: %d\n", state)
}
```

交叉编译要求在主机上有目标机器的内核构建树，在目标机器上有 stapbpf 二进制文件。然后只需使用`stap --remote`在本地编译探针，并通过 SSH 在目标机器上运行它们:

```
# stap --bpf --remote ssh://user@hostname example.stp
Pid 8346 added 89 pages

```

## 从哪里获得 SystemTap

Stapbpf 的开发正在进行中，因此建议您使用最新的源代码构建和安装 SystemTap。操作说明可在[这里](https://sourceware.org/git/?p=systemtap.git;a=blob_plain;f=README;hb=HEAD)找到。

*Last updated: November 15, 2018*