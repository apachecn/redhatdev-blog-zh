# 什么是 BPF 地图，如何在 stapbpf 中使用它们

> 原文：<https://developers.redhat.com/blog/2017/12/15/bpf-maps-used-stapbpf>

与 SystemTap 的默认后端相比，stapbpf 最显著的特点之一是没有内核模块运行时。相反，内核中的 BPF 机制主要处理它的运行时。因此，如果 BPF 为我们提供一种方法，在多次调用 BPF 程序时维护状态，并使用户空间程序能够与 BPF 程序通信，那将是非常有用的。这是由 BPF 地图完成的。在这篇博文中，我将介绍 BPF 地图，并解释它们在 stapbpf 实施中的作用。

## 什么是 BPF 地图？

BPF 映射本质上是由键/值对组成的通用数据结构。它们是使用 BPF 系统调用从用户空间创建的，该系统调用返回地图的文件描述符。键大小和值大小由用户指定，允许存储任意类型的键/值对。一旦创建了映射，就可以使用 BPF 系统调用从用户空间访问元素。一旦创建映射的用户进程终止，映射就会被自动释放(尽管可以强制映射比该进程持续更长时间)。Stapbpf 使用以下函数创建新的 bpf 地图。

```
int bpf_create_map(enum bpf_map_type map_type, unsigned key_size,
                   unsigned value_size, unsigned max_entries)
{
  union bpf_attr attr;
  memset(&attr, 0, sizeof(union bpf_attr));
  attr.map_type = map_type;
  attr.key_size = key_size;
  attr.value_size = value_size;
  attr.max_entries = max_entries;

  return syscall(__NR_bpf, BPF_MAP_CREATE, &attr, sizeof(attr));
}
```

`map_type`表示将被实例化的 BPF 地图的类型。目前，大约有 15 种不同的地图类型。我将重点介绍一下`BPF_MAP_TYPE_HASH`和`BPF_MAP_TYPE_ARRAY`，这是 stapbpf 中最常用的两种地图类型。虽然这两种类型的映射都由键/值对组成，但 BPF 数组映射键必须是 32 位数组索引。与 BPF 散列表相比，BPF 数组映射的优势在于查找速度更快。然而，这种速度是有代价的，因为数组映射元素的更新是非原子的，而 HashMap 元素的更新是原子的。

从内核端，可以通过 BPF 助手函数访问 BPF 地图。这些函数可以从 BPF 程序中调用，它们提供了一个映射接口，与用户空间接口非常相似。事实上，通过 BPF 系统调用和 BPF 助手函数发出的映射查找和更新操作最终调用了相同的内核端函数。这些函数依次调用基于所操作的地图类型的专用函数(例如`array_map_lookup_elem`、`htab_map_lookup_elem`)。尽管下面的例子是在 stapbpf 中使用的用户空间函数，但是从内核端 bpf 程序中添加/更新地图元素的调用行为类似。

```
int bpf_update_elem(int fd, void *key, void *value, unsigned long long flags)
{
  union bpf_attr attr;
  memset(&attr, 0, sizeof(union bpf_attr));
  attr.map_fd = fd;
  attr.key = ptr_to_u64(key);
  attr.value = ptr_to_u64(value);
  attr.flags = flags;

  return syscall(__NR_bpf, BPF_MAP_UPDATE_ELEM, &attr, sizeof(attr));
}
```

BPF 相对于其他性能分析工具的主要优势之一是它对安全性的重视。这种强调也适用于内核对 BPF 地图的处理。例如，内核跟踪对每个 BPF 映射的引用数量，只有当这个计数达到 0 时才释放一个映射。此外，内核的 BPF 验证器将拒绝任何在调用 BPF 助手函数时不执行适当的错误检查的 BPF 程序，例如，返回指向地图元素的指针或试图用大小不正确的键来索引地图。

## BPF 地图在 stapbpf 中是如何使用的？

BPF 地图在 stapbpf 的实施中发挥着重要作用。首先，它们允许跨多个探测处理程序和跨对单个探测处理程序的多次调用来维护状态。这为我们提供了一种相对简单的方法来实现 stapbpf 中的全局变量。全局变量存储在 HashMap 中，任何引用它们的 BPF 程序都可以访问它。使用 HashMap 类型是为了确保对 Map 的任何并发读写都得到安全处理。

映射也用于实现局部变量。每个探测器都被分配了自己的映射，在映射中存储了自己的局部变量，并与其他探测器相隔离。在 stapbpf 中实现数组非常简单，因为大多数数组操作都非常自然地对应于 bpf 图上的操作。一个例外是数组元素的迭代。目前，BPF 没有提供一种便捷的方法来迭代数组，因为 BPF 目前拒绝任何包含循环的 BPF 程序。

