# 使用 DPDK 实现 Open vSwitch 中 PMD 线程的自动负载平衡

> 原文：<https://developers.redhat.com/blog/2021/04/29/automatic-load-balancing-for-pmd-threads-in-open-vswitch-with-dpdk>

这篇文章是关于轮询模式驱动程序(PMD)在 [Open vSwitch 中的自动负载平衡功能，带有数据平面开发套件](https://docs.openvswitch.org/en/latest/intro/install/dpdk/)数据路径(OVS-DPDK)。该功能已经存在了一段时间，但我们最近在 Open vSwitch 2.15 中添加了新的用户参数。现在是一个很好的时间来看看这个功能在 OVS-DPDK。

当你读完这篇文章，你会明白 PMD 自动负载平衡功能解决的问题和操作它所需的用户参数。然后，你可以自己尝试一下。

## 带有 DPDK 的 Open vSwitch 中的 PMD 线程

在 OVS-DPDK 的环境中， *PMD 线程*，或*轮询模式驱动线程*，是一个在专用内核上 1:1 运行的线程，持续轮询端口以获取数据包。当它收到数据包时，它会根据数据包匹配的规则进行处理并转发。

每个 PMD 线程被分配一组来自连接到 OVS-DPDK 桥的不同端口的接收队列进行轮询。通常，端口是 DPDK 物理网络接口控制器(NIC)和`vhost-user`端口。

您可以选择 PMD 线程要使用多少内核以及使用哪些内核。增加内核数量可以让更多的处理周期用于数据包处理，从而提高 OVS-DPDK 的吞吐量。

例如，以下命令选择 PMD 线程要使用的内核 8 和内核 10:

```
$ ovs-vsctl set Open_vSwitch . other_config:pmd-cpu-mask=0x500
```

**注**:我们会大量引用一个 PMD 线程的*负载*；这是 PMD 线程用于在其核心上接收和处理数据包的处理周期数。

## PMD 线程和数据包处理负载

所有的接收队列都不会承载相同的流量，有些队列可能根本没有流量，因此有些 PMD 线程会比其他线程处理更多的数据包。换句话说，数据包处理负载在 PMD 线程之间是不平衡的。

在最坏的情况下，一些 PMD 线程可能会过载处理数据包，而其他 PMD 线程(可能是为了增加吞吐量而添加的)什么也不做。在这种情况下，一些内核没有帮助增加最大可能的吞吐量，因为它们的 PMD 线程没有有用的工作可做。

例如，在图 1 中，`dpdk0`和`dpdk1`端口有大量流量，内核 8 上的 PMD 线程过载处理这些数据包。`dpdk2`和`dpdk3`端口没有流量，内核 10 上的 PMD 线程处于空闲状态。

[![](img/718d852d8690a41db80665c26d2fdd30.png "1-ovs-dpdk-overload-sketch")](/sites/default/files/blog/2021/03/1-ovs-dpdk-overload-sketch.png)

Figure 1: The packet processing load is not balanced across PMD threads.

每当有重新配置时，比如在 OVS DPDK 中添加新端口，Rx 队列被重新分配给 PMD 线程。重新分配的主要目的是将需要最多处理的 Rx 队列分配给不同的 PMD 线程，以便尽可能多的 PMD 线程可以做有用的工作。

但是，如果在最后一次重新配置时负载未知，会发生什么呢？原因可能是刚刚添加了一个端口，或者流量尚未开始，或者负载随着时间的推移发生了变化，不再需要重新配置。这就是 PMD 自动负载平衡功能可以提供帮助的地方。

## PMD 自动负载平衡

如果启用了 PMD 自动负载平衡功能，它会定期运行。当满足某组条件时，它会触发将 Rx 队列重新分配给 PMD 线程，以改善 PMD 线程之间的负载平衡。

以下是它作为先决条件检查的主要条件:

*   如果任何当前 PMD 线程非常忙于处理数据包。
*   如果 PMD 线程负载之间的差异在重新分配后有可能改善。
*   如果距离上次重新分配还不算太早。

我们可以从命令行准确设置什么构成了*非常忙*、*改进*和*太快*。我们很快就会看到用户参数。

这个特性的好处是，只有当它检测到一个 PMD 线程当前非常忙*并且*估计在重新分配后方差会有所改善时，它才会进行重新分配。这有助于确保我们没有不必要的重新分配。

## PMD 自动负载平衡的用户参数

默认情况下，PMD 自动负载平衡功能处于禁用状态。您可以通过以下方式随时启用它:

```
$ ovs-vsctl --no-wait set open_vSwitch . other_config:pmd-auto-lb="true"
```

让我们来看看这个特性的用户参数。

### 负载阈值

*负载阈值*是在重新分配发生之前，一个 PMD 线程必须持续使用一分钟的处理周期的百分比。默认值为 95%。由于打开 vSwitch 2.15，您可以从命令行设置这一点。要将其设置为 70%，您需要输入以下内容:

```
$ ovs-vsctl --no-wait set open_vSwitch . other_config:pmd-auto-lb-load-threshold="70"
```

### 改良限度

*改进阈值*是在重新分配发生之前必须满足的 PMD 线程之间的负载差异的估计改进。为了计算估计的改进，进行重新分配的试运行，并且将估计的负载差异与当前差异进行比较。默认值为 25%。由于打开 vSwitch 2.15，您可以从命令行设置这一点。要将其设置为 50%，您需要输入以下内容:

```
$ ovs-vsctl --no-wait set open_vSwitch . other_config:pmd-auto-lb-improvement-threshold="50"
```

### 区间阈值

*间隔阈值*是可以触发两次重新分配之间的最小时间，以分钟为单位。这用于防止在流量模式可变的情况下触发频繁的重新分配。例如，您可能只想每 10 分钟或每几小时触发一次重新分配。默认值为一分钟；要将其设置为 10 分钟，您可以输入:

```
$ ovs-vsctl --no-wait set open_vSwitch . other_config:pmd-auto-lb-rebal-interval="10"
```

## PMD 自动负载平衡正在运行

让我们重新看一下图 1 中的简单例子，看看 PMD 自动负载平衡特性是如何工作的。如图 2 所示，我们从内核 8 上的一个 PMD 线程上的所有流量开始，内核 10 上的 PMD 线程上没有流量。

[![](img/718d852d8690a41db80665c26d2fdd30.png "1-ovs-dpdk-overload-sketch")](/sites/default/files/blog/2021/03/1-ovs-dpdk-overload-sketch.png)

Figure 2: One PMD thread on core 8 has all the traffic.

我们可以使用以下内容检查 Rx 队列分配:

```
$ ovs-appctl dpif-netdev/pmd-rxq-show
```

以下输出证实内核 8 上的 PMD 线程有两个活动 Rx 队列，而内核 10 上的 PMD 线程没有:

```
pmd thread numa_id 0 core_id 8:
  isolated : false
  port: dpdk0             queue-id:  0 (enabled)   pmd usage: 47 %
  port: dpdk1             queue-id:  0 (enabled)   pmd usage: 47 %
pmd thread numa_id 0 core_id 10:
  isolated : false
  port: dpdk2             queue-id:  0 (enabled)   pmd usage:  0 %
  port: dpdk3             queue-id:  0 (enabled)   pmd usage:  0 %

```

**注意**:这里显示的每个接收队列的使用百分比是紧密围绕各个接收队列的数据包处理进行测量的。它不包括任何 PMD 线程操作开销，也不包括在没有数据包的情况下轮询所花费的时间。使用`ovs-appctl dpif-netdev/pmd-stats-show`获得更多一般的 PMD 线程负载总量。

### 重新分配的情况

此时，流量生成器指示最大 11 Mpps 双向吞吐量。在这种情况下，PMD 自动负载平衡功能可以通过触发重新分配来提供帮助。重新分配后，应该使用内核 10 上的 PMD 线程。

让我们设置一些阈值并启用该功能:

```
$ ovs-vsctl set open_vSwitch . other_config:pmd-auto-lb-load-threshold="80"
$ ovs-vsctl set open_vSwitch . other_config:pmd-auto-lb-improvement-threshold="50"
$ ovs-vsctl set open_vSwitch . other_config:pmd-auto-lb-rebal-interval="1"
$ ovs-vsctl set open_vSwitch . other_config:pmd-auto-lb="true"

```

日志确认它已经启用，并且我们已经设置了值:

```
|dpif_netdev|INFO|PMD auto load balance is enabled interval 1 mins, pmd load threshold 80%, improvement threshold 50%
```

很快，我们看到一次试运行已经完成，并且已经请求重新配置数据路径，以将 Rx 队列重新分配给 PMD 线程:

```
|dpif_netdev|INFO|PMD auto lb dry run. requesting datapath reconfigure.
```

**注意**:您可以通过启用 debug 在`ovs-vswitchd.log`文件中获得更多关于操作和估计的详细信息；

```
$ ovs-appctl vlog/set dpif_netdev:file:dbg
```

### 检查结果

现在，如果我们重新检查统计数据，我们可以确认正在接收数据包的两个 Rx 队列被分配给了不同的 PMD 线程。

```
pmd thread numa_id 0 core_id 8:
  isolated : false
  port: dpdk0             queue-id:  0 (enabled)   pmd usage:  0 %
  port: dpdk2             queue-id:  0 (enabled)   pmd usage: 88 %
pmd thread numa_id 0 core_id 10:
  isolated : false
  port: dpdk1             queue-id:  0 (enabled)   pmd usage: 88 %
  port: dpdk3             queue-id:  0 (enabled)   pmd usage:  0 %

```

我们已经从只使用一个 PMD 线程的情况发展到同时使用两个 PMD 线程的情况，如图 3 所示。

[![The diagram shows a balanced traffic load.](img/e08edc49e8a8d34b47a2b764c3f0e2ac.png "4-ovs-dpdk-overunderload-horizontal-sketch")](/sites/default/files/blog/2021/03/4-ovs-dpdk-overunderload-horizontal-sketch.png)

Figure 3: Both PMD threads are fully utilized.

检查流量生成器，在这种情况下，吞吐量从 11 Mpps(在`dpdk0`和`dpdk1`端口上都有数据包丢弃)上升到 21 Mpps(没有任何丢弃)的最大配置速率。

**注意**:在上面的例子中，每个端口有一个 Rx 队列，但是在许多情况下，也可以通过使用每个端口多个 Rx 队列和 RSS 来更细粒度地分配工作负载。

## PMD 自动负载平衡的当前局限性

当然，所示示例是说明参数和用法的清晰案例。

重新分配代码将把最大负载的 Rx 队列分配给不同的 PMD 线程。它还将尝试确保所有 PMD 线程都有相同数量的已分配 Rx 队列。这是为了在针对当前流量负载的优化和为流量模式可能改变且没有 PMD 自动负载平衡的情况下提供弹性之间找到一个折衷。

在某些情况下，为 PMD 线程分配不同数量的 Rx 队列可能会进一步改善当前负载的平衡。这目前只能通过[手动锁定](https://docs.openvswitch.org/en/latest/topics/dpdk/pmd/#port-rx-queue-assigment-to-pmd-threads)来实现，并且是未来潜在的优化领域。

## 总结

在本文中，我们了解了具有 DPDK PMD 自动负载平衡功能的 Open vSwitch、它有助于解决的问题以及它的工作方式。该功能正在形成，我们已经在 Open vSwitch 2.15 中添加了新的用户参数。请随意尝试，并在 ovs-discuss@openvswitch.org 邮件列表上给出反馈。

*Last updated: October 14, 2022*