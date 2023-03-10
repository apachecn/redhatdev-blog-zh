# 在 RHEL 使用快速数据路径(XDP)地图 8:第 2 部分

> 原文：<https://developers.redhat.com/blog/2018/12/17/using-xdp-maps-rhel8>

## 潜入 XDP

在这个关于 XDP 的系列文章的第一部分中，我介绍了 XDP，并讨论了一个最简单的例子。现在让我们尝试做一些不那么琐碎的事情，探索一些更高级的 [eBPF](https://lwn.net/Articles/740157/) 特性——地图——和一些常见的陷阱。

XDP 在 Red Hat Enterprise Linux 8 中可用，你现在就可以下载并运行它。

## [不是]重新发明轮子

我们将开始向示例中添加数据包解析；为了简化这样的任务，我们重用通用网络协议的内核定义，将以下内容添加到我们的 XDP 程序的`include`部分:

```
#include <linux/in.h>
#include <linux/if_ether.h>
#include <linux/if_packet.h>
#include <linux/if_vlan.h>
#include <linux/ip.h>

```

我们现在需要通过 XDP 上下文访问包内容。让我们来看看它的定义:

```
struct xdp_md {
    __u32 data;
    __u32 data_end;
    __u32 data_meta;
    /* Below access go through struct xdp_rxq_info */
    __u32 ingress_ifindex; /* rxq->dev->ifindex */
    __u32 rx_queue_index; /* rxq->queue_index */
};

```

数据包内容在`ctx->data`和`ctx->data_end`之间。因此，我们可以添加解析代码，并尝试以某种方式使用地址。在这种情况下，我们丢弃 IPv4 目的地址为零的数据包:

```
/* Parse IPv4 packet to get SRC, DST IP and protocol */
static inline int parse_ipv4(void *data, __u64 nh_off, void *data_end, __be32 *src, __be32 *dest)
{
    struct iphdr *iph = data + nh_off;

    *src = iph->saddr;
    *dest = iph->daddr;
    return iph->protocol;
}

SEC("prog")
int xdp_drop(struct xdp_md *ctx)
{
    void *data_end = (void *)(long)ctx->data_end;
    void *data = (void *)(long)ctx->data;
    struct ethhdr *eth = data;
    __be32 dest_ip, src_ip;
    __u16 h_proto;
    __u64 nh_off;
    int ipproto;

    nh_off = sizeof(*eth);

    /* parse vlan */
    h_proto = eth->h_proto;
    if (h_proto == __constant_htons(ETH_P_8021Q) ||
        h_proto == __constant_htons(ETH_P_8021AD)) {
        struct vlan_hdr *vhdr;

        vhdr = data + nh_off;
        nh_off += sizeof(struct vlan_hdr);
        h_proto = vhdr->h_vlan_encapsulated_proto;
    }
    if (h_proto != __constant_htons(ETH_P_IP))
        goto pass;

    ipproto = parse_ipv4(data, nh_off, data_end, &src_ip, &dest_ip);
    if (!dst_ip)
        return XDP_DROP;

pass:
    return XDP_PASS;
}

```

## 验证机跳闸

上面的代码应该可以很好地编译，但是如果我们试图用`iproute`加载它，我们会得到一个糟糕的惊喜:

```
Prog section 'prog' rejected: Permission denied (13)!
- Type: 6
- Instructions: 19 (0 over limit)
- License:

Verifier analysis:

0: (61) r1 = *(u32 *)(r1 +0)
1: (71) r2 = *(u8 *)(r1 +13)
invalid access to packet, off=13 size=1, R1(id=0,off=0,r=0)
R1 offset is outside of the packet

Error fetching program/map!

```

它未能通过验证者检查！验证器错误消息可能有些误导，因为我们正在访问包的前几个字节。我们知道每个以太网帧必须至少有 64 个字节长，因此，我们知道我们访问的是数据包有效载荷内的有效偏移量。

相反，验证器只依赖于显式检查:在访问/操作包内的任何偏移量之前，我们必须添加一个条件检查，以确保这样的偏移量
在包主体内。在我们的示例中，在访问每个报头之前，我们必须通过添加如下补丁来确保报头尾部在数据包末尾以下:

```
@@ -17,6 +17,9 @@ static inline int parse_ipv4(void *data, __u64 nh_off, void *data_end,
  {
      struct iphdr *iph = data + nh_off;

+     if (iph + 1 > data_end)
+         return 0;
+
      *src = iph->saddr;
      *dest = iph->daddr;
      return iph->protocol;
@@ -34,6 +37,8 @@ int xdp_drop(struct xdp_md *ctx)
      int ipproto;

      nh_off = sizeof(*eth);
+     if (data + nh_off > data_end)
+         goto pass;

      /* parse vlan */
      h_proto = eth->h_proto;
@@ -43,6 +48,8 @@ int xdp_drop(struct xdp_md *ctx)

      vhdr = data + nh_off;
      nh_off += sizeof(struct vlan_hdr);
+     if (data + nh_off > data_end)
+         goto pass;
      h_proto = vhdr->h_vlan_encapsulated_proto;
      }
      if (h_proto != __constant_htons(ETH_P_IP))

```

验证者现在应该高兴了吧！

## 定制 XDP 装载机

我们已经在第一部分中讨论过地图；让我们看看如何在实践中使用它们。我们希望增强我们的 XDP 程序，允许用户配置在运行时丢弃的地址，并且能够读取相关的统计数据。

作为第一步，我们需要用一个定制的加载程序替换`iproute2`工具，因为该工具不允许地图操作。用于加载 XDP 程序的代码片段应该是这样的:

```
#include <bpf/bpf.h>
#include <bpf/libbpf.h>
#include <error.h>

// [ ... ]
    struct bpf_prog_load_attr prog_load_attr = {
        .prog_type = BPF_PROG_TYPE_XDP,
        .file = "xdp_drop_kern.o",
    };
// [ ... ]
    if (bpf_prog_load_xattr(&prog_load_attr, &obj, &prog_fd))
        error(1, errno, "can't load %s", prog_load_attr.file);

    ifindex = if_nametoindex(dev_name);
    if (!ifindex)
        error(1, errno, "unknown interface %s\n", dev_name);
    if (bpf_set_link_xdp_fd(ifindex, prog_fd, 0) < 0)
        error(1, errno "can't attach to interface %s:%d: "
              "%d:%s\n", dev_name, ifindex, errno,
              strerror(errno));
// [ ... ]
    // cleaning-up
    bpf_set_link_xdp_fd(ifindex, -1, 0);

```

我们使用的是捆绑在 Linux 内核源代码中的`libbpf`助手库。`bpf_prog_load_xattr()`加载由`prog_load_attr`参数指定的 eBPF 程序。它将解析指定对象的所有 elf 部分，提取所有相关信息并将其放入`obj`状态数据中。每个找到的程序(文本部分)通过一个新分配的文件描述符(`prog_fd`)加载到内核中。

这种文件描述符稍后用于通过`bpf_set_link_xdp_fd()`功能将加载的程序附加到选定的设备上。最后一个参数允许用户指定几个标志，比如替换现有 XDP 程序的标志，或者使用驱动程序级 XDP 挂钩的标志。默认情况下:

*   它会尝试使用驱动级钩子，然后退回到普通钩子。
*   如果 XDP 程序已经安装在指定的设备上，它将失败。

最后，在程序终止时调用的最后一个助手将 XDP 程序从网卡上分离，并释放所有相关的内核资源。

## 与用户空间交互

现在让我们进入有趣的部分:地图！用户空间和 eBPF 程序之间共享的每个数据结构都被称为“map”，但是实际上有几种不同的类型:hashmap、array、queue 等等。通常，有两种不同的变体:简单的和每 CPU 的。对于每个 CPU 的变体，每个条目都是为所有本地可用的 CPU 复制的；在内核内部，每个 CPU 将只访问它的私有副本。每 CPU 变体避免了任何与争用相关的问题，当 eBPF 程序必须在每个包的基础上修改数据条目时，它是首选。

用户空间和 eBPF 程序都将访问地图数据。在双方包含的头文件中添加数据类型定义很方便。在本例中，我们使用映射来指定要过滤的源地址，并计算每个指定地址丢弃的字节数和数据包数。要将这样的地图添加到我们的程序中，我们需要这样的东西:

```
    // in xdp_drop_common.h
    struct stats_entry {
        __u64 packets;
        __u64 bytes;
    };

    // in xdp_drop_kern.c
    #include "xdp_drop_common.h"
    // [ ... ]
    /* forwarding map */
    struct bpf_map_def SEC("maps") egress_map = {
        .type = BPF_MAP_TYPE_PERCPU_HASH,
        .key_size = sizeof(__be32),
        .value_size = sizeof(struct stats_entry),
        .max_entries = 100,
    };

    // in xdp_drop_user.c
    struct bpf_map *map;
    int map_fd;
    // [ ... ]
    map = bpf_object__find_map_by_name(obj, "drop_map");
    if (!map)
        error(1, errno, "can't load drop_map");
    map_fd = bpf_map__fd(map);
    if (map_fd < 0)
        error(1, errno, "can't get drop_map fd");

```

注意，我们的映射实际上是一个基于 CPU 的哈希表，它的定义只包含键和值的大小，因为内核只需要这些信息来进行分配、执行查找和更新条目。映射定义还包含此类映射中允许的最大条目数。Hashmaps 最初是空的，插入上述限制将会失败。数组具有等于指定限制的固定大小。用户空间可以通过指定的文件描述符访问映射。在这里使用`libbpf`助手可能看起来有点复杂，但是当 eBPF 程序公开多个地图时，它真的很有帮助。

我们现在准备添加用户空间/eBPF 交互:

```
    // in xdp_drop_kern.c
    struct stats_entry entry;
    // [ ... ]
    stats = bpf_map_lookup_elem(&drop_map, &src_ip);
    if (!stats)
        goto pass;

    stats->packets++
    stats->bytes += ctx->data_end - ctx->data;
    return XDP_DROP;

    // in xdp_drop_user.c
    // [ ... ]
    memset(&entry, 0, sizeof(entry));
    if (bpf_map_update_elem(map_fd, &saddr, entry, BPF_ANY))
        error(1, errno, "can't add address %s\n", argv[i]);
    // [ ... ]
    if (bpf_map_lookup_elem(map_fd, &ipv4_addr, &entry))
        error(1, errnom "no stats for rule %x %x\n",
              ipv4_addr);
    printf("addr %x drop %ld:%ld\n", ipv4_addr,
           entry.packets, entry.bytes);

```

现在，eBPF 程序只有在`drop_map`散列表中找到源 IP 地址时才会丢弃数据包，并更新相关的统计信息。用户空间程序用归零的统计信息填充这样的映射，并(定期)查找这样的条目，打印出 eBPF 程序报告的统计信息。

为了简洁起见，省略了提取源地址[list]并优雅地终止的样板用户空间代码；当它被包括在内时，我们就准备好构建和运行了。

## 一些地图注意事项

使用当前代码获得的结果可能令人失望，从用户空间程序的随机崩溃到 eBPF 过滤器明显无效。如果用户空间程序异常终止，它将使 XDP 程序与网络设备保持连接，以后的执行将在启动时失败。在这种情况下，用户需要用`iproute`手动分离 XDP 程序:

```
ip link set dev <NIC> xdp off

```

在某些幸运的情况下，当前代码几乎可以完美地工作，只是在关闭时无法分离 XDP 程序。

虽然你们中的一些人可能已经猜到了问题出在哪里，但是我们将使用 XDP/eBPF 调试工具来转储程序状态，方法是将以下内容添加到`xdp_drop_kern`:

```
@@ -33,6 +33,13 @@ static inline int parse_ipv4(void *data, __u64 nh_off, void *data_end,
      return iph->protocol;
  }

+     #define bpf_printk(fmt, ...) \
+     ({ \
+         char ____fmt[] = fmt; \
+         bpf_trace_printk(____fmt, sizeof(____fmt), \
+         ##__VA_ARGS__); \
+     })
+
      SEC("prog")
      int xdp_drop(struct xdp_md *ctx)
      {
@@ -45,6 +52,8 @@ int xdp_drop(struct xdp_md *ctx)
      __u64 nh_off;
      int ipproto;

+     bpf_printk("xdp_drop\n");
+
      nh_off = sizeof(*eth);
      if (data + nh_off > data_end)
          goto pass;
@@ -72,6 +81,8 @@ int xdp_drop(struct xdp_md *ctx)
      if (!stats)
          goto pass;

+     bpf_printk("xdp_drop pkts %lld:%lld\n", stats->packets, stats->bytes);
+
      stats->packets++;
      stats->bytes += ctx->data_end - ctx->data;
      return XDP_DROP;

```

然后，我们可以再次运行该示例，并观察在:

```
/sys/kernel/debug/tracing/trace

```

对于每个入口包，eBPF 助手被正确调用。

如果您足够幸运的话，您可能会观察到与用户空间创建的 map 条目相关联的统计数据看起来被破坏了，例如，即使在条目创建之后接收到第一个包，也包含相当随机的值。

我们使用的是每 CPU 映射:当设置条目时，内核从指定的数据地址读取 *<个可能的 CPU>*值，将每个值复制到内核映射中相应的每 CPU 值中——在我们的例子中，访问的是用户空间程序堆栈中除`stats`变量以外的数据。

此外，当用户空间进程试图从 map 中读取一个条目时，内核将相同数量的数据复制到指定的地址，再次命中堆栈上的数据，并导致上面提到的随机行为。

解决方案是简单地为映射条目分配足够的存储空间，如下所示:

```
    int nr_cpus = sysconf(_SC_NPROCESSORS_CONF);
     struct stats_entry *entry;
// [ ... ]
    entry = calloc(nr_cpus, sizeof(struct stats_entry));
    if (!entry)
        error(1, 0, "can't allocate entry\n");

```

并且，在阅读地图时，遍历并聚合所有值:

```
    struct stats_entry all = { 0, 0};

    if (bpf_map_lookup_elem(map_fd, &ipv4_addr, entry))
        error(1, errno, "no stats for address %x\n",
              ipv4_addr);

    for (j = 0; j < nr_cpus; j++) {
        all.packets += entry[j].packets;
        all.bytes += entry[j].bytes;
    }

```

现在我们的 IP 过滤器应用程序已经准备好了！

## 前方的路

在本文中，我们介绍了 XDP/eBPF 提供的一些功能，但是还有更多。例如，有更多的 eBPF 助手可以用于各种任务:在一些修改之后更新包校验和、包转发等等。

一个很好的起点是 Linux 内核源代码中的这个头文件，它包含了已实现的助手的官方文档:

```
include/uapi/linux/bpf.h

```

此外，仍然在内核源代码中的`samples/bpf/`目录包含几个更复杂的 XDP 示例。不过，去那里之前需要相关的背景知识。

上面讨论的示例的完整源代码可以在以下位置找到:

```
https://github.com/altoor/xdp_walkthrough_examples

```

黑客快乐！

## 另请参见:

*   [利用 XDP 实现高性能、低延迟的网络连接](https://developers.redhat.com/blog/2018/12/06/achieving-high-performance-low-latency-networking-with-xdp-part-1/)(本系列文章的第 1 部分)
*   [使用 eBPF 进行网络调试](https://developers.redhat.com/blog/2018/12/03/network-debugging-with-ebpf/)
*   [红帽企业版 Linux 8 公告](https://wp.me/p8e0as-2rYd)

***[现在下载 RHEL 8](https://developers.redhat.com/rhel8/)。***

*Last updated: May 7, 2019*