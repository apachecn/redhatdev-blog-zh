# 如何创建一个开放的虚拟网络分布式网关路由器

> 原文：<https://developers.redhat.com/blog/2018/11/08/how-to-create-an-open-virtual-network-distributed-gateway-router>

在本文中，我使用分布式网关路由器讨论了[开放虚拟网络(OVN)](http://www.openvswitch.org/support/dist-docs/ovn-architecture.7.html) 中的外部连接，这是[开放虚拟交换机](https://www.openvswitch.org/) (OVS)的子项目。

OVN 通过两种方式提供外部连接:

*   具有分布式网关端口的逻辑路由器，在本文中称为分布式网关路由器
*   逻辑网关路由器

在本文中，您将看到如何创建一个分布式网关路由器，以及它如何工作的一个例子。

与为 CMS(云管理系统)使用逻辑网关路由器相比，创建分布式网关路由器有一些优势:

*   创建分布式网关路由器更容易，因为 CMS 不需要创建逻辑网关路由器所需的转接逻辑交换机。
*   分布式网关路由器支持分布式北/南流量，而逻辑网关路由器集中在一个网关机箱上。
*   分布式网关路由器支持高可用性。

**注意**:CMS 可以是 OpenStack、[红帽 OpenShift](http://openshift.com/) 、[红帽虚拟化](https://www.redhat.com/en/technologies/virtualization/enterprise-virtualization)或者其他任何管理云的系统。

## 设置详细信息

先说部署细节。我将举一个有五个节点的例子，其中三个是控制器节点，其余的是计算节点。租户虚拟机在计算节点中创建。控制器节点以主动/被动模式运行 OVN 数据库服务器。

**注意**:当您运行命令`ovn-nbctl/ovn-sbctl`时，它应该在运行 OVN 数据库服务器的节点上运行。或者，您可以传递带有 IP 地址/端口的`--db`选项。

### OVN 的底盘

在 OVN 术语中，每个节点被称为*机箱*。机箱只不过是运行`ovn-controller`服务的一个节点。为了让机箱充当*网关机箱，*它应该能够为租户流量提供外部(北/南)连接。它还需要以下配置:

*   配置`ovn-bridge-mappings`，它提供了一个键值对列表，将一个物理网络名称映射到一个本地 OVS 网桥，该网桥提供到该网络的连接。

```
ovs-vsctl set open . external-ids:ovn-bridge-mappings=provider:br-provider
```

*   创建提供商 OVS 网桥，并向 OVS 网桥添加提供外部连接的接口:

```
ovs-vsctl --may-exist add-br br-provider
ovs-vsctl --may-exist add-port br-provider INTERFACE_NAME
```

在上面的设置中，所有控制器节点都充当网关机箱。

下面是显示我的设置的`ovn-sbctl`的输出:

```
Chassis "controller-0"
    hostname: "controller-0.localdomain"
    Encap geneve
        ip: "172.17.2.28"
        options: {csum="true"}
Chassis "controller-1"
    hostname: "controller-1.localdomain"
    Encap geneve
        ip: "172.17.2.26"
        options: {csum="true"}
Chassis "controller-2"
    hostname: "controller-2.localdomain"
    Encap geneve
        ip: "172.17.2.18"
        options: {csum="true"}
Chassis "compute-0"
    hostname: "compute-0.localdomain"
    Encap geneve
        ip: "172.17.2.15"
        options: {csum="true"}
Chassis "compute-1"
    hostname: "compute-1.localdomain"
    Encap geneve
        ip: "172.17.2.17"
        options: {csum="true"}
```

让我们首先创建几个逻辑交换机和逻辑端口，并将它们连接到一个逻辑路由器:

```
ovn-nbctl ls-add sw0
ovn-nbctl lsp-add sw0 sw0-port1
ovn-nbctl lsp-set-addresses sw0-port1 "00:00:01:00:00:03 10.0.0.3"

ovn-nbctl ls-add sw1
ovn-nbctl lsp-add sw1 sw1-port1
ovn-nbctl lsp-set-addresses sw1-port1 "00:00:02:00:00:03 20.0.0.3"

ovn-nbctl lr-add lr0
# Connect sw0 to lr0
ovn-nbctl lrp-add lr0 lr0-sw0 00:00:00:00:ff:01 10.0.0.1/24
ovn-nbctl lsp-add sw0 sw0-lr0
ovn-nbctl lsp-set-type sw0-lr0 router
ovn-nbctl lsp-set-addresses sw0-lr0 router
ovn-nbctl lsp-set-options sw0-lr0 router-port=lr0-sw0

# Connect sw1 to lr0
ovn-nbctl lrp-add lr0 lr0-sw1 00:00:00:00:ff:02 20.0.0.1/24
ovn-nbctl lsp-add sw1 sw1-lr0
ovn-nbctl lsp-set-type sw1-lr0 router
ovn-nbctl lsp-set-addresses sw1-lr0 router
ovn-nbctl lsp-set-options sw1-lr0 router-port=lr0-sw1

```

下面是`ovn-nbctl`的输出:

```
ovn-nbctl show
switch 05cf23bc-2c87-4d6d-a76b-f432e562ed71 (sw0)
    port sw0-port1
        addresses: ["00:00:01:00:00:03 10.0.0.3"]
    port sw0-lr0
        type: router
        router-port: lr0-sw0
switch 0dfee7ef-13b3-4cd0-87a1-7935149f551e (sw1)
    port sw1-port1
        addresses: ["00:00:02:00:00:03 20.0.0.3"]
    port sw1-lr0
        type: router
        router-port: lr0-sw1
router c189f271-86d6-4f7f-891c-672cb3aa543e (lr0)
    port lr0-sw0
        mac: "00:00:00:00:ff:01"
        networks: ["10.0.0.1/24"]
    port lr0-sw1
        mac: "00:00:00:00:ff:02"
        networks: ["20.0.0.1/24"]

```

端口`sw0-port1`可以与`sw1-port1`通信，因为交换机连接到逻辑路由器`lr0`。东西交通与 OVN 分布。

现在，让我们创建一个提供商逻辑交换机:

```
ovn-nbctl ls-add public
# Create a localnet port
ovn-nbctl lsp-add public ln-public
ovn-nbctl lsp-set-type ln-public localnet
ovn-nbctl lsp-set-addresses ln-public unknown
ovn-nbctl lsp-set-options ln-public network_name=provider
```

注意`network_name=provider`。`network_name`应该与`ovn-bridge-mappings` **中定义的列表相匹配。**当逻辑交换机中定义了本地网络端口时，运行在网关机箱上的`ovn-controller`会在集成桥和提供商桥之间创建一个 OVS 补丁端口，以便逻辑租户流量进出物理网络。

此时，来自逻辑交换机`sw0`和`sw1`的租户流量仍然不能进入`public`逻辑交换机，因为它和逻辑路由器`lr0`之间没有关联。

### 创建分布式路由器端口

我们先把`lr0`和`public`连接起来:

```
ovn-nbctl lrp-add lr0 lr0-public 00:00:20:20:12:13 172.168.0.200/24
ovn-nbctl lsp-add public public-lr0
ovn-nbctl lsp-set-type public-lr0 router
ovn-nbctl lsp-set-addresses public-lr0 router
ovn-nbctl lsp-set-options public-lr0 router-port=lr0-public
```

我们仍然需要将分布式网关端口`lr0-public`调度到网关机箱。这里的*排班*是什么意思？这意味着被选择作为网关路由器端口主机的机箱提供了集中的外部连接。南北租户流量将被重定向到此机箱，并充当网关。该机箱在通过配线端口向提供商网桥发送流量之前应用所有的 NAT 规则。这也意味着，当有人 ping 172.168.0.200 或发送 ARP 请求到 172.168.0.200 时，承载该请求的网关机箱将响应 ping 和 ARP 回复。

### 调度网关路由器端口

这可以通过两种方式实现:

*   非高可用性(non-HA)模式:网关路由器端口被配置为在单个网关机箱上调度。如果托管此端口的网关机箱由于某种原因关闭，外部连接将完全中断，直到 CMS(云管理系统)检测到这一情况并将其重新安排到另一个网关机箱。
*   HA 模式:网关路由器端口被配置为在一组网关机箱上调度。配置了高优先级的网关机箱要求网关路由器端口。如果此网关机箱由于某种原因关闭，下一个优先级更高的网关机箱将占用网关路由器端口。

#### 非高可用性模式下的计划

选择要在其中安排网关路由器端口的网关机箱。让我们安排在`controller-0`吧。有两种方法可以做到。运行以下命令之一:

```
ovn-nbctl set logical_router_port lr0-public options:redirect-chassis=controller-0

ovn-nbctl list logical_router_port lr0-public
_uuid : 0ced9cdb-fbc9-47f1-b2e2-97a49988d622
enabled : []
external_ids : {}
gateway_chassis : []
ipv6_ra_configs : {}
mac : "00:00:20:20:12:13"
name : "lr0-public"
networks : ["172.168.0.200/24"]
options : {redirect-chassis="controller-0"} peer : []
or
ovn-nbctl lrp-set-gateway-chassis lr0-public controller-0 20

```

在下面的`ovn-sbctl show`输出中，您可以看到`controller-0`正在托管网关路由器端口`lr0-public`。

```
ovn-sbctl show
Chassis "d86bd6f2-1216-4a73-bcaf-3200b8ed8126"
    hostname: "controller-0.localdomain"
    Encap geneve
    ip: "172.17.2.28"
    options: {csum="true"}
    Port_Binding "cr-lr0-public"
Chassis "20dc7bfb-a329-4cf9-a8ac-3485f7d5be46"
    hostname: "controller-1.localdomain"
    ...
    ...

```

#### 高可用性模式下的计划

在这种情况下，我们选择一组网关机箱，并为每个机箱设置优先级。优先级最高的机箱将托管网关路由器端口。

在我们的例子中，让我们设置所有的网关机箱:`controller-0`优先级为 20，`controller-1`优先级为 15，`controller-2`优先级为 10。

运行以下命令:

```
ovn-nbctl lrp-set-gateway-chassis lr0-public controller-0 20
ovn-nbctl lrp-set-gateway-chassis lr0-public controller-1 15
ovn-nbctl lrp-set-gateway-chassis lr0-public controller-2 10

```

您可以通过运行以下命令来验证配置:

```
ovn-nbctl list gateway_chassis
_uuid : 745d7f84-0516-4a0f-9b3d-772e5cb58a48
chassis_name : "controller-1"
external_ids : {}
name : "lr0-public-controller-1"
options : {}
priority : 15

_uuid : 6f2921d4-2555-4f81-9428-640cbf62151e
chassis_name : "controller-0"
external_ids : {}
name : "lr0-public-controller-0"
options : {}
priority : 20

_uuid : 97595b29-139d-4a43-9973-8995ffe17c64
chassis_name : "controller-2"
external_ids : {}
name : "lr0-public-controller-2"
options : {}
priority : 10
```

```
ovn-nbctl list logical_router_port lr0-public
_uuid : 0ced9cdb-fbc9-47f1-b2e2-97a49988d622
enabled : []
external_ids : {}
gateway_chassis : [6f2921d4-2555-4f81-9428-640cbf62151e, 745d7f84-0516-4a0f-9b3d-772e5cb58a48, 97595b29-139d-4a43-9973-8995ffe17c64]
ipv6_ra_configs : {}
mac : "00:00:20:20:12:13"
name : "lr0-public"
networks : ["172.168.0.200/24"]
options : {}
peer : []

ovn-sbctl show
Chassis "d86bd6f2-1216-4a73-bcaf-3200b8ed8126"
    hostname: "controller-0.localdomain"
    Encap geneve
        ip: "172.17.2.28"
        options: {csum="true"}
    Port_Binding "cr-lr0-public"
Chassis "20dc7bfb-a329-4cf9-a8ac-3485f7d5be46"
    hostname: "controller-1.localdomain"
    ...
    ...

```

您始终可以通过运行以下命令来删除网关机箱与分布式路由器端口的关联:

```
ovn-nbctl lrp-del-gateway-chassis lr0-public controller-1
```

为了支持高可用性，OVN 使用双向转发检测(BFD)协议。它在隧道端口上配置 BFD。当托管分布式网关端口的网关机箱出现故障时，所有机箱都会检测到故障(由于 BFD)，下一个优先级更高的网关机箱会占用该端口。有关更多详细信息，请参考[这个](https://docs.openstack.org/networking-ovn/latest/admin/routing.html#l3ha-support)，并运行以下命令来访问 OVN 手册页:`man ovn-nb`、`man ovn-northd`和`man ovn-controller`。

## 机箱重定向端口

在`ovn-sbctl show`的输出中，可以看到`Port_Binding "cr-lr0-public"` *。*什么是`cr-lr0-public`？对于每个调度的网关路由器端口，`ovn-northd`在内部创建一个类型为 *chassisredirect* 的逻辑端口。此端口代表在选定机箱上调度的分布式网关端口的实例。

## 当虚拟机发送外部流量时会发生什么？

现在，让我们从 OVN 逻辑数据路径管道的角度，简要地看看当与逻辑端口(假设为`sw0-port0`)相关联的 VM 向目的地 172.168.0.110 发送分组时会发生什么。让我们假设虚拟机运行在`compute-0`上，机箱重定向端口安排在`controller-0`上。172.168.0.110 可以与可通过提供商网络到达的物理服务器或虚拟机相关联。

在计算机箱上，会发生以下情况:

*   当虚拟机发送流量时，`sw0`的逻辑交换管道运行。
*   当数据包需要被路由时，它从逻辑交换机管道通过`lr0-sw0`端口进入入口路由器管道。
*   运行入口路由器流水线，做出路由决定，并将输出端口设置为`lr0-public`。

```
 Logical flows
 table=0 (lr_in_admission ), priority=50 , match=(eth.dst == 00:00:00:00:ff:01 && inport == "lr0-sw0"), action=(next;)
 ...
 table=7 (lr_in_ip_routing ), priority=49 , match=(ip4.dst == 172.168.0.0/24), action=(ip.ttl--; reg0 = ip4.dst; reg1 = 172.168.0.200; eth.src = 00:00:20:20:12:13; outport = "lr0-public"; flags.loopback = 1; next;)
 ...
 table=9 (lr_in_gw_redirect ), priority=50 , match=(outport == "lr0-public"), action=(outport = "cr-lr0-public"; next;)

```

*   由于`cr-lr0-public`被安排在`controller-0`上，数据包通过隧道端口发送到`controller-0`:

```
  table=32, priority=100,reg15=0x4,metadata=0x3 actions=load:0x3->NXM_NX_TUN_ID[0..23],set_field:0x4->tun_metadata0,move:NXM_NX_REG14[0..14]->NXM_NX_TUN_METADATA0[16..30],output:ovn-cont-0
```

在控制器-0 机箱上，会发生以下情况:

*   `controller-0`接收隧道端口上的流量，并将流量发送到逻辑路由器`lr0`的出口管道:

```
table=0, priority=100,in_port="ovn-comp-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,33)
```

*   应用 NAT 规则。也就是说，源 IP 地址 10.0.0.3 被 NAT 为 172.168.0.200:

```
  table=1 (lr_out_snat ), priority=25 , match=(ip && ip4.src == 10.0.0.0/24 && outport == "lr0-public" && is_chassis_resident("cr-lr0-public")), action=(ct_snat(172.168.0.200);)
```

*   数据包通过`lr0-public`端口发送到逻辑交换机`public`:

```
  table=3 (lr_out_delivery ), priority=100 , match=(outport == "lr0-public"), action=(output;)

```

*   数据包通过本地网络端口发送到提供商网桥并到达目的地。

现在让我们看看回复流量会发生什么。

在`controller-0`底盘上:

*   该数据包由提供商网桥中的物理接口接收，并通过`localnet`端口进入入口逻辑交换机`public`的管道:

```
 table=0,priority=100,in_port="patch-br-int-to",dl_vlan=0 actions=strip_vlan,load:0x1->NXM_NX_REG13[],load:0x7->NXM_NX_REG11[],load:0x8->NXM_NX_REG12[],load:0x4->OXM_OF_METADATA[],load:0x2->NXM_NX_REG14[],resubmit(,8)
```

*   从`public`开始，数据包通过`public-lr0`逻辑端口进入`lr0`的流水线。
*   在入口路由器管道中，应用了 UnSNAT 规则。也就是说，目的 IP 地址没有从 172.168.0.200 到 10.0.0.3 的 NAT:

```
 table=0 (lr_in_admission ), priority=50 , match=(eth.dst == 00:00:20:20:12:13 && inport == "lr0-public" && is_chassis_resident("cr-lr0-public")), action=(next;)
 ...
 table=3 (lr_in_unsnat ), priority=100 , match=(ip && ip4.dst == 172.168.0.200 && inport == "lr0-public" && is_chassis_resident("cr-lr0-public")), action=(ct_snat;)

```

*   由于 10.0.0.3 属于逻辑交换机`sw0`，数据包通过`lr0-sw0`进入`sw0`的入口管道:

```
 table=7 (lr_in_ip_routing ), priority=49 , match=(ip4.dst == 10.0.0.0/24), action=(ip.ttl--; reg0 = ip4.dst; reg1 = 10.0.0.1; eth.src = 00:00:00:00:ff:01; outport = "lr0-sw0"; flags.loopback = 1; next;)
```

*   运行`sw0`的入口管道，通过隧道端口将数据包发送到`compute-0`，因为 OVN 知道`sw0-port1`驻留在`compute-0`上。

在`compute-0`底盘上，会发生以下情况:

*   `compute-0`接收隧道端口上的流量，并将流量发送到逻辑交换机`sw0`的出口管道。
*   在出口管道中，数据包被传送到`sw0-port1`。

## 结论

本文概述了 OVN 的分布式网关路由器，它是如何创建的，以及当 VM 发送外部流量时会发生什么。希望这将有助于了解 OVN 的外部连接支持，并解决任何相关问题。

## 额外资源

请参阅关于[打开虚拟交换机](https://developers.redhat.com/blog/tag/open-vswitch/)和打开虚拟网络的其他虚拟网络文章:

*   [开放虚拟网络中的动态 IP 地址管理(OVN):第一部分](https://developers.redhat.com/blog/2018/09/03/ovn-dynamic-ip-address-management/)
*   [开放虚拟网络中的动态 IP 地址管理(OVN):第二部分](https://developers.redhat.com/blog/2018/09/27/dynamic-ip-address-management-in-open-virtual-network-ovn-part-two/)
*   [RHEL 无根开放式虚拟交换机](https://developers.redhat.com/blog/2018/03/23/non-root-open-vswitch-rhel/)
*   [打开 vSwitch-DPDK:多少 Hugepage 内存？](https://developers.redhat.com/blog/2018/03/16/ovs-dpdk-hugepage-memory/)
*   [打开 vSwitch: QinQ 性能](https://developers.redhat.com/blog/2017/06/27/open-vswitch-qinq-performance/)

*Last updated: January 5, 2022*