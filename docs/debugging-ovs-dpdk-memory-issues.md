# 使用 Open vSwitch DPDK 调试内存问题

> 原文：<https://developers.redhat.com/blog/2018/06/14/debugging-ovs-dpdk-memory-issues>

## 介绍

本文是关于使用数据平面开发套件 (OvS-DPDK)使用 [Open vSwitch 调试内存不足问题。它解释了使用 OvS-DPDK 时内存不足的情况，并显示了在这些情况下生成的日志条目。它还显示了一些其他日志条目和命令，以便进一步调试。](http://docs.openvswitch.org/en/latest/intro/install/dpdk/)

当您读完这篇文章时，您将能够发现您有一个内存不足的问题，并且您将知道如何修复它。剧透:通常在相关的 NUMA 节点上有更多的内存。它基于 OvS 2.9。

## 背景

与 DPDK 类型的应用程序一样，预计已经设置并安装了大页面内存。更多信息请参见[设置大页面](http://docs.openvswitch.org/en/latest/intro/install/dpdk/?highlight=hugepage#setup-hugepages)。

下一步是指定为 OvS-DPDK 预分配的内存量。这是使用开放式 vSwitch 数据库(OVSDB)完成的。在下面的例子中，在 NUMA 节点 0 和 NUMA 节点 1 上预先分配了 4GB 的大页面内存。

```
# ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem=4096,4096
```

如果未指定`dpdk-socket-mem`，NUMA 0 的默认值为 1GB。

现在，让我们来看看内存不足的时候。

## 初始化

当 DPDK 初始化时，可能会耗尽内存，这发生在`ovs-vswitchd`正在运行并且 OVSDB 条目`dpdk-init`被设置为`true`时。

初始化期间需要注意的一个有用的日志条目是:

```
|dpdk|INFO|EAL ARGS: ovs-vswitchd -c 0x1 --socket-mem 4096,4096
```

这将确认您*认为您正在设置的*实际上已经被设置并被传递给 DPDK(从而避免了别人指出您的脚本是错误的尴尬)。

初始化过程中最有可能耗尽内存的原因是没有正确设置大量的页面内存:

```
|dpdk|INFO|EAL ARGS: ovs-vswitchd -c 0x1 --socket-mem 4096,4096
|dpdk|INFO|EAL: 32 hugepages of size 1073741824 reserved, but no mounted hugetlbfs found for that size
```

另一种方式是您请求了过多的内存:

```
|dpdk|INFO|EAL ARGS: ovs-vswitchd -c 0x1 --socket-mem 32768,0
|dpdk|ERR|EAL: Not enough memory available on socket 0! Requested: 32768MB, available: 16384MB
```

或者你根本没有要求:

```
|dpdk|INFO|EAL ARGS: ovs-vswitchd -c 0x1 --socket-mem 0,0
|dpdk|ERR|EAL: invalid parameters for --socket-mem
```

所有这些问题都可以通过正确设置大型页面并请求预分配适当的数量来解决。

## 添加端口或更改 MTU

这些情况被组合在一起，因为它们都可能导致为端口请求新的缓冲池。在可能的情况下，这些缓冲池将被共享和重用，但由于不同的端口 NUMA 节点或 MTU，这并不总是可能的。

对于新请求，每个缓冲区的大小是固定的(基于 MTU ),但缓冲区的数量是可变的，如果没有足够的内存用于初始请求，OvS-DPDK 将重试较少数量的缓冲区。

当 DPDK 无法为任何一个请求提供所请求的内存时，它会报告以下信息:

```
|dpdk|ERR|RING: Cannot reserve memory
```

虽然这可能看起来很严重，但没什么可担心的，因为 OvS 会处理这个问题，只需重试更低的次数。但是，如果重试不起作用，则日志中会出现以下内容:

```
|netdev_dpdk|ERR|Failed to create memory pool for netdev dpdk0, with MTU 9000 on socket 0: Cannot allocate memory
```

这种情况是一个功能问题。

*   如果您正在添加端口，它将不可用。
*   如果您正在更改 MTU，MTU 更改会失败，但端口将继续使用以前的 MTU 运行。

如何修复这些错误？一般的指导只是在相关的 NUMA 节点上给 OvS-DPDK 更多的内存，或者坚持使用较低的 MTU。

## 启动虚拟机

与为它添加 vhost 端口(上一节)相比，启动 VM 时内存不足的原因似乎并不明显。关键是 NUMA 的重新分配。

当 VM 启动时，DPDK 检查从来宾共享的内存的 NUMA 节点。这可能导致从同一 NUMA 节点请求新的缓冲池。但是当然，在 NUMA 节点上可能没有预先分配给`dpdk-socket-mem`的内存，或者可能没有足够的内存。

日志条目类似于添加端口/更改 MTU 的情况:

```
|netdev_dpdk|ERR|Failed to create memory pool for netdev vhost0, with MTU 1500 on socket 1: Cannot allocate memory
```

修复方法是在相关的 NUMA 节点上拥有足够的内存，或者更改 libvirt/QEMU 设置，以便虚拟机内存来自不同的 NUMA 节点。

## 运行时，添加端口或添加队列

我们不是已经讨论过添加端口了吗？是的，我们做到了；然而，这个部分是针对当我们得到一个请求的缓冲池，但是一段时间后，这被证明是不够的。

这可能是因为有许多端口和队列共享一个缓冲池，当一些缓冲区为 Rx 队列保留时，一些正在处理，一些正在等待从 Tx 队列返回，因此没有足够的缓冲区分配。

例如，在使用物理网卡时出现这种情况时，日志条目可能如下所示:

```
|dpdk|ERR|PMD: ixgbe_alloc_rx_queue_mbufs(): RX mbuf alloc failed queue_id=0
|dpdk|ERR|PMD: ixgbe_dev_rx_queue_start(): Could not alloc mbuf for queue:0
|dpdk|ERR|PMD: ixgbe_dev_start(): Unable to start rxtx queues
|dpdk|ERR|PMD: ixgbe_dev_start(): failure in ixgbe_dev_start(): -1
|netdev_dpdk|ERR|Interface dpdk0 start error: Input/output error
```

对于 vhost 端口，不会预留缓冲区，但是您可以在运行时看到，在轮询 vhost 端口时无法获得新的缓冲区。日志条目可能如下所示:

```
|dpdk(pmd91)|ERR|VHOST_DATA: Failed to allocate memory for mbuf.
```

如果需要所有端口，最简单的解决方法是减少物理网卡的接收队列或保留缓冲区的数量。这可以通过以下命令完成:

```
# ovs-vsctl set Interface dpdk0 options:n_rxq=4
```

或者使用以下命令:

```
# ovs-vsctl set Interface dpdk0 options:n_rxq_desc=1024
```

或者，可以增加内存以确保有一个大的缓冲池可用(也就是说，避免较少量的重试)，但这种方法只能扩展到目前为止。

## 进一步调试

如果内存不足，日志中会有一条错误消息。如果您想进一步了解正在分配、重用和释放的内存池的详细信息，您可以打开调试模式:

```
# ovs-appctl vlog/set netdev_dpdk:console:dbg
# ovs-appctl vlog/set netdev_dpdk:syslog:dbg
# ovs-appctl vlog/set netdev_dpdk:file:dbg
```

分配、重用和释放的消息将如下所示:

```
|netdev_dpdk|DBG|Allocated "ovs_mp_2030_0_262144" mempool with 262144 mbufs
|netdev_dpdk|DBG|Reusing mempool "ovs_mp_2030_0_262144"
|netdev_dpdk|DBG|Freeing mempool "ovs_mp_2030_0_262144"
```

缓冲池的名称(即`mempool`)为我们提供了一些信息:

```
2030   : Padded size of the buffer (derived from MTU)

0      : NUMA node the memory is allocated from

262144 : Number of buffers in the pool
```

还有一个命令显示端口正在使用哪个`mempool`，以及许多其他细节(未显示):

```
# ovs-appctl netdev-dpdk/get-mempool-info dpdk0

mempool <ovs_mp_2030_0_262144>@0x7f35ff77ce40
...
```

## 总结

如果你读到这里，这可能意味着你遇到了 OvS-DPDK 的问题。很遗憾听到这个消息。希望在阅读完上面的指南后，你能够确定这个问题是否是由于内存不足引起的，并且你会知道如何修复它。

关于需要多少内存以及如何为多 NUMA(包括`dpdk-socket-mem`)配置 OvS-DPDK 的一些指导可以在本博客的 [OVS-DPDK:多少内存](https://developers.redhat.com/blog/2018/03/16/ovs-dpdk-hugepage-memory/)和 [OVS-DPDK:多 NUMA](https://developers.redhat.com/blog/2017/06/28/ovs-dpdk-parameters-dealing-with-multi-numa/) 文章中找到。

*Last updated: June 13, 2018*