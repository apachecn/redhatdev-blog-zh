# 从 XDP 开始

> 原文：<https://developers.redhat.com/blog/2021/04/01/get-started-with-xdp>

XDP(快速数据路径)是 Linux 中一个强大的新网络功能，它支持在网络数据包进入网络堆栈之前对其进行高性能的可编程访问。但是 XDP 有很高的学习曲线。许多开发人员都为此功能撰写了介绍博客，如保罗·阿贝尼的 [*实现与 XDP 的高性能、低延迟联网:第一部分*](https://developers.redhat.com/blog/2018/12/06/achieving-high-performance-low-latency-networking-with-xdp-part-1/) 和托克的 *[使用 Red Hat Enterprise Linux 8](https://www.redhat.com/en/blog/using-express-data-path-xdp-red-hat-enterprise-linux-8)* 中的快速数据路径(XDP)。

XDP 是基于 [扩展的柏克莱包过滤( eBPF)](https://opensource.com/article/17/9/intro-ebpf) 并且还在快速发展。eBPF/XDP 编码格式和风格也在变化。因此，开发人员正在创建工具和框架，以使 eBPF 和 XDP 应用程序易于编写。这些资源中的两个，即 [libbpf](https://github.com/libbpf/libbpf.git) 库和 [xdp-tools](https://github.com/xdp-project/xdp-tools) 实用程序，是本文的主题。

本文展示了如何通过以下任务开始编写 XDP 程序:

1.  编写并运行一个小的介绍性程序:
    1.  写一个程序来丢弃所有的数据包。
    2.  构建并查看 BPF 对象。
    3.  加载一个 BPF 对象。
    4.  显示正在运行的 BPF 对象的信息。
    5.  卸载 BPF 对象。
2.  扩展程序，让您处理特定类型的数据包。
3.  使用数据包计数器来使用 BPF 地图。
4.  添加一个定制的用户空间工具来加载 BPF 程序。

读者需要熟悉 C 代码和 IP 头结构。所有示例都经过了 Red Hat Enterprise Linux (RHEL) 8.3 的测试。

## 先决条件

要准备开发环境，请安装以下软件包:

```
$ sudo dnf install clang llvm gcc libbpf libbpf-devel libxdp libxdp-devel xdp-tools bpftool kernel-headers
```

## 任务 1:和 XDP 一起编写并运行一个简单的程序

本节讲述使用 XDP 需要完成的最少任务。

### 任务 1.1:编写一个程序来丢弃所有数据包

让我们从一个简单的 C 语言 XDP 程序开始:

```
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

SEC("xdp_drop")
int xdp_drop_prog(struct xdp_md *ctx)
{
    return XDP_DROP;
}

char _license[] SEC("license") = "GPL";
```

内核头文件包提供了`linux/bpf.h`头文件，它定义了所有支持的 BPF 助手和 xdp_actions，就像本例中使用的`XDP_DROP`动作。

libbpf-devel 提供了`bpf/bpf_helpers.h`头，它提供了一些有用的 eBPF 宏，比如本例中使用的`SEC`宏。`SEC`，是*部分的缩写，*用于将编译后的对象的一个片段放在不同的 ELF 部分，我们将在后面的`llvm-objdump`命令输出中看到。

`xdp_drop_prog()`函数有一个参数`struct xdp_md *ctx`，我们还没有用到它。我以后再讲。这个函数返回`XDP_DROP`，这意味着我们将丢弃所有传入的数据包。其他 XDP 动作包括`XDP_PASS`、`XDP_TX`和`XDP_REDIRECT`。

最后，最后一行正式指定了与该程序相关的许可证。一些 eBPF 助手只能被 GPL 许可的程序访问，验证者将使用这些信息来实施这样的限制。

### 任务 1.2:构建并转储 BPF 对象

让我们用`clang`构建上一节中的程序:

```
$ clang -O2 -g -Wall -target bpf -c xdp_drop.c -o xdp_drop.o 
```

`-O`选项指定使用哪个优化级别，而`-g`生成调试信息。

您可以使用`llvm-objdump`来显示构建后的 ELF 格式。如果你想知道一个程序是做什么的，而你又没有源代码，那么`llvm-objdump`是非常有用的。`-h`选项显示对象中的部分，`-S`选项显示与反汇编对象代码交错的源代码。我们将依次展示这些选项。

```
$ llvm-objdump -h xdp_drop.o 
xdp_drop:       file format ELF64-BPF

Sections:
Idx Name            Size     VMA              Type
  0                 00000000 0000000000000000
  1 .strtab         000000ad 0000000000000000
  2 .text           00000000 0000000000000000 TEXT
  3 xdp_drop        00000010 0000000000000000 TEXT
  4 license         00000004 0000000000000000 DATA
  5 .debug_str      00000125 0000000000000000
  6 .debug_abbrev   000000ba 0000000000000000
  7 .debug_info     00000114 0000000000000000
  8 .rel.debug_info 000001c0 0000000000000000
  9 .BTF            000001df 0000000000000000
 10 .rel.BTF        00000010 0000000000000000
 11 .BTF.ext        00000050 0000000000000000
 12 .rel.BTF.ext    00000020 0000000000000000
 13 .eh_frame       00000030 0000000000000000 DATA
 14 .rel.eh_frame   00000010 0000000000000000
 15 .debug_line     00000084 0000000000000000
 16 .rel.debug_line 00000010 0000000000000000
 17 .llvm_addrsig   00000002 0000000000000000
 18 .symtab         000002d0 0000000000000000

$ llvm-objdump -S -no-show-raw-insn xdp_drop.o 
xdp_drop:       file format ELF64-BPF

Disassembly of section xdp_drop:

0000000000000000 xdp_drop_prog:
;       return XDP_DROP;
       0:       r0 = 1
       1:       exit

```

### 任务 1.3:加载 BPF 对象

构建完对象后，有多种方法可以加载它。

> **警告**:不要在默认界面加载测试 XDP 程序。相反，使用 [veth 接口](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/#veth)进行测试。这是明智的，以保护您不失去网络连接，因为该程序正在丢弃数据包。

加载程序最简单的方法是使用`ip`命令，就像这样:

```
$ sudo ip link set veth1 xdpgeneric obj xdp_drop.o sec xdp_drop 
```

但是`ip`不支持我们后面要讲的 [BPF 类型格式(BTF)](https://www.kernel.org/doc/html/latest/bpf/btf.html) 类型地图。虽然最新的 [ip-next](https://git.kernel.org/pub/scm/network/iproute2/iproute2-next.git/commit/?id=f98ce50046b433687c0a661b6b9107a0603d1058) 版本通过添加 libbpf 支持修复了这个问题，但它还没有被反向移植到主线上。

在 RHEL 上加载 XDP 对象的推荐方式是使用`xdp-loader`。这个命令依赖于 libbpf，它完全支持 BTF，并且是在一个界面上加载多个程序的唯一方法。

要获得对 XDP 的 Red Hat 支持，请使用`libxdp`，正如文章[中所解释的，XDP 在 RHEL 8.3 发行说明](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/8.3_release_notes/rhel-8-3-0-release#enhancement_networking)中是有条件支持的。

现在让我们用`xdp-loader`加载接口`veth1`上的对象。我们指定`-m sbk`使用 skb 模式。其他可能的模式包括本机和卸载。但是因为不是所有的网卡驱动都支持这些模式，所以在本文中我们只使用 skb 模式。`-s xdp_drop`选项指定了我们创建的部分`xdp_drop`的用途:

```
$ sudo xdp-loader load -m skb -s xdp_drop veth1 xdp_drop.o 
```

### 任务 1.4:显示正在运行的 BPF 对象的信息

还有多种方法可以显示关于已加载的 XDP 程序的信息:

```
$ sudo xdp-loader status CURRENT XDP PROGRAM STATUS:

Interface        Prio  Program name     Mode     ID   Tag               Chain actions
-------------------------------------------------------------------------------------
lo               <no XDP program>
ens3             <no XDP program>
veth1                  xdp_dispatcher   skb      15   d51e469e988d81da
 =>              50    xdp_drop_prog             20   57cd311f2e27366b  XDP_PASS

$ sudo bpftool prog show
15: xdp  name xdp_dispatcher  tag d51e469e988d81da  gpl
        loaded_at 2021-01-13T03:24:43-0500  uid 0
        xlated 616B  jited 638B  memlock 4096B  map_ids 8
        btf_id 12
20: ext  name xdp_drop_prog  tag 57cd311f2e27366b  gpl
        loaded_at 2021-01-13T03:24:43-0500  uid 0
        xlated 16B  jited 40B  memlock 4096B
        btf_id 16

$ sudo ip link show veth1
4: veth1@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 xdpgeneric qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ba:4d:98:21:3b:b3 brd ff:ff:ff:ff:ff:ff link-netns ns
    prog/xdp id 15 tag d51e469e988d81da jited

```

如果用`ip` cmd 加载程序，只能同时加载一个 XDP 程序。如果用`xdp-loader`加载你的程序，默认会加载两个程序。一个是`xdp_loader`创作的`xdp_dispatcher`，一个是我们写的`xdp_drop_prog`。上面发出的第二条命令显示`xdp_dispatcher`以 ID 15 运行，而`xdp_drop_prog`以 ID 20 运行。

### 任务 1.5:卸载 XDP 计划

如果您使用`ip cmd`加载程序，您可以通过以下方式卸载程序:

```
$ sudo ip link set veth1 xdpgeneric off 
```

使用与您加载文件的方式相对应的`xdp`标志。在这个例子中，我们指定了`xdpgeneric off`，因为我们从`ip link set veth1 xdpgeneric obj`开始加载程序。如果你用`ip link set veth1 xdp obj`加载你的开始，指定`xdp off`。

要卸载一个接口上的所有 XDP 程序，发出带有`-a`选项的`xdp-loader`:

```
$ sudo xdp-loader unload -a veth1 
```

## 任务 2:丢弃带有 XDP 的特定数据包

第一个例子丢弃了每个数据包，这没有实际用途。现在让我们做些真正的事情。本节中的示例会丢弃所有 IPv6 数据包:

```
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>
#include <linux/if_ether.h>
#include <arpa/inet.h>

SEC("xdp_drop")
int xdp_drop_prog(struct xdp_md *ctx)
{
    void *data_end = (void *)(long)ctx->data_end;
    void *data = (void *)(long)ctx->data;
    struct ethhdr *eth = data;
    __u16 h_proto;

    if (data + sizeof(struct ethhdr) > data_end)
        return XDP_DROP;

    h_proto = eth->h_proto;

    if (h_proto == htons(ETH_P_IPV6))
        return XDP_DROP;

    return XDP_PASS;
}

char _license[] SEC("license") = "GPL";

```

将刚才显示的代码与第一个程序进行比较。这里我们添加了另外两个头文件:`linux/if_ether.h`来获取`ethhdr`结构，`arpa/inet.h`来获取`htons()`函数。

本例中也使用了`struct xdp_md`。它是这样定义的(在 Linux 5.10 中):

```
struct xdp_md {
    __u32 data;
    __u32 data_end;
    __u32 data_meta;
    /* Below access go through struct xdp_rxq_info */
    __u32 ingress_ifindex; /* rxq->dev->ifindex */
    __u32 rx_queue_index;  /* rxq->queue_index  */

    __u32 egress_ifindex;  /* txq->dev->ifindex */
};

```

数据包内容位于`ctx->data`和`ctx->data_end`之间。数据以以太网报头开始，因此我们将数据分配给`ethhdr`,如下所示:

```
    void *data = (void *)(long)ctx->data;
    struct ethhdr *eth = data;

```

当访问`struct ethhdr`中的数据时，我们必须通过检查`data + sizeof(struct ethhdr) > data_end`是否为真来确保我们没有访问无效的区域，如果为真则返回而不采取进一步的行动。这个检查是由在运行时验证你的程序的 [BPF 验证器](https://www.kernel.org/doc/html/latest/networking/filter.html#ebpf-verifier)强制进行的。

然后，通过检查`h_proto == htons(ETH_P_IPV6)`确定以太网报头中的协议是否为 IPv6，如果是，则通过返回`XDP_DROP`丢弃数据包。对于其他数据包，我们只返回`XDP_PASS`。

## 任务 3:映射并统计已处理的数据包

在前面的示例中，我们丢弃了 IPv6 数据包。在本例中，我们将记录我们丢弃了多少数据包。这个例子介绍了 BPF 的另一个特性:地图。BPF 映射用于在内核和用户空间之间共享数据。我们可以在内核中更新地图数据，并从用户空间读取它，反之亦然。

下面是一个新的 BTF 定义的映射的例子(由上游[提交 abd29c931459](https://git.kernel.org/pub/scm/linux/kernel/git/netdev/net.git/commit/?id=abd29c931459) 引入):

```
struct {
        __uint(type, BPF_MAP_TYPE_PERCPU_ARRAY);
        __type(key, __u32);
        __type(value, long);
        __uint(max_entries, 1);
} rxcnt SEC(".maps");

```

该地图命名为`rxcnt`，类型为`BPF_MAP_TYPE_PERCPU_ARRAY`。这种类型表示每个 CPU 内核将有一个 map 实例；因此，如果您有 4 个核心，您将有 4 个地图实例。我们将使用每个映射来计算每个内核处理了多少个数据包。该结构的其余部分定义了键/值类型，并将最大条目数限制为 1，因为我们只需要计算一个数字(收到的 IPv6 数据包的数量)。在 C 代码中，看起来我们在每个 CPU 上定义了一个大小为 1 的数组，例如`unsigned int rxcnt[1]`。

我们的程序用函数`bpf_map_lookup_elem()`查找`rxcnt`条目的值，并更新该值。以下是完整的代码:

```
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>
#include <linux/if_ether.h>
#include <arpa/inet.h>

struct {
        __uint(type, BPF_MAP_TYPE_PERCPU_ARRAY);
        __type(key, __u32);
        __type(value, long);
        __uint(max_entries, 1);
} rxcnt SEC(".maps");

SEC("xdp_drop_ipv6")
int xdp_drop_ipv6_prog(struct xdp_md *ctx)
{
        void *data_end = (void *)(long)ctx->data_end;
        void *data = (void *)(long)ctx->data;
        struct ethhdr *eth = data;
        __u16 h_proto;
        __u32 key = 0;
        long *value;

        if (data + sizeof(struct ethhdr) > data_end)
                return XDP_DROP;

        h_proto = eth->h_proto;

        if (h_proto == htons(ETH_P_IPV6)) {
                value = bpf_map_lookup_elem(&rxcnt, &key);
                if (value)
                        *value += 1;
                return XDP_DROP;
        }

        return XDP_PASS;
}

char _license[] SEC("license") = "GPL";

```

让我们将新程序命名为`xdp_drop_ipv6_count.c`，并构建它来创建目标文件`xdp_drop_ipv6_count.o`。加载对象后，向该接口发送一些 IPv6 数据包。使用`bpftool map show`命令，我们可以看到地图中的`rxcnt` ID 是 13。然后我们可以用`bpftool map dump id 13`来说明 CPU 0 处理了 13 个包，CPU 1 处理了 7 个包:

```
$ sudo xdp-loader load -m skb -s xdp_drop_ipv6 veth1 xdp_drop_ipv6_count.o 
...*receive some IPv6 packets*

$ sudo bpftool map show bpftool map show
13: percpu_array  name rxcnt  flags 0x0
        key 4B  value 8B  max_entries 1  memlock 4096B
        btf_id 20
19: array  name xdp_disp.rodata  flags 0x480
        key 4B  value 84B  max_entries 1  memlock 8192B
        btf_id 28  frozen
# bpftool map dump id 13
[{
        "key": 0,
        "values": [{
                "cpu": 0,
                "value": 13
            },{
                "cpu": 1,
                "value": 7
            },{
                "cpu": 2,
                "value": 0
            },{
                "cpu": 3,
                "value": 0
            }
        ]
    }
]

```

BPF 支持更多的[地图类型](https://prototype-kernel.readthedocs.io/en/latest/bpf/ebpf_maps_types.html)，比如 BPF _ 地图 _ 类型 _ 哈希、BPF _ 地图 _ 类型 _ 数组等。

## 任务 4:使用自定义加载程序加载 XDP 对象

我们可以用`ip`加载 XDP 对象，用`bpftool`显示地图编号。但是如果我们想要更高级的功能(创建、读取和写入地图，将 XDP 程序附加到界面上，等等)。)，需要我们自己写加载器。

这是一个如何显示我们丢弃的总包数和每秒包数(PPS)的例子。在这个例子中，我硬编码了很多东西，比如内核对象名、节名等等。这些都可以通过您自己代码中的参数来设置。

代码中的注释解释了每个函数的用途:

```
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <linux/if_link.h>
#include <signal.h>
#include <net/if.h>
#include <assert.h>

/* In this example we use libbpf-devel and libxdp-devel */
#include <bpf/bpf.h>
#include <bpf/libbpf.h>
#include <xdp/libxdp.h>

/* We define the following global variables */
static int ifindex;
struct xdp_program *prog = NULL;

/* This function will remove XDP from the link when the program exits. */
static void int_exit(int sig)
{
    xdp_program__close(prog);
    exit(0);
}

/* This function will count the per-CPU number of packets and print out
 * the total number of dropped packets number and PPS (packets per second).
 */
static void poll_stats(int map_fd, int interval)
{
    int ncpus = libbpf_num_possible_cpus();
    if (ncpus < 0) {
        printf("Error get possible cpus\n");
        return;
    }
    long values[ncpus], prev[ncpus], total_pkts;
    int i, key = 0;

    memset(prev, 0, sizeof(prev));

    while (1) {
        long sum = 0;

        sleep(interval);
        assert(bpf_map_lookup_elem(map_fd, &key, values) == 0);
        for (i = 0; i < ncpus; i++)
            sum += (values[i] - prev[i]);
        if (sum) {
            total_pkts += sum;
            printf("total dropped %10llu, %10llu pkt/s\n",
                   total_pkts, sum / interval);
        }
        memcpy(prev, values, sizeof(values));
    }
}

int main(int argc, char *argv[])
{
    int prog_fd, map_fd, ret;
    struct bpf_object *bpf_obj;

    if (argc != 2) {
        printf("Usage: %s IFNAME\n", argv[0]);
        return 1;
    }

    ifindex = if_nametoindex(argv[1]);
    if (!ifindex) {
        printf("get ifindex from interface name failed\n");
        return 1;
    }

    /* load XDP object by libxdp */
    prog = xdp_program__open_file("xdp_drop_ipv6_count.o", "xdp_drop_ipv6", NULL);
    if (!prog) {
        printf("Error, load xdp prog failed\n");
        return 1;
    }

    /* attach XDP program to interface with skb mode
     * Please set ulimit if you got an -EPERM error.
     */
    ret = xdp_program__attach(prog, ifindex, XDP_MODE_SKB, 0);
    if (ret) {
        printf("Error, Set xdp fd on %d failed\n", ifindex);
        return ret;
    }

    /* Find the map fd from the bpf object */
    bpf_obj = xdp_program__bpf_obj(prog);
    map_fd = bpf_object__find_map_fd_by_name(bpf_obj, "rxcnt");
    if (map_fd < 0) {
        printf("Error, get map fd from bpf obj failed\n");
        return map_fd;
    }

    /* Remove attached program when it is interrupted or killed */
    signal(SIGINT, int_exit);
    signal(SIGTERM, int_exit);

    poll_stats(map_fd, 2);

    return 0;
}

```

将`ulimit`设置为无限制，并使用`-lbpf -lxdp`标志构建程序。然后运行程序，显示数据包计数输出:

```
$ sudo ulimit -l unlimited
$ gcc xdp_drop_ipv6_count_user.c -o xdp_drop_ipv6_count -lbpf -lxdp
$ sudo ./xdp_drop_ipv6_count veth1
total dropped          2,          1 pkt/s
total dropped        129,         63 pkt/s
total dropped        311,         91 pkt/s
total dropped        492,         90 pkt/s 
total dropped        674,         91 pkt/s
total dropped        856,         91 pkt/s
total dropped       1038,         91 pkt/s
^C

```

## 摘要

本文帮助您了解 XDP 程序是什么样子，如何添加 BPF 地图，以及如何编写自定义加载程序。要了解更多关于 XDP 编程的信息，请访问[xdp-教程](https://github.com/xdp-project/xdp-tutorial)。

*Last updated: April 7, 2022*