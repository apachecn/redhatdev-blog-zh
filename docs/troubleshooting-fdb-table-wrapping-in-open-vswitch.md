# Open vSwitch 中 FDB 表换行故障排除

> 原文：<https://developers.redhat.com/blog/2018/09/19/troubleshooting-fdb-table-wrapping-in-open-vswitch>

当大多数人使用正常规则，即使用 L2 学习，为虚拟网络部署 [Open vSwitch](https://developers.redhat.com/search?t=Open+vSwitch) 配置时，他们不会考虑配置 **F** 或**D**ATA**B**ase(**FDB**)的大小。

当使用基于硬件的交换机时，FDB 尺寸通常相当大，大 FDB 尺寸是一个关键卖点。然而，对于 Open vSwitch，默认 FDB 值相当小，例如，在 2.9 版和更低版本中，它只有 2K 个条目。从版本 2.10 开始，FDB 大小增加到 8K 条目。请注意，对于开放式 vSwitch，每个桥都有自己的 FDB 表，其大小可以单独配置。

这篇博客解释了配置太小的 FDB 表的影响，如何识别哪个网桥受到太小的 FDB 表的影响，以及如何适当地配置 FDB 表的大小。

## 太小的 FDB 表的影响

当 FDB 表已满并且需要添加新条目时，一个旧条目将被移除，以便为新条目 ¹ 腾出空间。这叫做 *FDB 裹胸*。如果从条目被删除的 MAC 地址接收到数据包，则删除另一个条目以腾出空间，并重新添加数据包的源 MAC 地址。

当网络中存在的 MAC 地址多于配置的 FDB 表的容量，并且所有 MAC 地址都被频繁发现时，表中会发生大量 ping/pong 操作。

ping/pong 越多，维护表所需的 CPU 资源就越多。此外，如果从被驱逐的 MAC 地址收到流量，流量会从所有端口溢出。

> ¹Open v switch 中删除旧条目的算法如下。在特定的网桥上，找到具有最多 FDB 条目的端口，并删除最旧的条目。

## open v switch–FDB 表太小的具体表现

除了 FDB 表更新之外，当 FDB 条目被移除时，Open vSwitch 还必须清理流表。这由打开的 vSwitch *重新验证器*线程来完成。因为这个流表清理需要相当多的 CPU 周期，所以 FDB 表回绕问题的第一个迹象可能是高的重新验证器线程利用率。以下示例显示了空闲系统中大约 83%的高重新验证器线程利用率(通过将 CPU%列中显示的百分比相加得出):

```
$ pidstat -t -p `pidof ovs-vswitchd` 1 | grep -E "UID|revalidator"
07:37:56 AM   UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
07:37:57 AM   995         -    188565    5.00    5.00    0.00   10.00     2  |__revalidator110
07:37:57 AM   995         -    188566    6.00    4.00    0.00   10.00     2  |__revalidator111
07:37:57 AM   995         -    188567    6.00    5.00    0.00   11.00     2  |__revalidator112
07:37:57 AM   995         -    188568    5.00    5.00    0.00   10.00     2  |__revalidator113
07:37:57 AM   995         -    188569    5.00    5.00    0.00   10.00     2  |__revalidator116
07:37:57 AM   995         -    188570    5.00    6.00    0.00   11.00     2  |__revalidator117
07:37:57 AM   995         -    188571    5.00    5.00    0.00   10.00     2  |__revalidator114
07:37:57 AM   995         -    188572    5.00    6.00    0.00   11.00     2  |__revalidator115
```

## FDB 包装问题的故障排除

让我们弄清楚高的重新验证器线程 CPU 使用率是否与请求清理的 FDB 有关。这可以通过检查覆盖率计数器来完成。下面显示了与重新验证程序运行原因相关的所有覆盖率计数器(其值大于零):

```
$ ovs-appctl coverage/show   | grep -E "rev_|Event coverage"
Event coverage, avg rate over last: 5 seconds, last minute, last hour,  hash=e4a796fd:
rev_reconfigure            0.0/sec     0.067/sec        0.0144/sec   total: 299
rev_flow_table             0.0/sec     0.000/sec        0.0003/sec   total: 2
rev_mac_learning          20.4/sec    18.167/sec       12.4039/sec   total: 44660
```

在上面的输出中，您可以看到`rev_mac_learning`每秒钟触发了大约 20 次重新验证过程。这个挺高的。理论上，由于正常的 FDB 老化过程，它仍然可能发生，尽管在这种特定情况下，最后一分钟/小时值应该更低。

然而，正常老化可以通过使用相同的覆盖率计数器来隔离:

```
$ ovs-appctl coverage/show   | grep -E "mac_learning_|Event"
Event coverage, avg rate over last: 5 seconds, last minute, last hour,  hash=086fdd98:
mac_learning_learned     1836.2/sec  1157.800/sec     1169.0800/sec   total: 7752613
mac_learning_expired       0.0/sec     0.000/sec        1.1378/sec   total: 4353
```

如你所见，有`mac_learning_learned`和`mac_learning_expired`计数器。在上面的输出中，您可以看到已经学习了很多新的 MAC 地址:大约每秒 1，836 个。对于大小为 2K 的 FDB 表来说，这是非常高的，并且表明我们正在替换 FDB 条目。

如果您运行的是 Open vSwitch v2.10 或更高版本，它具有额外的覆盖计数器:

```
$ ovs-appctl coverage/show   | grep -E "mac_learning_|Event"
Event coverage, avg rate over last: 5 seconds, last minute, last hour,  hash=0ddb1578:
mac_learning_learned       0.0/sec     0.000/sec       10.6514/sec   total: 38345
mac_learning_expired       0.0/sec     0.000/sec        2.2756/sec   total: 8192
mac_learning_evicted       0.0/sec     0.000/sec        8.3758/sec   total: 30153
mac_learning_moved         0.0/sec     0.000/sec        0.0000/sec   total: 1
```

对上述内容的解释:

*   `mac_learning_learned`:显示学习的 MAC 条目总数
*   `mac_learning_expired`:显示过期 MAC 条目的总数
*   `mac_learning_evicted`:显示被逐出的 MAC 条目的总数，即由于表已满而移出的条目
*   `mac_learning_moved`:显示“端口移动”MAC 条目的总数，即 MAC 地址移动到不同端口的条目

现在，您如何确定哪座桥有 FDB 缠绕问题？对于 v2.9 和更早的版本，这是一个手动的过程，使用命令`ovs-appctl fdb/show`多次转储 FDB 表，并比较条目。

对于 v2.10 和更高版本，引入了一个新命令`ovs-appctl fdb/stats-show`，它显示了每个网桥的所有上述统计信息:

```
$ ovs-appctl fdb/stats-show ovs0
Statistics for bridge "ovs0":
  Current/maximum MAC entries in the table: 8192/8192
  Total number of learned MAC entries     : 52779
  Total number of expired MAC entries     : 8192
  Total number of evicted MAC entries     : 36395
  Total number of port moved MAC entries  : 1
```

**注意**:统计数据可以用`ovs-appctl fdb/stats-clear`命令清除，例如，得到每秒的速率:

```
$ ovs-appctl fdb/stats-clear ovs0; sleep 1; ovs-appctl fdb/stats-show ovs0
statistics successfully cleared
Statistics for bridge "ovs0":
  Current/maximum MAC entries in the table: 8192/8192
  Total number of learned MAC entries     : 1902
  Total number of expired MAC entries     : 0
  Total number of evicted MAC entries     : 1902
  Total number of port moved MAC entries  : 0
```

## 固定 FDB 表的大小

使用 Open vSwitch，您可以轻松调整 FDB 表的大小，并且可以根据网桥进行配置。执行此操作的命令如下:

```
ovs-vsctl set bridge <bridge> other-config:mac-table-size=<size>
```

更改配置时，请注意以下几点:

*   FDB 条目的数量可以从 10 到 1，000，000。
*   该配置立即生效。
*   当前条目不会从表中刷新。
*   如果配置的数量小于表中当前条目的数量，则最旧的条目将被淘汰。您可以在*过期 MAC 条目*统计中看到这一点。

为什么不把默认值改成 100 万，不用再担心这个了？资源消耗:表中的每个条目都分配内存。虽然 Open vSwitch 仅在条目被使用时分配内存，但是将默认值更改为过高的值可能会成为一个问题，例如，当有人进行 [MAC 泛洪攻击](https://en.wikipedia.org/wiki/MAC_flooding)时。

那么配置什么样的大小才是正确的呢？这很难说，取决于您的用例。根据经验，您应该将您的表配置得比网桥上活动 MAC 地址的平均数量稍大一些。

## 查看 FDB 环绕效果的简单脚本

如果你想试验计数器，下面这个来自 *Jiri Benc* 的复制脚本可以让你复制 FDB 环绕的效果。

创建一个开放式虚拟交换机桥:

```
$ ovs-vsctl add-br ovs0
$ ip link set ovs0 up
```

创建复制脚本:

```
$ cat > ~/reproducer.py <<EOF
#!/usr/bin/python
from scapy.all import *

data = [(str("00" + str(RandMAC())[2:]), str(RandIP())) for i in range(int(sys.argv[1]))]

s = conf.L2socket(iface="ovs0")
while True:
    for mac, ip in data:
        p = Ether(src=mac, dst=mac)/IP(src=ip, dst=ip)
        s.send(p)
EOF

$ chmod +x ~/reproducer.py
```

**注意**:重现器 Python 脚本需要安装 [Scapy](https://scapy.readthedocs.io/en/latest/installation.html) 。

启动再现器:

```
$ ./reproducer.py 10000
```

现在，您可以使用前面故障排除部分中的计数器命令来查看 FDB 表换行信息，然后适当地设置 FDB 的大小。

## 其他 Open vSwitch 和 Open Virtual Network 资源

红帽的很多产品，比如红帽 OpenStack 平台和红帽虚拟化，现在都在使用 Open vSwitch 的子项目 Open Virtual Network (OVN)。 [Red Hat OpenShift 集装箱平台](https://developers.redhat.com/products/openshift/overview/)即将使用 OVN。Red Hat 开发人员博客上的一些其他虚拟网络文章:

*   [开放虚拟网络中的动态 IP 地址管理(OVN):第一部分](https://developers.redhat.com/blog/2018/09/03/ovn-dynamic-ip-address-management/)
*   [红帽企业版 Linux 中的非根开放 v switch](https://developers.redhat.com/blog/2018/03/23/non-root-open-vswitch-rhel/)
*   [使用 Open vSwitch DPDK 调试内存问题](https://developers.redhat.com/blog/2018/06/14/debugging-ovs-dpdk-memory-issues/)
*   [故障排除打开 vSwitch DPDK PMD 线程核心关联](https://developers.redhat.com/blog/2018/06/20/troubleshooting-open-vswitch-dpdk-pmd-thread-core-affinity/)
*   [打开 vSwitch-DPDK:多少 Hugepage 内存？](https://developers.redhat.com/blog/2018/03/16/ovs-dpdk-hugepage-memory/)

*Last updated: January 17, 2022*