bpf 映射在 stapbpf 中扮演了另一个关键角色，作为内核空间 BPF 程序和 stapbpf 的用户空间运行时之间的通信通道。这种通道的一个用途是通知 stapbpf 已经从探测处理程序调用了出口函数。该函数更新由 stapbpf 定期检查的映射中的值。一旦发现这个值被设置为 true，stapbpf 就开始关闭。

对于下面的 stap 脚本，我将描述在脚本执行期间如何使用 BPF 地图。

```
#! /usr/bin/stap
global total

probe kernel.function("vfs_write").return
{
  if ($return > 0)
  total += $return
}

probe timer.s(10)
{
  printf("%d bytes written in 10 seconds\n", total)
  exit()
}

```

这个简单的脚本记录并打印了`vfs_write`函数在十秒钟内写入的总字节数。Stapbpf 将其翻译成两个 bpf 程序:一个用于`vfs_write`返回探测器，另一个用于定时器探测器。在这个脚本的执行过程中使用了两个映射。Map 0 存储 stapbpf 的退出状态，而 map 1 存储脚本的全局变量(在这种情况下，map 1 只包含`total`)。为了确保安全的并发访问，两个映射都是 HashMap 类型。

现在，我将把注意力集中在这个短语上，并浏览一下它被翻译成的 BPF 指令。这通常包括读取当前值`$return`的指令，但为了简单起见，我不打算包括这个(不涉及 BPF 地图，而是在运行时使用 BPF 提供的数据结构读取这个值)。

```
  0:  [read the current value of $return and store it in r6]
  1:  r1 = addr_of_map1
  2:  *(u32 *)(r10 - 4) = 0
  3:  r2 = r10
  4:  r2 += -4
  5:  call bpf_map_lookup_elem
  6:  if r0 == 0x0 goto exit_insn
  7:  r0 = *(u64 *)(r0 + 0)
  8:  r0 += r6
  9:  r1 = addr_of_map1
  10: *(u32 *)(r10 - 12) = 0
  11: r2 = r10
  12: r2 += -12
  13: *(u64 *)(r10 - 8) = r0
  14: r3 = r10
  15: r3 += -8
  16: r4 = 0
  17: call bpf_map_update_elem

```

我们从已经存储在 BPF 寄存器 6 中的`$return`开始，所以下一步是读取`total`的当前值。为此，BPF 程序需要调用`bpf_map_lookup_elem`。作为参数，这个 BPF 助手函数接受一个 BPF 映射的地址和我们希望查找其值的键的地址。BPF 的 ABI 要求我们在调用`bpf_map_lookup_elem`之前将这些参数存储在`r1`和`r2`中。存储映象 1 的地址只需要一条指令，因为在装入这个 BPF 程序之前，映象 1 的地址作为立即值包含在指令 1 中。与`total`相关联的键恰好是 0，所以指令 2-4 包括将 0 存储在 BPF 堆栈上，并将它的地址存储在`r2`中。这里使用`r10`，因为它总是包含当前堆栈帧的地址。指令 5 包含对`bpf_map_lookup_elem`的调用，该调用将`total`的地址存储在`r0`中。但是，如果出现错误，将存储 0。内核的 BPF 验证器要求我们检查是否返回了 0。指令 6 执行该检查，如果检测到这样的错误，则跳转到退出指令。现在知道这个地址是有效的，我们将`total`存储在`r0`中并添加`$return`(指令 7 和 8)。

下一步是用新的 total 值更新 map 1。这需要调用`bpf_map_update_elem`。它有四个参数——映射地址、键、值和“标志”(标志让我们指定是添加新元素还是简单地更新现有元素)。指令 9 至 16 执行将这些参数存储在`r1`至`r4`中所需的步骤。这个过程与上一段所描述的非常相似。一旦完成，`bpf_map_update_elem`被调用，总数最终被设置为正确的值。每次`vfs_write`返回探测器启动时，重复这个过程。一旦定时器探头触发并完成打印，`exit`使用`bpf_map_update_elem`将退出状态设置为真。从这一点开始，这两个探测器将使用`bpf_map_lookup_elem`看到退出状态为真，并在 stapbpf 执行各种关闭程序时简单地执行早期返回。

## 未来工作和 BPF 备选方案

由于 stapbpf 处于早期开发阶段，在改进 bpf 地图的使用方式方面仍有许多工作要做。例如，虽然 SystemTap 的默认后端支持聚合、字符串数组和多维数组等集合，但 stapbpf 目前仅支持标量的一维数组。虽然 stapbpf 支持使用 bpf 哈希表对变量和数组元素进行原子更新，但默认后端支持首选探测处理程序级别的原子性。作为 stapbpf 的用户，bpf 地图本身仍然隐藏着实现细节。如果用户希望更直接地与 BPF 地图交互，其他 BPF 前端如 BCC (BPF 编译器集合)支持使用例如 C 和 Python 开发定制 BPF 工具。

* * *

**无论你是 Linux 新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/linux-cheatsheet/) **都可以在遇到你最近没做过的任务时辅助你。**

*Last updated: April 20, 2018*