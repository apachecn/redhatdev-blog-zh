# Linux 虚拟接口简介:隧道

> 原文：<https://developers.redhat.com/blog/2019/05/17/an-introduction-to-linux-virtual-interfaces-tunnels>

Linux 已经支持多种类型的隧道，但是新用户可能会对它们的差异感到困惑，不确定哪一种最适合给定的用例。在本文中，我将简要介绍 Linux 内核中常用的隧道接口。没有代码分析，只有对接口及其在 Linux 上的用法的简单介绍。任何有网络背景的人都可能对这个信息感兴趣。通过发出 iproute2 命令`ip link help`，可以获得隧道接口列表以及特定隧道配置的帮助。

本帖涵盖了以下常用接口:

*   [IPIP 隧道](#ipip)
*   [坐隧道](#sit)
*   [ip6tnl 隧道](#ip6tnl)
*   [VTI 和 VTI6](#vti)
*   [GRE 和 GRETAP](#gre)
*   [IP6GRE 和 IP6GRETAP](#ip6gre)
*   [FOU](#fou)
*   [GUE](#gue)
*   日内瓦
*   [ERSPAN 和 IP6ERSPAN](#erspan)

看完这篇文章，你就会知道这些接口是什么，它们之间的区别，什么时候使用它们，以及如何创建它们。

## IPIP 地道

IPIP 隧道，顾名思义，是 IP over IP 隧道，在 [RFC 2003](https://tools.ietf.org/html/rfc2003) 中定义。IPIP 隧道标题看起来像:

![](img/f53c143208b022406d5b00e28bfc2541.png)

它通常用于通过公共 IPv4 互联网连接两个内部 IPv4 子网。它的开销最低，但只能传输 IPv4 单播流量。这意味着你**不能**通过 IPIP 隧道发送组播。

IPIP 隧道支持 IP over IP 和 MPLS over IP。

**注意**:当`ipip`模块加载时，或者第一次创建 IPIP 设备时，Linux 内核会在每个命名空间中创建一个`tunl0`默认设备，属性为`local=any`和`remote=any`。当接收到 IPIP 协议包时，如果找不到另一个本地/远程属性与它们的源地址或目的地址更匹配的设备，内核会将它们转发到`tunl0`作为后备设备。

以下是创建 IPIP 隧道的方法:

```
On Server A:
# ip link add name ipip0 type ipip local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR
# ip link set ipip0 up
# ip addr add INTERNAL_IPV4_ADDR/24 dev ipip0
Add a remote internal subnet route if the endpoints don't belong to the same subnet
# ip route add REMOTE_INTERNAL_SUBNET/24 dev ipip0

On Server B:
# ip link add name ipip0 type ipip local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR
# ip link set ipip0 up
# ip addr add INTERNAL_IPV4_ADDR/24 dev ipip0
# ip route add REMOTE_INTERNAL_SUBNET/24 dev ipip0

```

`Note`:请根据您的测试环境将 LOCAL_IPv4_ADDR、REMOTE_IPv4_ADDR、INTERNAL_IPV4_ADDR、REMOTE_INTERNAL_SUBNET 替换为相应的地址。以下配置示例也是如此。

## 耐着性子地道

SIT 代表简单的互联网过渡。主要目的是互连位于全球 IPv4 互联网中的孤立 IPv6 网络。

最初，它只有 IPv6 over IPv4 隧道模式。但经过多年的发展，它获得了对几种不同模式的支持，如`ipip`(IPIP 隧道也是如此)、`ip6ip`、`mplsip`、`any`。模式`any`用于接受 IP 和 IPv6 流量，这在某些部署中可能很有用。SIT tunnel 还支持 [ISATA](https://www.ietf.org/rfc/rfc4214.txt) ，下面是[的使用示例](http://www.litech.org/isatap)。

SIT 隧道标题看起来像:

![](img/5597feadbe8e40627ce76ae507bd7fe0.png)

当加载`sit`模块时，Linux 内核将创建一个默认设备，名为`sit0`。

以下是创建 SIT 隧道的方法:

```
On Server A:
# ip link add name sit1 type sit local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR mode any
# ip link set sit1 up
# ip addr add INTERNAL_IPV4_ADDR/24 dev sit1

```

然后，在远程端执行相同的步骤。

## ip6tnl 隧道

ip6tnl 是 IPv4/IPv6 over IPv6 隧道接口，看起来像是 SIT 隧道的 IPv6 版本。隧道标题看起来像:

![](img/9a212f88a326f905c85658418c2c7f2b.png)

ip6tnl 支持`ip6ip6`、`ipip6`、`any`模式。模式`ipip6`为 IPv4 over IPv6，模式`ip6ip6`为 IPv6 over IPv6，模式`any`同时支持 IPv4/IPv6 over IPv6。

当加载`ip6tnl`模块时，Linux 内核将创建一个默认设备，名为`ip6tnl0`。

以下是创建 ip6tnl 隧道的方法:

```
# ip link add name ipip6 type ip6tnl local LOCAL_IPv6_ADDR remote REMOTE_IPv6_ADDR mode any

```

## VTI 和 VTI6

Linux 上的虚拟隧道接口(VTI)类似于 Cisco 的 VTI 和 Juniper 的安全隧道(st.xx)实现。

这个特殊的隧道驱动程序实现了 IP 封装，可以与 xfrm 一起使用，给出安全隧道的概念，然后在其上使用内核路由。

一般来说，VTI 隧道的工作方式与 ipip 或 sit 隧道几乎相同，只是它们添加了 fwmark 和 IPsec 封装/解封装。

VTI6 相当于 IPv6 中的 VTI。

以下是创建 VTI 隧道的方法:

```
# ip link add name vti1 type vti key VTI_KEY local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR
# ip link set vti1 up
# ip addr add LOCAL_VIRTUAL_ADDR/24 dev vti1

# ip xfrm state add src LOCAL_IPv4_ADDR dst REMOTE_IPv4_ADDR spi SPI PROTO ALGR mode tunnel
# ip xfrm state add src REMOTE_IPv4_ADDR dst LOCAL_IPv4_ADDR spi SPI PROTO ALGR mode tunnel
# ip xfrm policy add dir in tmpl src REMOTE_IPv4_ADDR dst LOCAL_IPv4_ADDR PROTO mode tunnel mark VTI_KEY
# ip xfrm policy add dir out tmpl src LOCAL_IPv4_ADDR dst REMOTE_IPv4_ADDR PROTO mode tunnel mark VTI_KEY

```

您也可以通过 [libreswan](https://libreswan.org/wiki/Route-based_VPN_using_VTI) 或 [strongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/RouteBasedVPN) 来配置 IPsec。

## GRE 和 GRETAP

通用路由封装，也称为 GRE，在 [RFC 2784](https://tools.ietf.org/html/rfc2784) 中定义

GRE 隧道在内部和外部 IP 报头之间增加了一个额外的 GRE 报头。理论上，GRE 可以用有效的以太网类型封装任何第 3 层协议，不像 IPIP 只能封装 IP。GRE 标题看起来像这样:

![](img/624f8cf9b1f5bbbf83a9e67ca1c79eb4.png)

请注意，您可以通过 GRE 隧道传输多播流量和 IPv6。

当加载`gre`模块时，Linux 内核将创建一个默认设备，名为`gre0`。

以下是创建 GRE 隧道的方法:

```
# ip link add name gre1 type gre local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR [seq] key KEY

```

GRE 隧道工作在 OSI 第 3 层，而 GRETAP 工作在 OSI 第 2 层，这意味着在内部报头中有一个以太网报头。

![](img/9a235a9ad7ce4651c860450d4061d4d5.png)

下面是创建 GRETAP 隧道的方法:

```
# ip link add name gretap1 type gretap local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR

```

## IP6GRE 和 IP6GRETAP

IP6GRE 是 IPv6 的 GRE 等价物，它允许我们封装 IPv6 上的任何第 3 层协议。隧道标题看起来像:

![](img/b42300ade88e1fbb5d056fc0dd5e0a9d.png)

IP6GRETAP 和 GRETAP 一样，在内部头中有一个以太网头:

![](img/7e0a4d9a547addcf6a87068319a4fb24.png)

以下是创建 GRE 隧道的方法:

```
# ip link add name gre1 type ip6gre local LOCAL_IPv6_ADDR remote REMOTE_IPv6_ADDR
# ip link add name gretap1 type ip6gretap local LOCAL_IPv6_ADDR remote REMOTE_IPv6_ADDR

```

## FOU

隧道可以发生在网络堆栈的多个级别。IPIP、SIT、GRE 隧道是 IP 级别的，而 FOU(UDP 上的 foo)是 UDP 级别的隧道。

使用 UDP 隧道有一些优势，因为 UDP 可以与现有的硬件基础设施一起工作，如网卡中的 [RSS](https://en.wikipedia.org/wiki/Network_interface_controller#RSS) ，交换机中的 [ECMP](https://en.wikipedia.org/wiki/Equal-cost_multi-path_routing) ，以及校验和卸载。开发者的[补丁集](https://lwn.net/Articles/614433/)显示了 SIT 和 IPIP 协议的显著性能提升。

目前，FOU 隧道支持基于 IPIP、SIT、GRE 的封装协议。FOU 标题示例如下:

![](img/07bd6070fe17e9cad483fa7f3fd59bef.png)

以下是创建 FOU 隧道的方法:

```
# ip fou add port 5555 ipproto 4
# ip link add name tun1 type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225 encap fou encap-sport auto encap-dport 5555

```

第一个命令为绑定到 5555 的 IPIP 配置了一个 FOU 接收端口；对于 GRE，需要设置`ipproto 47`。第二个命令设置了一个新的 IPIP 虚拟接口(tun1 ),配置为 FOU 封装，目标端口为 5555。

**注意**:红帽企业版 Linux 不支持 FOU。

## GUE

[通用 UDP 封装](https://tools.ietf.org/html/draft-ietf-intarea-gue) (GUE)是另一种 UDP 隧道。FOU 和 GUE 的区别在于，GUE 有自己的封装报头，其中包含协议信息和其他数据。

目前，GUE 隧道支持内部 IPIP、SIT、GRE 封装。GUE 报头的一个示例如下:

![](img/22d760470941413ebb1d8c957c556f9e.png)

以下是创建 GUE 隧道的方法:

```
# ip fou add port 5555 gue
# ip link add name tun1 type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225 encap gue encap-sport auto encap-dport 5555

```

这将为绑定到 5555 的 IPIP 设置一个 GUE 接收端口，并为 GUE 封装配置一个 IPIP 隧道。

**注意**:红帽企业版 Linux 不支持 GUE。

## 日内瓦

通用网络虚拟化封装(GENEVE)支持 VXLAN、NVGRE 和 STT 的所有功能，旨在克服它们的局限性。许多人认为 GENEVE 最终会完全取代这些早期的格式。隧道标题看起来像:

![](img/b3b45f66bb86e29c4bee812f1c409bec.png)

这看起来非常类似于 [VXLAN](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/#vxlan) 。主要的区别是 GENEVE 头是灵活的。通过使用一个新的类型-长度-值(TLV)字段来扩展报头，可以非常容易地添加新特性。更多细节可以看最新的 [geneve ietf 草案](https://tools.ietf.org/html/draft-ietf-nvo3-geneve-08)或者参考这个[什么是 geneve？](https://www.redhat.com/en/blog/what-geneve)文章。

[开放虚拟网络(OVN)](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/13/html/networking_with_open_virtual_network/open_virtual_network_ovn) 使用 GENEVE 作为默认封装。以下是创建 GENEVE 隧道的方法:

```
# ip link add name geneve0 type geneve id VNI remote REMOTE_IPv4_ADDR

```

## ERSPAN 和 IP6ERSPAN

封装远程交换端口分析器(ERSPAN)使用 GRE 封装将基本端口镜像功能从第 2 层扩展到第 3 层，从而允许镜像流量通过可路由的 IP 网络发送。ERSPAN 标题看起来像:

![](img/c1a69f6d3a894cc75c2f19ae85cbefe8.png)

ERSPAN 隧道允许 Linux 主机充当 ERSPAN 流量源，并将 ERSPAN 镜像流量发送到远程主机或 ERSPAN 目的地，目的地接收并解析 Cisco 或其他支持 ERSPAN 的交换机生成的 ERSPAN 数据包。这种设置可用于分析、诊断和检测恶意流量。

Linux 目前支持两个 ERSPAN 版本的大部分特性:v1 (type II)和 v2 (type III)。

以下是创建二跨隧道的方法:

```
# ip link add dev erspan1 type erspan local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR seq key KEY erspan_ver 1 erspan IDX
or
# ip link add dev erspan1 type erspan local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR seq key KEY erspan_ver 2 erspan_dir DIRECTION erspan_hwid HWID

Add tc filter to monitor traffic
# tc qdisc add dev MONITOR_DEV handle ffff: ingress
# tc filter add dev MONITOR_DEV parent ffff: matchall skip_hw action mirred egress mirror dev erspan1

```

## 摘要

下面是我们介绍的所有隧道的汇总。

| 隧道/链路类型 | 外部集管 | 封装标题 | 内部集管 |
| ipip | IPv4 | 没有人 | IPv4 |
| 使就座 | IPv4 | 没有人 | IPv4/IPv6 |
| ip6tnl | IPv6 | 没有人 | IPv4/IPv6 |
| vti | IPv4 | IPsec | IPv4 |
| vti6 | IPv6 | IPsec | IPv6 |
| 美国研究生入学考试（Graduate Record Examination） | IPv4 | 美国研究生入学考试(Graduate Record Examination) | IPv4/IPv6 |
| 格雷塔普 | IPv4 | 美国研究生入学考试(Graduate Record Examination) | 以太网+ IPv4/IPv6 |
| ip6gre | IPv6 | 美国研究生入学考试(Graduate Record Examination) | IPv4/IPv6 |
| ip6gretap | IPv6 | 美国研究生入学考试(Graduate Record Examination) | 以太网+ IPv4/IPv6 |
| 喝醉的 | IPv4/IPv6 | 用户数据报协议(User Datagram Protocol) | IPv4/IPv6/GRE |
| gue | IPv4/IPv6 | UDP + GUE | IPv4/IPv6/GRE |
| 日内瓦 | IPv4/IPv6 | UDP +日内瓦 | 以太网+ IPv4/IPv6 |
| 埃尔斯潘 | IPv4 | GRE + ERSPAN | IPv4/IPv6 |
| IP 6 ers span | IPv6 | GRE + ERSPAN | IPv4/IPv6 |

**注意**:本教程中的所有配置都是易变的，不会在服务器重启后继续存在。如果您想让配置在重启后保持不变，请考虑使用网络配置守护进程，比如[网络管理器](https://developer.gnome.org/NetworkManager/stable/)，或者特定于发行版的机制。

*也可阅读:[虚拟网络的 Linux 接口介绍](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/)*

*Last updated: October 18, 2019*