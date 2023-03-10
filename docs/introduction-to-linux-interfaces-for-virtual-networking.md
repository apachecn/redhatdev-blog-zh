# 虚拟网络的 Linux 接口介绍

> 原文：<https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking>

[Linux](https://developers.redhat.com/topics/linux/) 拥有丰富的虚拟网络功能，作为托管虚拟机和[容器](https://developers.redhat.com/blog/category/containers/)以及云环境的基础。在这篇文章中，我将简要介绍所有常用的虚拟网络接口类型。没有代码分析，只有对接口及其在 Linux 上的用法的简单介绍。任何有网络背景的人都可能对这篇博文感兴趣。使用命令`ip link help`可以获得接口列表。

这篇文章涵盖了以下常用的接口和一些容易混淆的接口:

*   [桥](#bridge)
*   [粘合界面](#bonded)
*   [团队设备](#team)
*   [【虚拟局域网】](#vlan)[](#vxlan)
*   [VXLAN(虚拟可扩展局域网)](#vxlan)
*   [MACVLAN](#macvlan)
*   [IPVLAN](#ipvlan)
*   [MACVTAP/IPVTAP](#macvtap)
*   [MACsec(媒体访问控制安全)](#macsec)
*   [虚拟以太网](#veth)
*   [VCAN(虚拟能)](#vcan)
*   [VXCAN(虚拟 CAN 隧道)](#vxcan)
*   [IPOIB(IP-over-InfiniBand)](#ipoib)
*   [NLMON(网络链接监视器)](#nlmon)
*   [虚拟接口](#dummy)
*   [IFB(中间功能块)](#ifb)
*   [网域名称](#netdevsim)

看完这篇文章，你就会知道这些接口是什么，它们之间有什么区别，什么时候使用，如何创建。关于隧道等其他接口，请参见[Linux 虚拟接口介绍:隧道](https://developers.redhat.com/blog/2019/05/17/an-introduction-to-linux-virtual-interfaces-tunnels/)

## 桥

Linux 网桥的行为类似于网络交换机。它在连接到它的接口之间转发数据包。它通常用于在路由器、网关或主机上的虚拟机和网络命名空间之间转发数据包。它还支持 STP、VLAN 过滤器和多播监听。

[![Bridge diagram](img/347a78e0902e3ab32f3f258ac5a19f88.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/bridge.png)

当您希望在虚拟机、容器和主机之间建立通信通道时，请使用网桥。

以下是创建桥的方法:

```
# ip link add br0 type bridge
# ip link set eth0 master br0
# ip link set tap1 master br0
# ip link set tap2 master br0
# ip link set veth1 master br0

```

这将创建一个名为`br0`的桥接设备，并将两个 TAP 设备(`tap1`、`tap2`)、一个 VETH 设备(`veth1`)和一个物理设备(`eth0`)设置为其从属设备，如上图所示。

## 粘合界面

Linux 绑定驱动程序提供了一种将多个网络接口聚合成一个逻辑“绑定”接口的方法。绑定接口的行为取决于模式；一般来说，模式提供热备用或负载平衡服务。

[![Bonded interface](img/6643928560d579674d08e999ba3d6ac2.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/bond.png)

当您想要提高链接速度或在服务器上进行故障转移时，请使用绑定接口。

以下是创建绑定接口的方法:

```
ip link add bond1 type bond miimon 100 mode active-backup
ip link set eth0 master bond1
ip link set eth1 master bond1

```

这将创建一个名为`bond1`的绑定接口，模式为主动备份。对于其他模式，请参见[内核文档](https://www.kernel.org/doc/Documentation/networking/bonding.txt)。

## 团队设备

类似于绑定接口，组设备的目的是提供一种机制，在 L2 层将多个 NIC(端口)组合成一个逻辑 NIC(teamdev)。

[![Team device](img/29e71f2cb7490e75f312b17689805c30.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/team.png)

要意识到的主要事情是，组设备并不试图复制或模仿绑定的接口。它所做的是使用不同的方法解决相同的问题，例如使用无锁(RCU) TX/RX 路径和模块化设计。

但是绑定接口和团队之间也有一些功能上的区别。例如，一个团队支持 LACP 负载平衡、NS/NA (IPV6)链路监控、D-Bus 接口等。，这在成键中是不存在的。关于结合和团队之间区别的更多细节，参见[结合与团队特征](https://github.com/jpirko/libteam/wiki/Bonding-vs.-Team-features)。

当你想使用一些绑定不能提供的特性时，使用团队。

以下是创建团队的方法:

```
# teamd -o -n -U -d -t team0 -c '{"runner": {"name": "activebackup"},"link_watch": {"name": "ethtool"}}'
# ip link set eth0 down
# ip link set eth1 down
# teamdctl team0 port add eth0
# teamdctl team0 port add eth1

```

这创建了一个名为`team0`的带有模式`active-backup`的团队接口，并添加了`eth0`和`eth1`作为`team0`的子接口。

Linux 最近增加了一个名为 [net_failover](https://www.kernel.org/doc/html/latest/networking/net_failover.html) 的新驱动。它是另一个用于虚拟化的故障转移主网络设备，并管理一个主要( [passthru/VF【虚拟功能】](https://wiki.libvirt.org/page/Networking#PCI_Passthrough_of_host_network_devices)设备)从网络设备和一个备用(原始准虚拟接口)从网络设备。

[![Net_failover driver](img/7c01e063200609a6c52794bcba8dac1d.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/net_failover.png)

## VLAN

VLAN，也称为虚拟局域网，通过向网络数据包添加标签来分隔广播域。VLANs 允许网络管理员将同一台交换机下或不同交换机之间的主机分组。

VLAN 标题看起来像:

[![VLAN header](img/9aece27d60e68fd1f0bfbbbf2c597f13.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/vlan_01.png)

当您希望在虚拟机、命名空间或主机中分隔子网时，请使用 VLAN。

以下是创建 VLAN 的方法:

```
# ip link add link eth0 name eth0.2 type vlan id 2
# ip link add link eth0 name eth0.3 type vlan id 3

```

这将添加名为`eth0.2`的 VLAN 2 和名为`eth0.3`的 VLAN 3。拓扑如下所示:

[![VLAN topology](img/8980913443a894ef3c2639f723201485.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/vlan.png)

***注意*** :配置 VLAN 时，需要确保连接到主机的交换机能够处理 VLAN 标签，例如，通过将交换机端口设置为中继模式。

## 切换

VXLAN(虚拟可扩展局域网)是一种隧道协议，旨在解决 IEEE 802.1q 中 VLAN id(4096)有限的问题，由 [IETF RFC 7348](https://tools.ietf.org/html/rfc7348) 描述。

凭借 24 位段 ID，即 VXLAN 网络标识符(VNI)，VXLAN 最多可支持 2^24 (16，777，216)个虚拟 LAN，这是 VLAN 容量的 4，096 倍。

VXLAN 将带有 VXLAN 报头的第 2 层帧封装到 UDP-IP 数据包中，如下所示:

[![VXLAN encapsulates Layer 2 frames with a VXLAN header into a UDP-IP packet](img/6f09905f4547601553b6074ef24477d6.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/vxlan_01.png)

VXLAN 通常部署在数据中心的虚拟化主机上，这些主机可能分布在多个机架上。

[![Typical VXLAN deployment](img/9764a21fa7280677c7558c69f9959af5.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/vxlan.png)

VXLAN 的使用方法如下:

```
# ip link add vx0 type vxlan id 100 local 1.1.1.1 remote 2.2.2.2 dev eth0 dstport 4789

```

作为参考，你可以阅读 [VXLAN 内核文档](https://www.kernel.org/doc/Documentation/networking/vxlan.txt)或者[本 VXLAN 简介](https://vincent.bernat.ch/en/blog/2017-vxlan-linux)。

## MACVLAN

使用 VLAN，您可以在单个接口的基础上创建多个接口，并根据 VLAN 标签过滤包。使用 MACVLAN，您可以在一个接口上创建多个具有不同第 2 层(即以太网 MAC)地址的接口。

在 MACVLAN 之前，如果您想要从虚拟机或命名空间连接到物理网络，您需要创建 TAP/VETH 设备，并将一端连接到网桥，同时将物理接口连接到主机上的网桥，如下所示。

[![Configuration before MACVLAN](img/518a77c51db3b4fe573ac121df47e095.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/br_ns.png)

现在，使用 MACVLAN，您可以将与 MACVLAN 相关联的物理接口直接绑定到名称空间，而不需要桥。

[![Configuration with MACVLAN](img/54af4c20e03c90ee43fe5d2dbae7ae3a.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macvlan.png)

有五种 MACVLAN 类型:

1.Private:不允许同一物理接口上的 MACVLAN 实例之间进行通信，即使外部交换机支持发夹模式。

[![Private MACVLAN configuration](img/723f674421292726586af53be1c409ef.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macvlan_01.png)

2.VEPA:同一物理接口上从一个 MACVLAN 实例到另一个实例的数据通过物理接口传输。要么连接的交换机需要支持发夹模式，要么必须有一个 TCP/IP 路由器转发数据包以允许通信。

[![VEPA MACVLAN configuration](img/b65203c40dd8e6fd1aacdf06961e335e.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macvlan_02.png)

3.桥:所有端点通过物理接口用一个简单的桥直接相互连接。

[![Bridge MACVLAN configuration](img/33a02f5ae0aec73e9f9bd93968a603ac.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macvlan_03.png)

4.Passthru:允许单个虚拟机直接连接到物理接口。

[![Passthru MACVLAN configuration](img/90a02d501fa61fbfb36e753203bcc452.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macvlan_04.png)

5.源:源模式用于根据允许的源 MAC 地址列表过滤流量，以创建基于 MAC 的 VLAN 关联。请参见[提交消息](https://git.kernel.org/pub/scm/linux/kernel/git/davem/net.git/commit/?id=79cf79abce71)。

根据不同的需要选择类型。桥接模式是最常用的。

当您想要从容器直接连接到物理网络时，请使用 MACVLAN。

以下是设置 MACVLAN 的方法:

```
# ip link add macvlan1 link eth0 type macvlan mode bridge
# ip link add macvlan2 link eth0 type macvlan mode bridge
# ip netns add net1
# ip netns add net2
# ip link set macvlan1 netns net1
# ip link set macvlan2 netns net2

```

这会在桥接模式下创建两个新的 MACVLAN 设备，并将这两个设备分配给两个不同的名称空间。

## IPVLAN

IPVLAN 类似于 MACVLAN，不同之处在于端点具有相同的 MAC 地址。

[![IPVLAN configuration](img/00b9ff6712b963f38bbe25395b8c9751.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/ipvlan.png)

IPVLAN 支持 L2 和 L3 模式。IPVLAN L2 模式的作用类似于网桥模式下的 MACVLAN。父接口看起来像一个网桥或交换机。

[![IPVLAN L2 mode](img/d0adccee9e2a7eb8c8708d223378c2e5.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/ipvlan_01.png)

在 IPVLAN L3 模式中，父接口的作用类似于路由器，数据包在端点之间路由，这提供了更好的可扩展性。

[![IPVLAN L3 mode](img/bdd2fead629784a21d894221c9af104c.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/ipvlan_02.png)

关于何时使用 IPVLAN， [IPVLAN 内核文档](https://www.kernel.org/doc/Documentation/networking/ipvlan.txt)称 MACVLAN 和 IPVLAN“在许多方面非常相似，具体的用例可以很好地定义选择哪种设备。如果以下情况之一定义了您的用例，那么您可以选择使用 ipvlan -
(a)连接到外部交换机/路由器的 Linux 主机配置了策略，每个端口只允许一个 mac。
(b)在主设备上创建的虚拟设备数量超过 mac 容量，并使 NIC 处于混杂模式，性能下降是一个问题。
(c)如果从属设备将被放入敌对/不可信的网络名称空间，其中从属设备上的 L2 可能被改变/误用

以下是设置 IPVLAN 实例的方法:

```
# ip netns add ns0
# ip link add name ipvl0 link eth0 type ipvlan mode l2
# ip link set dev ipvl0 netns ns0

```

这将创建一个名为`ipvl0`的 IPVLAN 设备，模式为 L2，分配给命名空间`ns0`。

## MACVTAP/IPVTAP

MACVTAP/IPVTAP 是一种新的设备驱动程序，旨在简化虚拟化桥接网络。当在物理接口上创建 MACVTAP/IPVTAP 实例时，内核还会创建一个字符设备/dev/tapX，就像使用 [TUN/TAP](https://en.wikipedia.org/wiki/TUN/TAP) 设备一样，KVM/QEMU 可以直接使用它。

有了 MACVTAP/IPVTAP，您可以用一个模块代替 TUN/TAP 和 bridge 驱动器的组合:

[![MACVTAP/IPVTAP instance](img/3bc3d2d3396557c6773cfcb1b8bddef3.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macvtap.png)

通常，MACVLAN/IPVLAN 用于使来宾和主机直接显示在主机所连接的交换机上。MACVTAP 和 IPVTAP 之间的区别与 MACVLAN/IPVLAN 相同。

下面是如何创建 MACVTAP 实例:

```
# ip link add link eth0 name macvtap0 type macvtap

```

## MACsec

MACsec(媒体访问控制安全)是用于有线以太网局域网安全的 IEEE 标准。与 IPsec 类似，作为第 2 层规范，MACsec 不仅可以保护 IP 流量，还可以保护 ARP、邻居发现和 DHCP。MACsec 头看起来像这样:

[![MACsec header](img/8d55a2e303d2c4f7f6febf4d033b9e58.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macsec_01.png)

MACsec 的主要用例是保护标准 LAN 上的所有消息，包括 ARP、NS 和 DHCP 消息。

[![MACsec configuration](img/0a98f4e3a38621a44aa3e00a111a1e53.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/macsec.png)

以下是设置 MACsec 配置的方法:

```
# ip link add macsec0 link eth1 type macsec

```

***注*** :这只在接口`eth1`上增加了一个名为`macsec0`的 MACsec 设备。有关更详细的配置，请参见 Sabrina Dubroca 的本 [MACsec 简介中的“配置示例”部分。](https://developers.redhat.com/blog/2016/10/14/macsec-a-different-solution-to-encrypt-network-traffic/)

## VETH

VETH(虚拟以太网)设备是一个本地以太网隧道。设备成对创建，如下图所示。

在设备对中的一台设备上传输的数据包会立即被另一台设备接收。当任一设备出现故障时，该设备对的链路状态为故障。

[![Pair of VETH devices](img/89dd5539a3634cdcf2427d44512b999b.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/veth.png)

当名称空间需要与主主机名称空间通信或者在名称空间之间通信时，使用 VETH 配置。

下面是设置 VETH 配置的方法:

```
# ip netns add net1
# ip netns add net2
# ip link add veth1 netns net1 type veth peer name veth2 netns net2

```

这创建了两个名称空间，`net1`和`net2`，以及一对 VETH 设备，它将`veth1`分配给名称空间`net1`，将`veth2`分配给名称空间`net2`。这两个名称空间用 VETH 对连接。分配一对 IP 地址，就可以在两个名称空间之间 ping 和通信。

## VCAN

与网络回送设备类似，VCAN(虚拟 CAN)驱动程序提供了虚拟本地 CAN(控制器局域网)接口，因此用户可以通过 VCAN 接口发送/接收 CAN 消息。现在 CAN 主要用于汽车领域。

更多 CAN 协议信息，请参考[内核 CAN 文档](https://www.kernel.org/doc/Documentation/networking/can.txt)。

当您想要在本地主机上测试 CAN 协议实现时，请使用 VCAN。

以下是创建 VCAN 的方法:

```
# ip link add dev vcan1 type vcan

```

## VXCAN

与 VETH 驱动程序类似，VXCAN(虚拟 CAN 隧道)在两个 VCAN 网络设备之间实现本地 CAN 流量隧道。创建 VXCAN 实例时，两个 VXCAN 设备是成对创建的。当一端收到数据包时，数据包会出现在设备对上，反之亦然。VXCAN 可用于跨名称空间的通信。

当您想要跨名称空间发送 CAN 消息时，请使用 VXCAN 配置。

以下是设置 VXCAN 实例的方法:

```
# ip netns add net1
# ip netns add net2
# ip link add vxcan1 netns net1 type vxcan peer name vxcan2 netns net2

```

***注意*** :红帽企业版 Linux 还不支持 VXCAN。

## IPOIB

IPOIB 设备支持无限带宽 IP 协议。这通过 InfiniBand (IB)传输 IP 数据包，因此您可以将 IB 设备用作快速网卡。

IPoIB 驱动程序支持两种操作模式:数据报和连接。在数据报模式下，使用 IB UD(不可靠数据报)传输。在连接模式下，使用 IB RC(可靠连接)传输。连接模式利用了 IB 传输的连接特性，允许 MTU 的最大 IP 数据包大小为 64K。

更多详情，请参见 [IPOIB 内核文档](https://www.kernel.org/doc/Documentation/infiniband/ipoib.txt)。

当您有 IB 设备并且想要通过 IP 与远程主机通信时，请使用 IPOIB 设备。

以下是创建 IPOIB 设备的方法:

```
# ip link add ib0 name ipoib0 type ipoib pkey IB_PKEY mode connected

```

## NLMON

NLMON 是一个 Netlink 监视器设备。

当您想要监视系统 Netlink 消息时，请使用 NLMON 设备。

以下是创建 NLMON 设备的方法:

```
# ip link add nlmon0 type nlmon
# ip link set nlmon0 up
# tcpdump -i nlmon0 -w nlmsg.pcap

```

这将创建一个名为`nlmon0`的 NLMON 设备并对其进行设置。使用数据包嗅探器(例如，`tcpdump`)捕获 Netlink 消息。Wireshark 的最新版本具有解码 Netlink 消息的功能。

## 虚拟界面

虚拟接口完全是虚拟的，例如环回接口。虚拟接口的目的是提供一种设备来路由数据包，而无需实际传输它们。

使用虚拟接口使非活动 SLIP(串行线路互联网协议)地址看起来像本地程序的真实地址。如今，虚拟接口主要用于测试和调试。

下面是创建虚拟接口的方法:

```
# ip link add dummy1 type dummy
# ip addr add 1.1.1.1/24 dev dummy1
# ip link set dummy1 up

```

## IFB

IFB(中间功能块)驱动程序提供了一种设备，允许集中来自几个源的流量，并对传入的流量进行整形，而不是丢弃它。

当您希望对传入流量进行排队和整形时，请使用 IFB 接口。

以下是创建 IFB 界面的方法:

```
# ip link add ifb0 type ifb
# ip link set ifb0 up
# tc qdisc add dev ifb0 root sfq
# tc qdisc add dev eth0 handle ffff: ingress
# tc filter add dev eth0 parent ffff: u32 match u32 0 0 action mirred egress redirect dev ifb0

```

这创建了一个名为`ifb0`的 IFB 设备，并用 SFQ(随机公平排队)替换了根 qdisc 调度程序，这是一个无类排队调度程序。然后，它在`eth0`上添加一个入口 qdisc 调度程序，并将所有入口流量重定向到`ifb0`。

更多 IFB qdisc 的使用案例，请参考这个[IFB 的 Linux 基金会 wiki](https://wiki.linuxfoundation.org/networking/ifb)。

## 额外资源

*   [Red Hat 开发者博客上的虚拟网络文章](https://developers.redhat.com/search?t=Virtual+networking+articles)
*   [开放虚拟网络中的动态 IP 地址管理(OVN)](https://developers.redhat.com/blog/2018/09/03/ovn-dynamic-ip-address-management/)
*   [红帽企业版 Linux 中的非根开放 v switch](https://developers.redhat.com/blog/2018/03/23/non-root-open-vswitch-rhel/)
*   [打开 Red hat 开发者博客上的 vSwitch 文章](https://developers.redhat.com/search?t=Open+vSwitch)

## netdevsim 接口

netdevsim 是一个模拟的网络设备，用于测试各种网络 API。此时，它特别关注测试硬件
卸载、tc/XDP BPF 和 SR-IOV。

可以按如下方式创建 netdevsim 设备

```
# ip link add dev sim0 type netdevsim
# ip link set dev sim0 up

```

要启用 tc 卸载:

```
# ethtool -K sim0 hw-tc-offload on

```

要加载 XDP BPF 或 tc BPF 程序，请执行以下操作:

```
# ip link set dev sim0 xdpoffload obj prog.o

```

要为 SR-IOV 测试添加 VFs:

```
# echo 3 > /sys/class/net/sim0/device/sriov_numvfs
# ip link set sim0 vf 0 mac 

```

要更改 vf 编号，您需要首先完全禁用它们:

```
# echo 0 > /sys/class/net/sim0/device/sriov_numvfs
# echo 5 > /sys/class/net/sim0/device/sriov_numvfs

```

默认情况下，netdevsim 不在 RHEL 编译

*Last updated: October 12, 2022*