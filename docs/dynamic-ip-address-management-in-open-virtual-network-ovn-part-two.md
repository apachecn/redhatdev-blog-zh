# 开放虚拟网络(OVN)中的动态 IP 地址管理(下)

> 原文：<https://developers.redhat.com/blog/2018/09/27/dynamic-ip-address-management-in-open-virtual-network-ovn-part-two>

在本系列的第一部分中，我们探索了开放虚拟网络的动态 IP 地址管理(IPAM)功能。我们讨论了逻辑交换机上的`subnet`、`ipv6_prefix`和`exclude_ips`选项。然后，我们看到这些选项如何应用于其地址已被设置为特殊“动态”值的逻辑交换机端口。OVN 是 Open vSwitch 的子项目，用于许多 Red Hat 产品中的虚拟网络，如 Red Hat OpenStack 平台、Red Hat 虚拟化和未来版本中的 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview/)。

在这一部分中，我们将探索该特性中的一些疏忽和缺点，这些是如何被纠正的，以及在未来的版本中为 OVN 准备了什么。

## 子网更改

让我们首先创建一个简单的逻辑交换机，它有几个使用动态地址的逻辑交换机端口:

```
ovn-nbctl ls-add sw
ovn-nbctl set Logical_Switch sw other_config:subnet=192.168.1.0/24
ovn-nbctl lsp-add sw sw-p1
ovn-nbctl lsp-set-addresses sw-p1 "dynamic"
ovn-nbctl lsp-add sw sw-p2
ovn-nbctl lsp-set-addresses sw-p2 "dynamic"

```

这创建了一个带有端口`sw-p1`和`sw-p2`的逻辑交换机`sw`。端口`sw-p1`被分配地址 192.168.1.2，端口`sw-p2`被分配地址 192.168.1.3。

但是等等:我们犯了一个错误！我们实际上打算将交换机的子网设置为`192.168.0.0/24`。我们来纠正一下。

```
ovn-nbctl set Logical_Switch sw other_config:subnet=192.168.0.0/24

```

