# 通过部分硬件卸载加速开放式虚拟交换机

> 原文：<https://developers.redhat.com/blog/2019/01/09/speeding-up-open-vswitch-with-partial-hardware-offloading>

[打开 vSwitch (OVS)](http://www.openvswitch.org/) 可以使用内核数据路径或用户空间数据路径。在通过 TC Flower 数据包分类器使用硬件卸载的内核数据路径方面有一些有趣的发展，但在本文中，重点将放在使用数据平面开发套件(DPDK)加速的用户空间数据路径及其新特性— *部分流硬件卸载*—以进一步加速虚拟交换机。

本文解释了虚拟交换机以前和现在的工作方式，以及为什么新功能可以在提高数据包处理速率的同时潜在地节省资源。

## 带和不带流硬件卸载的 DPDK 加速 OVS

让我们首先回顾一下 DPDK 加速的 OVS 在没有流硬件卸载的情况下是如何工作的。应该有一个或多个用户空间线程负责不断地轮询网卡上的新数据包，对它们进行分类，并执行相应的操作。对更高速度的需求从未停止，为了更快，每个阶段都需要做好自己的工作。

DPDK 提供了优化的方法来查询新的包，获取任何包，并在需要时将它们发送出去。这是输入输出部分。接下来是数据包分类，依次包括三个阶段。

第一阶段在收到称为 EMC(精确匹配缓存)的数据包时使用。正如您所料，这是最快的机制，但它也有局限性。基本思想是计算一个特定于数据包的值(散列),并使用该值在包含要执行的操作的缓存中搜索流规则。

然而，为每个数据包计算哈希值是一项昂贵的任务，因此，如果网卡支持硬件卸载，这就是硬件卸载的第一个例子，现在大多数网卡都支持硬件卸载。从版本 2.5.0 开始，OVS-DPDK 使用网卡提供的 RSS 哈希在缓存中搜索流。现在我们有额外的周期来处理下一个数据包！

然而，如上所述，缓存有其局限性，例如处理哈希冲突，这需要解析数据包报头以确保找到正确的流。缓存也不能太大或太小，因此根据用例/流量模式，缓存可能不是非常高效。这方面有所改进，例如“有条件的 EMC 插入”，但这是另一篇文章的主题。

今天，OVS-DPDK 的最终目标是将所有的每包处理工作(将包匹配到特定的流规则并执行相应的动作)推给网卡。这将释放系统资源(如主处理器和内存)来做其他工作，提高数据包处理速度，而虚拟交换机将负责管理卡和相关任务，例如提供流量统计。这就是所谓的流硬件卸载，目前还没有。但是从 OVS 2.10 开始，实验性的**部分硬件卸载**已经可用。默认禁用，目前仅限于某些网卡和流。

实验性部分硬件卸载的想法是，OVS-DPDK 将流规则和唯一标记一起推送到网卡，网卡将匹配属于每个流规则的数据包，并相应地标记它们。然后，虚拟交换机将使用每个唯一的标记来找到特定的流规则，然后在软件中执行必要的动作。虽然它看起来很像上面描述的 EMC，但在这种情况下，一些昂贵的任务在网卡中执行。例如，虚拟交换机不需要像以前一样解析所有的数据包报头，因为标记保证是唯一的，如果流的数量超过 EMC 可以处理的数量，它也不需要避免在软件中使用另一级缓存。

总之，OVS-DPDK 利用硬件中对网卡流标记操作的支持来跳过主机中一些非常昂贵的 CPU 操作。通过这种方式，OVS-DPDK 可以处理更多的数据包，或者潜在地减少陷入网络操作的处理器数量。

关于 OVS-DPDK 和流硬件卸载的更多信息可以在[项目的文档](https://github.com/openvswitch/ovs/blob/master/Documentation/howto/dpdk.rst)中找到。

## 其他资源

红帽开发者博客上还有其他 [OVS](https://developers.redhat.com/blog/?s=openvswitch) 和 [OVN](https://developers.redhat.com/blog/?s=OVN) 的文章，大家可能会感兴趣，包括:

*   [使用 Open vSwitch DPDK 调试内存问题](https://developers.redhat.com/blog/2018/06/14/debugging-ovs-dpdk-memory-issues/)
*   [OVN 的性能改进:过去和未来](https://developers.redhat.com/blog/2019/01/02/performance-improvements-in-ovn-past-and-future)
*   [OVS-DPDK 参数:应对多 NUMA](https://developers.redhat.com/blog/2017/06/28/ovs-dpdk-parameters-dealing-with-multi-numa/)
*   [打开 vSwitch-DPDK:多少 Hugepage 内存？](https://developers.redhat.com/blog/2018/03/16/ovs-dpdk-hugepage-memory/)
*   [OVN 的 IP 包缓冲](https://developers.redhat.com/blog/2018/12/07/ip-packet-buffering-in-ovn/)
*   [开放虚拟网络中的动态 IP 地址管理(OVN):第一部分](https://developers.redhat.com/blog/2018/09/03/ovn-dynamic-ip-address-management)
*   [如何创建一个开放的虚拟网络分布式网关路由器](https://developers.redhat.com/blog/2018/11/08/how-to-create-an-open-virtual-network-distributed-gateway-router/)
*   [虚拟网络 Linux 接口介绍](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/)

*Last updated: January 8, 2019*