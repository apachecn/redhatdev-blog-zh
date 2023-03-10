# 在 open vswitch 中调试 vhost 用户 tx 连接

> 原文：<https://developers.redhat.com/blog/2020/05/29/debugging-vhost-user-tx-contention-in-open-vswitch>

要理解如何使用 [Open vSwitch (OVS)](https://github.com/openvswitch/ovs) 周期并不容易，尤其是因为各种参数和配置选项会影响 OVS 的行为。Open vSwitch 社区的成员正在积极研究 Open vSwitch 中导致数据包丢失的原因。到目前为止所做的努力包括[为 vHost TX 重试](https://github.com/openvswitch/ovs/commit/c161357d5d96)添加自定义统计数据、[跟踪 vHost TX 竞争](https://github.com/openvswitch/ovs/commit/9ff24b9c9323)，以及[添加覆盖计数器来计算 vHost IRQ](https://github.com/openvswitch/ovs/commit/3d56e4ac445d)。我们对使用数据平面开发套件(DPDK)进行快速 I/O 的用户空间数据路径特别感兴趣。

添加这些统计数据是一项持续的工作，我们不会涵盖所有的方面。在某些情况下，统计数据让人怀疑是什么导致了一种行为。

在本文中，我将介绍我们添加的一个新计数器，以了解更多关于 vHost 传输路径中的争用。我还将向您展示如何使用新的计数器`perf`，并且我将讨论我们正在进行的下一步工作。

## 再现争用的测试环境

在本节中，我们将设置一个测试环境来重现 vHost 传输路径中的争用。我们的参考系统运行的是带有`openvswitch2.11-2.11.0-35.el7fdp.x86_64`的 Red Hat Enterprise Linux (RHEL) 7.7。如果您的环境已经设置好并正在运行，您可以跳过这一部分。

### 配置 OVS

假设您的系统中已经运行了 Red Hat Enterprise Linux 7.7，您可以用一个网桥配置 OVS，该网桥有两个插入式物理端口和两个`vhost-user-client`端口。

物理端口连接到一个 [TRex 现实流量生成器](https://trex-tgn.cisco.com/)。流量生成器将向网桥发送单向数据包流，传递到`vhost-user-clients`端口。我们精心制作了这些数据包，将流量发送到 OVS 端第一个物理端口的两个队列。`vhost-user-clients`端口连接到一个虚拟机(VM ),该虚拟机在`io forward`模式下使用`testpmd`发回数据包。

如果您需要有关设置 TRex 流量生成器、配置运行 OVS 的主机或该设置的其他方面的更多详细信息，请参见 Eelco Chaudron 的 *[自动打开 vSwitch PVP 测试](https://developers.redhat.com/blog/2017/09/28/automated-open-vswitch-pvp-testing/)* 简介。

### 配置网桥和主机

此处 ASCII 图中的设置与上面链接文章中所示的物理接口到虚拟接口再到物理接口(PVP)的设置略有不同。

```
+------+   +-----+   +---------+
|      |   |     |   |         |
|     0+---+1   4+---+0        |
| tgen |   | ovs |   | testpmd |
|     1+---+2   5+---+1        |
|      |   |     |   |         |
+------+   +-----+   +---------+

```

在主机上配置网桥，如下所示:

```
# ovs-vsctl set Open_vSwitch . other_config:dpdk-init=true
# ovs-vsctl set Open_vSwitch . other_config:pmd-cpu-mask=0x00008002
# ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
# ovs-vsctl add-port br0 dpdk0 -- \
    set Interface dpdk0 type=dpdk -- \
    set Interface dpdk0 options:dpdk-devargs=0000:01:00.0 -- \
    set Interface dpdk0 ofport_request=1 -- \
    set Interface dpdk0 options:n_rxq=2
# ovs-vsctl add-port br0 dpdk1 -- \
    set Interface dpdk1 type=dpdk -- \
    set Interface dpdk1 options:dpdk-devargs=0000:01:00.1 -- \
    set Interface dpdk1 ofport_request=2 -- \
    set Interface dpdk1 options:n_rxq=2
# ovs-vsctl add-port br0 vhost0 -- \
    set Interface vhost0 type=dpdkvhostuserclient -- \
    set Interface vhost0 options:vhost-server-path="/tmp/vhost-sock0" -- \
    set Interface vhost0 ofport_request=4
# ovs-vsctl add-port br0 vhost1 -- \
    set Interface vhost1 type=dpdkvhostuserclient -- \
    set Interface vhost1 options:vhost-server-path="/tmp/vhost-sock1" -- \
    set Interface vhost1 ofport_request=5

```

检查轮询配置:

```
# ovs-appctl dpif-netdev/pmd-rxq-show
pmd thread numa_id 0 core_id 1:
  isolated : false
  port: dpdk0             queue-id:  0  pmd usage: NOT AVAIL
  port: dpdk1             queue-id:  1  pmd usage: NOT AVAIL
  port: vhost0            queue-id:  0  pmd usage: NOT AVAIL
pmd thread numa_id 0 core_id 15:
  isolated : false
  port: dpdk0             queue-id:  1  pmd usage: NOT AVAIL
  port: dpdk1             queue-id:  0  pmd usage: NOT AVAIL
  port: vhost1            queue-id:  0  pmd usage: NOT AVAIL

```

我们可以让这个 OVS 通过一个`NORMAL`动作进行桥接，在这种情况下，它的行为就像一个标准的交换机在它的端口上学习媒体访问控制(MAC)地址。为了简化设置，让我们只为基本映射编写一些 OpenFlow 规则:

*   物理端口`dpdk0`上的接收将数据包推送到`vhost0`。
*   虚拟端口`vhost0`上的接收将数据包推送到`dpdk0`。
*   物理端口`dpdk1`上的接收将数据包推送到`vhost1`。
*   虚拟端口`vhost1`上的接收将数据包推送到`dpdk1`。

这是映射:

```
# ovs-ofctl del-flows br0
# ovs-ofctl add-flow br0 in_port=1,actions=4
# ovs-ofctl add-flow br0 in_port=4,actions=1
# ovs-ofctl add-flow br0 in_port=2,actions=5
# ovs-ofctl add-flow br0 in_port=5,actions=2

```

这就完成了测试环境。

## 捕获 vHost TX 争用

现在让我们来看看 OVS 的新承保柜台:

```
# ovs-appctl coverage/show |grep vhost
vhost_tx_contention      39082.8/sec 11553.017/sec      192.5503/sec   total: 758359

```

### 添加性能探测器

实际上，计数器没有考虑哪些核心会受到争用的影响。我们可以利用`perf`在不阻止 OVS 的情况下获取更多信息。只需在发生争用的分支中添加一个探测器:

```
# perf probe -x $(which ovs-vswitchd) 'netdev_dpdk_vhost_tx_lock=__netdev_dpdk_vhost_send:22 netdev->name:string qid'
Added new event:
  probe_ovs:netdev_dpdk_vhost_tx_lock (on __netdev_dpdk_vhost_send:22 in /usr/sbin/ovs-vswitchd with name=netdev->name:string qid)

```

现在你可以在所有的`perf`工具中使用计数器。

### 在性能中使用覆盖率计数器

这里，我们要求`perf`记录一个特定的事件:

```
# perf record -e probe_ovs:netdev_dpdk_vhost_tx_lock -aR sleep 1
[ perf record: Woken up 15 times to write data ]
[ perf record: Captured and wrote 3.938 MB perf.data (44059 samples) ]

```

我们还可以对本次`perf`会议做一个报告:

```
# perf report -F +pid --stdio
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 44K of event 'probe_ovs:netdev_dpdk_vhost_tx_lock'
# Event count (approx.): 44059
#
# Overhead      Pid:Command  Trace output
# ........  ...............  ..................................
#
    61.30%    33003:pmd60    (55ef4abe5494) name="vhost0" qid=0
    38.70%    33006:pmd61    (55ef4abe5494) name="vhost0" qid=0

#
# (Tip: For a higher level overview, try: perf report --sort comm,dso)
#

```

新的覆盖率计数器使得解释竞争更加容易。我们可以看到竞争发生在`pmd60`(在内核 1 上，通过查看 OVS 日志)和`pmd61`(在内核 15 上)之间。两个`pmd`线程都试图在`vhost0`端口的零队列上发送数据包。

## 结论

使用`perf`来调试争用是很有趣的，它在这种情况下是有效的，因为我们试图捕捉错误或缓慢路径上的事件。但是`perf`涉及到对性能有明显影响的上下文切换。如果不考虑性能影响，我们就无法使用它。

即使开发人员可以通过阅读代码源来进行调查，支持或运营团队也会更喜欢更高级的工具或跟踪。DPDK 社区已经[启动了一个工作组](http://inbox.dpdk.org/dev/20200318190241.3150971-1-jerinj@marvell.com/)，在 DPDK 基础设施代码的关键位置设置影响最小的跟踪。我们还远没有达到`perf`的富裕程度，但这很可能是明年部分社区关注的焦点。

*Last updated: June 26, 2020*