好了，让我们看看这是如何影响逻辑交换机端口地址的。如果您运行的是 [Open vSwitch](http://www.openvswitch.org/) (OVS) 2.9 系列或更早版本，那么您会看到以下内容:

```
$ ovn-nbctl --columns=name,dynamic_addresses,addresses list logical_switch_port
name                : "sw-p2"
dynamic_addresses   : "0a:00:00:00:00:02 192.168.1.3"
addresses           : [dynamic]

name                : "sw-p1"
dynamic_addresses   : "0a:00:00:00:00:01 192.168.1.2"
addresses           : [dynamic]

```

啊？动态地址没有更新。在 2.10 版之前，如果更新了`subnet`、`ipv6_prefix`或`exclude_ips`，动态地址不会自动更新。如果您想要更新动态地址，您需要从受影响的逻辑交换机端口清除`dynamic_addresses`。清除交换机`sw`上所有交换机端口上的`dynamic_addresses`的最简单方法如下:

```
for port in $(ovn-nbctl --bare --columns=port find logical_switch name=sw) ; do ovn-nbctl clear logical_switch_port $port dynamic_addresses ; done

```

现在，让我们再看一下逻辑交换机端口:

```
$ ovn-nbctl --columns=name,dynamic_addresses,addresses list logical_switch_port
name                : "sw-p2"
dynamic_addresses   : "0a:00:00:00:00:03 192.168.0.2"
addresses           : [dynamic]

name                : "sw-p1"
dynamic_addresses   : "0a:00:00:00:00:04 192.168.0.3"
addresses           : [dynamic]

```

那里；那更好。这里有几件事需要注意。首先，IP 地址分配给交换机端口的顺序并不总是可预测的。分配给交换机端口的 IP 地址的最后一个二进制八位数与其之前的二进制八位数进行了交换。此外，每个交换机端口上的 MAC 地址都已更新。当我们清除`dynamic_addresses`时，交换机端口上的 MAC 地址分配丢失了。因此，`ovn-northd`给端口分配新的 MAC 地址。不幸的是，如果您使用动态 MAC 地址，这是不可避免的。

好消息是，从 OVS 2.10.0 开始，这不再是必要的。更新逻辑交换机上的`subnet`、`ipv6_prefix`或`exclude_ips`将自动更新所有逻辑交换机端口上的`dynamic_addresses`。更好的消息是，只有受影响的值被更新，所以在这种特殊情况下，每个交换机端口上的 MAC 地址保持不变。

## 冲突的地址

让我们使用上一节中的交换机，并添加第三个交换机端口:

```
ovn-nbctl lsp-add sw sw-p3
ovn-nbctl lsp-set-addresses sw-p3 "00:00:00:00:00:03 192.168.0.3"

```

现在，让我们看看我们的交换机端口:

```
name                : "sw-p3"
dynamic_addresses   : []
addresses           : ["00:00:00:00:00:03 192.168.0.3"]

name                : "sw-p2"
dynamic_addresses   : "0a:00:00:00:00:03 192.168.0.2"
addresses           : [dynamic]

name                : "sw-p1"
dynamic_addresses   : "0a:00:00:00:00:04 192.168.0.3"
addresses           : [dynamic]

```

糟糕——我们的新交换机端口的地址与我们的一个动态地址冲突。这将导致发送数据包时出错。有几种方法可以解决这个问题。

解决这个问题的一个方法是清除`sw-p2`的`dynamic_addresses`，然后`sw-p2`将获得一个新的动态地址。如前所述，这也意味着`sw-p2`将被分配一个新的 MAC 地址。

另一种方法是在`sw-p3`上使用`ovn-nbctl lsp-set-addresses`，这样它就有一个不冲突的地址。

从 OVS 版本 2.10.0 开始，这种冲突不再发生。相反，`sw-p2`会自动将其 IP 地址更新为子网中的下一个可用地址。该代码假设静态分配的地址总是正确的，而动态地址是“错误的”,在发生冲突时需要更新。

从 OVS 版本 2.11.0 开始，导致这种类型的冲突将变得更加困难。看看当我们对 OVS 的现任主人做以下尝试时会发生什么:

```
$ ovn-nbctl lsp-set-addresses sw-p3 "00:00:00:00:00:03 192.168.0.3"
ovn-nbctl: Error on switch sw: duplicate IPv4 address 192.168.0.3

```

上述消息表明`ovn-nbctl`检测到冲突，并且冲突地址未在`sw-p3`上设置。仍然有可能使用以下命令在`sw-p3`上设置冲突地址:

```
# Don't do this!
$ ovn-nbctl set Logical_Switch_Port sw-p3 "00:00:00:00:00:03 192.168.0.3"

```

这样做仍然会导致在北向数据库中设置冲突的地址，并且会导致`sw-p2`被分配一个新的 IP 地址。

## 其他已修复的问题

在这最后一节中，我们将检查一些在 OVS 2.10 系列中修复的小问题。与前两节中探讨的问题相比，这些问题发生的可能性要小得多，而且它们是相似的。这里有一个简短的总结:

*   在 2.10 之前，如果交换机端口上的 MAC 地址从静态分配更改为动态分配，则 MAC 地址不会更新。在 2.10+中，MAC 地址是动态分配的。
*   在 2.10 之前，如果 IPv6 地址是动态分配的，并且端口上的 MAC 地址发生了变化，则 IPv6 地址不会更新。在 2.10+中，当 MAC 地址更改时，也会重新计算 IPv6 地址。

## IPAM 在 OVN 的未来

IPAM 提供了一种简便的方法，让 IP 地址和 MAC 地址自动分配给你的逻辑交换机端口。在[第 1 部分](https://developers.redhat.com/blog/2018/09/03/ovn-dynamic-ip-address-management/)中，我们探索了在 OVN 启用 IPAM 的基础，在这一部分中，我们看到了最近已经修复的一些缺点。但是接下来会发生什么呢？新的开发与其说集中在修复问题上，不如说集中在增加功能上。

管道中的一个改进是允许配置可分配的 MAC 地址池。正如我们在这些帖子中看到的，OVN 将分配以“0a”开头的 MAC 地址但是，如果您希望 OVN 分配 MAC 地址，但又想选择要分配的 MAC 地址范围，该怎么办呢？这是目前正在开发的。一个想法是提供一个开始和结束地址，允许 OVN 在这个范围内分配地址。另一个想法是允许配置组织唯一标识符(OUI ),并使用该 OUI 作为前缀来分配 OVN 地址。

另一个改进是提供一致的 IPv4 地址和 MAC 地址配对。目前，OVN 独立分配 MAC 和 IPv4 地址。然而，在 ARP 表中，每次尝试使用相同的 MAC 地址分配相同的 IPv4 地址会更友好。

上述两个想法目前都在开发中，目标是在 OVS 的 2.11 系列中可用。我相信那些正在阅读这些博文的人会有进一步添加功能的想法。如果你这样做了，请在这篇文章中留下你的建议。

*Last updated: February 17, 2022*