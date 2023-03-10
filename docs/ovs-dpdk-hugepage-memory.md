# 打开 vSwitch-DPDK:多少 Hugepage 内存？

> 原文：<https://developers.redhat.com/blog/2018/03/16/ovs-dpdk-hugepage-memory>

## 介绍

为了最大化 [Open vSwitch DPDK](http://docs.openvswitch.org/en/latest/intro/install/dpdk/) 数据路径的性能，它会预分配大页面内存。作为用户，您有责任告诉 Open vSwitch 要预分配多少页面内存。经常会出现究竟使用什么值的问题。答案是，看情况。

没有简单的答案，因为它取决于端口的 MTU 大小、端口之间的 MTU 差异以及这些端口是否在同一个 NUMA 节点上。让事情变得更复杂的是，在 OVS-DPDK 的不同地方，需要考虑多个开销、对齐和舍入。一切正常吗？好吧，那你可以不看了！然而，如果没有，请继续阅读。

这个博客将帮助你决定用 OVS-DPDK 预分配多少页面内存。用许多行嵌套的 C 代码宏来填充博客以显示每个场景的精确计算是没有用的。相反，我们将给出一些指导方针和例子。这个博客与 OVS 版本 2.6、2.7、2.8 和 2.9 相关。

## 内存是用来做什么的？

我们讨论的内存主要用于表示 OVS-DPDK 中数据包的缓冲区。这些缓冲器中的一组或多组被预先分配，并且每当新的分组到达 OVS-DPDK 端口时，其中一个缓冲器被使用。OVS-DPDK 端口对于物理网卡可能是类型 *dpdk* ，对于虚拟网卡可能是*dpdkvshostuser、*或*dpdkvshostuserclient*。它们将只使用与端口关联的 NUMA 节点上的缓冲区。

## 内存池共享

一组相同大小的缓冲区存储在一个环形结构中，称为内存池。一个内存池可以用于单个端口，也可以在多个端口之间共享，条件是经过一些舍入后，端口具有匹配的缓冲区大小，并且与同一个 NUMA 节点相关联。

例如，MTU 为 1500 字节的 NUMA 0 上的物理 *dpdk* 端口可以与以下对象共享内存池:

```
A virtual vhost port on NUMA 0 with an MTU of 1500 Bytes
A physical dpdk port on NUMA 0 with an MTU of 1800 Bytes
```

但是，它不能与以下对象共享内存池:

```
A virtual vhost port on NUMA 0 with an MTU of 9000 Bytes
A physical dpdk port on NUMA 1 with an MTU of 1500 Bytes
```

## 缓冲区大小

从用户的角度来看，您只是为一个端口请求了一个 MTU，但这是 OVS-DPDK 中计算的起点。与其他 OVS 数据包类型的元数据集成和对齐舍入都需要考虑在内，并在代码中进行计算。它会根据 MTU 和舍入而变化，但可以肯定的是，每个缓冲区的开销会在 1000 字节到 2000 字节之间。

我们来看一些常见的例子。

如果您将端口的 MTU 设置为 1500 字节:

```
Total size per buffer = 3008 Bytes
```

在另一个将 MTU 设置为 9000 字节的示例中:

```
Total size per buffer = 10176 Bytes
```

这是每个单独缓冲区的大小。那么问题是，一个 mempool 中有多少缓冲区？

## 内存池大小

我们前面提到过，内存池可以在具有相似 MTU 和相同 NUMA 关联的多个端口之间共享，这并不少见。为了满足使用一个内存池的多个端口的需求，最初 OVS-DPDK 在每个内存池中请求 256K 个缓冲区。这似乎很多，但我们必须考虑到，对于这些端口中的任何一个，缓冲区可能正在进行处理，或者它们可能正在与多个物理 NIC 队列相关联地使用。在 OVS 2.9 中，它们也可以在软件发送队列中。

如果 256K 缓冲区不可用，则请求的数量减半，直到 16K。如果在 16K 时仍然不能创建内存池，就会报告一个错误。根据设置 MTU 的上下文，这意味着无法正确创建端口，或者 MTU 更改失败。

让我们将关于为 mempool 请求多少缓冲区的信息与每个缓冲区的总大小结合起来(来自上面 1500 字节和 9000 字节的例子)。

对于 1500 字节的 MTU:

```
Size of requested mempool = 3008 Bytes * 256K
Size of requested mempool = 788 MBytes
```

对于 9000 字节的 MTU:

```
Size of requested mempool = 10176 Bytes * 256K
Size of requested mempool = 2668 MBytes
```

现在让我们考虑这样一种情况，MTU 为 1500 字节和 9000 字节的端口在 NUMA 节点 0 上共存。MTU 差异太大，不允许共享内存池，因此每个 MTU 都有一个内存池。

最初，OVS-DPDK 将要求:

```
Size of mempools = 788 MBytes + 2668 MBytes
Size of mempools = 3456 MBytes
```

该图显示了两个物理端口和两个虚拟端口共享这些 MTU 的关联:

![](img/5541f797c27c9ba05488746e8e4f15f2.png)

## 设置大页面内存

从上面的示例中，我们知道我们希望在 NUMA 节点 0 上有 3456 MB 的可用空间，以支持 MTU 为 1500 字节和 9000 字节的端口。我们来设定吧！
通常，它以 1024 MB 的块为单位进行设置，因此在 NUMA 节点 0 上预分配 4096 MB。严格地说，这满足了这个例子的要求。但是，如上所述，端口将只使用与它们关联的同一个 NUMA 节点中的内存池，因此在其他 NUMA 节点上也进行预分配是一个好的做法。

```
# ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem=4096,4096
```

你可以在这篇[博文](https://developers.redhat.com/blog/2017/06/28/ovs-dpdk-parameters-dealing-with-multi-numa/)中找到更多关于 OVS-DPDK 参数(包括 *dpdk-socket-mem* 和 NUMA)的信息。

## 结论

很难简单计算出为 OVS-DPDK 预分配所需的内存。端口 MTU 和舍入往往会导致此变量。作为指南，使用上面的例子应该能够提供一个很好的近似计算。哦，好消息是，如果你弄错了，OVS-DPDK 将使用更少的内存重试。因此，这里面有一点灵活性！

*Last updated: September 18, 2018*