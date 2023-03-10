# 用 XDP 实现高性能、低延迟网络:第一部分

> 原文：<https://developers.redhat.com/blog/2018/12/06/achieving-high-performance-low-latency-networking-with-xdp-part-1>

## XDP:从零到 14 Mpps

在过去的几年中，内核社区一直在使用不同的方法来寻求不断提高的网络性能。虽然在几个领域取得了显著的进步，但新一轮与架构相关的安全问题和相关应对措施抵消了大部分成果，而且针对一些数据包处理密集型工作负载的纯内核解决方案仍然落后于 bypass 解决方案，即数据平面开发套件(DPDK)，相差近一个数量级。

但是内核社区从不睡觉(几乎是字面上的意思),基于内核的网络性能的圣杯在 *XDP 的名字下被发现:快速数据路径*。XDP 在[红帽企业 Linux 8](https://wp.me/p8e0as-2rYd) 中可用，你现在就可以下载并运行。

该技术允许您通过自定义的[扩展 BPF (eBPF)](https://lwn.net/Articles/740157/) 程序来实现新的网络功能(和/或重新实现现有的功能)，该程序附加到堆栈深处的内核数据包处理，消除了套接字缓冲区(skb)管理的开销，减少了每个数据包的内存管理开销，并允许更有效的扩展。XDP eBPF 程序可以访问数据包操作和数据包转发的助手，提供了几乎无限的机会来改变和扩展内核行为，而无需添加新的内核代码，同时达到更高的处理速度。一个有 XDP 支持的现代司机可以轻松处理超过 14 辆 Mpps。

对这个机会感到兴奋，但对未知感到害怕？这篇文章将引导您走向您的第一个 XDP 程序，从零开始构建一个工作示例，并允许您从那里构建一个光速网络应用程序。

## XDP 概述

XDP 允许您将 eBPF 程序附加到内核内部的低级钩子上。在为当前数据包分配 SKB 之前，这种挂钩由网络设备驱动程序在入口流量处理功能(通常是 NAPI `poll()`方法)内部实现。

程序入口函数只有一个参数:描述当前包的上下文。该程序可以以任意的方式操纵这样的包，但是它必须遵守 eBPF 验证器所施加的约束(稍后会有更多的介绍)。最后，这样的程序必须返回一个控制动作给拥有它的设备驱动程序，指定设备驱动程序接下来应该如何处理已处理的数据包，例如，将它传递给上层处理或丢弃它。

eBPF 验证程序对您可以编写的代码施加了一些限制，执行严格的检查以确保程序:

*   不包含循环
*   仅访问有效内存(例如，不超出数据包边界)
*   使用有限数量的 eBPF 指令，因为 Linux 4.14 内核中的限制是 128K

验证器本身在内核中运行，并在 eBPF 程序通过 eBPF syscall 加载时执行。一旦程序被成功加载，它就可以连接到任何数量的设备的 XDP 挂钩上。内核源代码将用户空间库(`libbpf`)与助手函数捆绑在一起，以简化此类任务。

喜欢冒险的(或高级)用户可以直接使用 eBPF 汇编程序编写 eBPF 程序，但是使用更高级的编程语言并让编译器将代码翻译成 eBPF 要容易得多。eBPF 社区选择了 LLVM 编译器和 C 语言来完成这项任务。由于验证器对编译后的代码采取行动，编译器执行的任何优化都会影响结果。例如，编译器会展开固定迭代次数的循环。这允许您规避“无循环”的约束，但是它也增加了生成的 eBPF 指令的数量；嵌套循环很容易达到 128K 的限制。

## XDP 风味

并非所有的网络设备驱动程序都实现了 XDP 挂钩。在这种情况下，您可能会退回到通用的 XDP 钩子，它由核心内核实现，无论特定的网络设备驱动程序特性如何，它都是可用的。由于这种挂钩发生在网络堆栈的后期，在 SKB 分配之后，在那里观察到的性能比基于驱动程序的 XDP 挂钩低得多，但它仍然允许用 XDP 进行试验。Linux 4.18 及更高版本中支持 XDP 挂钩的网络设备驱动程序有:

*   bnxt
*   雷
*   i40e
*   ixgbe
*   mlx4
*   mlx5
*   nfp
*   qede
*   发酵桶
*   电源 _net

这个列表会随着内核版本的不同而不同，所以值得检查一下在运行的内核中是否明确支持你自己的驱动程序；见下文如何做到这一点。

最后，XDP/eBPF 程序的行为可以由用户空间通过程序定义和使用的许多映射来控制和检查。用户空间也可以访问和修改这样的地图，仍然使用`libbpf`助手。

## “你好，世界”

XDP 不是一种编程语言，但是它*使用*一种编程语言(eBPF ),并且每一个与编程相关的教程都必须从一个“Hello world”程序开始。虽然我们可以将调试消息回显到 XDP/eBPF 程序中——我们稍后会这样做——但我们将从一个更简单的 XDP 示例开始:eBPF 程序几乎什么都不做，只是将每个处理过的数据包传递到内核堆栈。

```
// SPDX-License-Identifier: GPL-2.0

#define KBUILD_MODNAME "xdp_dummy"
#include <uapi/linux/bpf.h>
#include "bpf_helpers.h"

SEC("proc")
int xdp_dummy(struct xdp_md *ctx)
{
    return XDP_PASS;
}

char _license[] SEC("license") = "GPL";

```

让我们看看上面代码的细节。它使用 C 语言语法，包括两个外部头。第一个由大多数发行版的内核头包提供，包含 XDP 程序返回代码的定义，在本例中为`XDP_PASS`，这意味着内核将把包传递给网络处理。其他可用的值有`XDP_DROP`(做同样的事情)、`XDP_TX`(从接收它的接口发回数据包)、`XDP_REDIRECT`(从另一个接口发送数据包)。

第二个头文件仍然是 Linux 内核的一部分，但大多数发行版通常不打包，它包含一个可用 eBPF 助手列表和
宏的定义。后者用于将编译后的对象片段放在不同的 ELF 节中。这些部分将由 eBPF 加载程序
解释，以检测例如程序定义的映射(并允许用户空间访问它们)。

最后，最后一行正式指定了与该程序相关的许可证。一些 eBPF 助手只能被 GPLed 程序访问，验证者将使用这些信息来执行这样的限制。

## 一些粗糙的边缘

让我们构建并运行它！需要 LLVM/Clang 版本 3.7 或更高版本(本文使用版本 6.0，Fedora 28 默认)和一个合理的、未过时的
`make`版本(这里是 4.2.1)。这是使用的 makefile:

```
KDIR ?= /lib/modules/$(shell uname -r)/source
CLANG ?= clang
LLC ?= llc
ARCH := $(subst x86_64,x86,$(shell arch))

BIN := xdp_dummy.o
CLANG_FLAGS = -I. -I$(KDIR)/arch/$(ARCH)/include \
-I$(KDIR)/arch/$(ARCH)/include/generated \
-I$(KDIR)/include \
-I$(KDIR)/arch/$(ARCH)/include/uapi \
-I$(KDIR)/arch/$(ARCH)/include/generated/uapi \
-I$(KDIR)/include/uapi \
-I$(KDIR)/include/generated/uapi \
-include $(KDIR)/include/linux/kconfig.h \
-I$(KDIR)/tools/testing/selftests/bpf/ \
-D__KERNEL__ -D__BPF_TRACING__ -Wno-unused-value -Wno-pointer-sign \
-D__TARGET_ARCH_$(ARCH) -Wno-compare-distinct-pointer-types \
-Wno-gnu-variable-sized-type-not-at-end \
-Wno-address-of-packed-member -Wno-tautological-compare \
-Wno-unknown-warning-option \
-O2 -emit-llvm

all: $(BIN)

xdp_dummy.o: xdp_dummy.c
$(CLANG) $(CLANG_FLAGS) -c $< -o - | \
$(LLC) -march=bpf -mcpu=$(CPU) -filetype=obj -o $@

```

这是相当重要的:因为程序使用内核头文件，我们需要处理特定于架构的头文件依赖性。用上面的 makefile 调用`make`来发布您的选择可能会令人失望，因为您可能会得到如下结果:

```
clang -nostdinc -I. \
[... long argument list elided for clarity ...]
-O2 -emit-llvm -c xdp_dummy.c -o - | \
llc -march=bpf -mcpu= -filetype=obj -o xdp_dummy.o
xdp_dummy.c:5:10: fatal error: 'bpf_helpers.h' file not found
#include "bpf_helpers.h"
^~~~~~~~~~~~~~~
1 error generated.

```

几乎没有发行包包含完整的内核源代码，包括所需的`bpf_helper.h`。不幸的是，目前的解决方案是下载完整的 Linux 源代码，将其解压到本地光盘上的某个地方，然后调用`make`如下:

```
KDIR= make

```

现在我们有了 eBPF/XDP 程序，但是我们必须将它加载到内核中才能获得任何效果。举个不那么简单的例子，加载部分通常由控制性的
用户空间程序来完成，该程序还将通过一些地图来监视 XDP 程序/与它交互。为了即时满足，我们可以用`iproute`来代替:

```
ip link set dev <net device name> xdp object xdp_dummy.o

```

上面的代码将我们的 XDP 程序附加到指定的网络设备上，尝试使用设备钩子，否则就退回到通用钩子。那些勇敢的并且有支持的设备驱动程序的人可以用`xdpdrv`代替`xdp`来强制使用驱动程序专用的钩子。如果`xdpdrv`不可用，连接将失败。

## 下一步是什么

有了您新编写的 XDP 程序，您可以体验前所未有的包过滤速度——除非您已经通过拔下以太网电缆做到了这一点——因为一个有 XDP 支持的现代驱动程序可以轻松处理 14 个以上的 Mpps！但是你可能会对做一些有用的事情感兴趣，比如丢弃特定的数据包或者收集关于 XDP 计划活动的统计数据。在本系列的下一篇文章中，我们将处理这个问题，通过介绍包解析、调试和映射用法，继续讨论一个不那么琐碎的例子！

您可能也会对本文感兴趣:[在 RHEL 使用快速数据路径(XDP)地图 8:第 2 部分](https://developers.redhat.com/blog/2018/12/17/using-xdp-maps-rhel8/)。

*Last updated: May 7, 2019*