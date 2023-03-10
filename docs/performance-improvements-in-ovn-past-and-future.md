# OVN 的性能改进:过去和未来

> 原文：<https://developers.redhat.com/blog/2019/01/02/performance-improvements-in-ovn-past-and-future>

[OVN(开放式虚拟网络)](https://www.ovn.org/en/)是[开放式虚拟交换机(OVS)](http://www.openvswitch.org/) 的子组件。它允许通过连接逻辑路由器和逻辑交换机来表达覆盖网络。云提供商和云管理系统多年来一直使用 OVS 作为创建和管理覆盖网络的有效方法。

最近，OVN 获得了自己的地位，因为它被更多地用于红帽产品中。结果是对 OVN 真实情况的审查越来越多。这使得 OVN 增加了新的功能。更重要的是，这导致了巨大的变化，以提高在 OVN 的表现。

在本文中，我将讨论 OVN 在过去一年中增加的两个改变游戏规则的性能改进，并且我将讨论我们在未来一年中可能看到的未来变化。

## 最近的改进

### `ovn-nbctl`妖魔化

我们的首要性能目标之一是确定支持 [OpenShift Online](https://www.openshift.com/products/online/) 规模的集群的可行性。根据一些预期的数字，我们设置了一些测试，在这些测试中，我们将按照预期的规模构建集群，然后模拟一次 pod 的创建和删除，看看 OVN 的表现如何。根据我们最初的测试，我们对结果不是很满意。随着规模的增长，对集群进行更改所需的时间越来越长。

在隔离组件并对它们进行分析之后，我们终于有了一个关于导致问题的原因的工作理论:ovn-nbctl。`ovn-nbctl`是一个命令行实用程序，允许与 OVN 北向数据库进行交互。该命令是 [OpenShift](http://openshift.com/) 构建其覆盖网络的主要机制。我能够创建一个 shell 脚本来模拟我们的规模测试的设置，但是只关注于`ovn-nbctl`调用。以下是脚本:

```
#!/bin/bash

# Create router
ovn-nbctl lr-add router

# Create switches
for i in $(seq 1 159); do
    j=$(printf '%02x' $i)
    ovn-nbctl ls-add ls$i
    ovn-nbctl lsp-add ls$i lsp-ro$i
    ovn-nbctl lrp-add router ro-lsp$i 00:00:00:00:00:$j 10.0.0.$i/16
    ovn-nbctl set Logical_Switch_Port lsp-ro$i options:router-port=ro-lsp$i type=router addresses=router
done

for i in $(seq 1 92); do
    for j in $(seq 1 159); do
        k=$(printf '%02x' $i)
        l=$(printf '%02x' $j)
        create_addrset=$((($j - 1) % 2))
        addrset_index=$((($j - 1) / 2))
        ovn-nbctl lsp-add ls$j lsp${j}_10.$i.0.$j
        ovn-nbctl lsp-set-addresses lsp${j}_10.$i.0.$j "00:00:00:00:$k:$l 10.$i.0.$j"
        if [ $create_addrset -eq 0 ]; then
            ovn-nbctl create Address_Set name=${i}_${addrset_index} addresses=10.$i.0.$j 1> /dev/null
        else
            ovn-nbctl add Address_Set ${i}_${addrset_index} addresses 10.$i.0.$j
        fi
        ovn-nbctl acl-add ls$j to-lport 1000 "outport == \"lsp${j}_10.$i.0.$j\" && ip4.src == \$${i}_${addrset_index}" allow
        ovn-nbctl acl-add ls$j to-lport 900 "outport == \"lsp${j}_10.$i.0.$j\"" drop
    done
done

```

脚本中的第一个循环创建了 159 台逻辑交换机，并将它们全部连接到一台逻辑路由器。下一组嵌套循环模拟将 pod 添加到 OpenShift 集群时执行的操作:

1.  其中一台交换机添加了一个交换机端口。
2.  交换机端口的地址被添加到地址集中。每个地址集由两个地址组成。因此，循环中的交替运行将创建一个新的地址集，或者将交换机端口的地址添加到在前一次循环迭代中创建的地址集中。
3.  为新端口创建 ACL。一个 ACL 允许流量从其地址集中的其它地址流向该端口。另一个丢弃所有其他流量。

这些环路导致 159 台交换机有 92 个端口，总共 14，628 个逻辑交换机端口。循环的每次迭代调用`ovn-nbctl`五次，这意味着总共有 73，460 次对`ovn-nbctl`的调用。

当我们运行该脚本时，结果如下:

```
$ time ./scale.sh 

real	759m27.270s
user	500m40.805s
sys	36m5.682s

```

单从那段时间很难下结论，但我认为公平地说，花超过 12 个小时来完成不好。所以我们来看看内循环的每次迭代需要多长时间。点击下面的图片放大。

[![How long iteration of the inner loop takes](img/7fe9ed931dc51d8e99f6403319ab2037.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/cluster_creation_resize.png)

正如您在图中看到的，随着测试的继续，完成一次循环迭代所花费的时间会增加。到最后，一次迭代需要超过 7 秒钟才能完成！这是为什么呢？

为了理解这个问题，让我们先仔细看看 OVSDB 客户机和服务器是如何工作的。OVSDB 客户端和服务器使用 JSONRPC 进行通信。为了避免在客户端代码中嵌入原始 JSONRPC，OVS 提供了一种基于 C 的 IDL(接口定义语言)。IDL 有两个责任。首先，在编译时，IDL 读取数据库的模式并生成本机 C 代码，以允许程序以类型安全的方式读取和操作数据库数据。其次，在运行时，IDL 充当 C 结构和 JSONRPC 之间的翻译器。

典型的 OVSDB 客户机首先确定它对哪个数据库感兴趣，对数据库中的哪些表感兴趣，以及对这些表中哪些列的值感兴趣。客户机向服务器发出一个请求，请求这些数据库、表和列的当前值。服务器用编码为 JSON 的当前值进行响应。然后，客户机的 IDL 将 JSON 翻译成本机 C 结构。客户端代码现在可以通过检查 C 结构来读取数据库的内容。如果客户机想要修改数据库，它可以制作自己的 C 数据，并将其传递给 IDL 进行处理。然后，IDL 将其转换为 JSONRPC 数据库事务，并发送给服务器。

IDL 还帮助数据库客户机处理来自服务器的消息。如果数据库中的数据发生变化，服务器可以向客户机发送一条更新 JSONRPC 消息。然后，IDL 使用这个 JSONRPC 用更新的信息修改 C 数据。

这对于长期运行的 OVSDB 客户机来说非常有效。在当前数据库的初始转储之后，客户机和服务器之间的交互以小块的形式发生。`ovn-nbctl`的问题在于它是一个短暂的过程。`ovn-nbctl`启动，从北向数据库请求所有数据，将 JSON 数据处理成 C 数据，然后通常在退出前执行一个操作。通过我们的分析，我们发现大多数时间都花在了`ovn-nbctl`处理启动时的初始数据库转储上。当您考虑到测试结束时正在处理的数据量时，这是有意义的。

我们创建的解决方案是让`ovn-nbctl`可以选择成为一个长期运行的流程。`ovn-nbctl`配备了一个选项，允许其 OVSDB 客户端部分在后台持续运行。对`ovn-nbctl`的进一步调用将命令传递给守护进程。然后，守护进程将命令的结果传回 CLI。通过这样做，OVSDB 客户机只需要从服务器转储一个数据库，然后逐步更新内容。`ovn-nbctl`流程可以运行得更快，因为它们不再需要每次都转储整个数据库。

这个解决方案最初是由 Red Hat 的 Jakub Sitnicki 实现的，后来由 VMware 的本·普法夫进行了改进。

将`ovn-nbctl`作为守护进程运行的实际机制很简单。您可以运行以下命令:

```
$ export OVN_NB_DAEMON=$(ovn-nbctl --pidfile --detach)

```

通过设置`OVN_NB_DAEMON`变量，对`ovn-nbctl`的任何进一步调用都将连接到守护进程。

有了这一改进，我们修改了前面的脚本，在开头添加了后台化行。运行修改后的脚本会产生以下结果:

```
$ time ./scale-daemon-new.sh 

real	5m9.950s
user	1m18.702s
sys	1m41.979s

```

那要快得多！事实上，速度快了 99%以上。这是每个循环迭代的图表。单击图表以放大它。

[![Faster iterations times after modifying the script](img/ccdbdedccd00fb4a932e5cef223baa9d.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/cluster_creation_daemonized_resize.png)

这比我们之前看到的要平坦得多。随着时间的推移，T2 仍然有轻微的增长。这是因为随着总数据量的增长，某些`ovn-nbctl`命令变得更加复杂。增长的规模比上一次要小得多。

### 端口组

在 OVN 测试中看到的另一个瓶颈是当 ACL 被大量使用时速度会大大降低。下面的脚本很好地说明了这个问题:

```
#!/bin/bash

NUM_PORTS=${1:-100}

ADDR_SET="\"10.1.1.1\""
for x in 1 2 3 4 ; do
    for y in {0..255} ; do
        ADDR_SET="${ADDR_SET},\"10.0.$x.$y\""
    done
done
ovn-nbctl create Address_Set name=set1 addresses=$ADDR_SET
ovn-nbctl list Address_Set

ovn-nbctl ls-add ls0
COUNT=0
while test ${COUNT} -lt ${NUM_PORTS} ; do
    echo "ovn lsp$COUNT"
    ovn-nbctl --wait=hv lsp-add ls0 lsp${COUNT}
    ovn-nbctl acl-add ls0 to-lport 1000 "outport == \"lsp${COUNT}\" && ip4.src == \$set1" allow
    ovs-vsctl add-port br-int port${COUNT} \
        -- set Interface port${COUNT} external_ids:iface-id=lsp${COUNT}
    COUNT=$[${COUNT} + 1]
done

```

让我们讨论一下剧本中发生了什么。首先，创建一个名为`set1`的地址集，其中有 1020 个 IP 地址(10.0.1.0 到 10.0.4.255)。然后我们创建一个逻辑交换机`ls0`并添加`NUM_PORTS`到其中。`NUM_PORTS`默认为 100，但是可以通过向脚本传递一个参数将其更改为任何值。在我们的测试中，我们对`NUM_PORTS`使用 1000。除了创建端口之外，我们还添加了一个新的 ACL，允许流量从我们的地址集中的任何地址流向这个新端口。这里还有两条重要的其他线

*   `ovn-nbctl lsp-add`命令包含`--wait=hv`。这意味着该命令将被阻塞，直到被所有虚拟机管理程序上运行的`ovn-controller`进程处理。在我们的例子中，我们运行在 OVS 沙盒中，所以只有一个管理程序。这意味着只需等待一个`ovn-controller`结束。
*   有一个`ovs-vsctl add-port`命令将逻辑交换机端口绑定到 OVS `br-int`桥。这使得 OpenFlow 可以由`ovn-controller`为这个特定的逻辑交换机端口生成。

这个脚本并不牵强。地址集中的地址数量可能比典型的部署要多，但是在使用 ACL 时，部署通常会使用这种模式。除了 ACL 适用的逻辑交换机端口之外，它们创建的 ACL 几乎完全相同。

让我们看看当我们运行此脚本并创建 1，000 个逻辑交换机端口时会发生什么。

```
$ time ./load_net.sh 1000

real	125m16.022s
user	2m47.103s
sys	0m11.376s

```

脚本需要两个多小时才能完成。像以前一样，很难仅仅根据时间来衡量任何事情。让我们看看当我们对循环中的每次迭代计时时会发生什么。点击图片放大。

[![Time taken for each iteration in the loop](img/95c9c3afa02f743fd935e52e6c25e0fd.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/acl_no_port_groups_resize.png)

与之前的`ovn-nbctl`问题一样，我们可以看到随着测试的进行，时间会稳步增加。随着网络获得的 ACL 越来越多，完成环路所需的时间也越来越长。这是为什么呢？

答案与`ovn-controller`生成 OpenFlow 的方式有关。尽管我们的 ACL 将 1020 个成员的地址集作为一个单元引用，但这并不能直接转化为 OpenFlow。相反，地址集必须扩展到每个单独的地址，并且每个单独的地址在单独的流中进行评估。当我们添加第一个端口时，`ovn-controller`为 OpenFlow 表 44 中的 ACL 生成类似于以下内容的 OpenFlow:

```
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.1 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.2 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.3 actions=resubmit(,45)
...
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.255.255 actions=resubmit(,45)

```

作为解释:

*   表 44 位于逻辑交换机出口管道上，属于`to-lport`ACL 的流就是在这里写入的。
*   `metadata`是数据包数据路径的标识符。在这种情况下，0x1 指的是我们的逻辑开关。
*   `reg15`是存储输出端口号的 OpenFlow 寄存器。在这种情况下，0x1 指的是我们添加的第一个逻辑交换机端口。该流检查输出端口，因为我们的 ACL 是一个`to-lport` ACL。如果它是一个`from-lport` ACL，那么我们将检查输入端口，并且我们将在一个不同的 OpenFlow 表中这样做。
*   `nw_src`是网络层源地址。
*   `resubmit(,45)`动作允许处理在开放流表 45 继续。在这种情况下，操作是“重新提交”,因为我们 ACL 上有一个“允许”操作。如果它是“drop”，那么流应该是`actions=drop`。

在这种情况下，我们创建了 1，020 个流，每个流对应地址集中的一个地址。现在，让我们看看添加第二个端口时会发生什么:

```
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.1 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x2,nw_src=10.0.1.1 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.2 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x2,nw_src=10.0.1.2 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.3 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x2,nw_src=10.0.1.3 actions=resubmit(,45)
...
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.4.255 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x2,nw_src=10.0.4.255 actions=resubmit(,45)

```

我们现在已经添加了一组相同的流，除了`reg15`匹配`0x2`而不是`0x1`。换句话说，我们为添加的新端口创建了另外 1，020 个流。你可能会注意到一个模式的形成。表 44 中产生的流数量大约是端口数量乘以地址集中的地址数量。在完成脚本后，让我们计算一下表中的流总数:

```
$ ovs-ofctl dump-flows br-int | wc -l
1033699
$ ovs-ofctl dump-flows br-int | grep table=44 | wc -l
1025001

```

流量过百万，表 44 占总数的 99%。请记住，在我们的脚本中，我们在添加逻辑交换机端口时使用了`--wait=hv`。这意味着每当我们添加新的交换机端口时，我们必须等待`ovn-controller`生成每个流。由于该表每次迭代都会增加 1，020+个流，因此添加一个端口最终需要几十秒钟。

这个问题的解决方案是尽量减少生成的流的数量。在 OVS 的 OpenFlow 实现中实际上有一个内置的构造，叫做*合取匹配*。合取匹配允许将存在于同一个表中并且具有相同结果动作的 OpenFlow 规则组合成更紧凑的形式。如果这些规则具有相似的匹配标准，则可以制作属于公共匹配标准的第一部分的流的列表，接着是属于第二部分的流的列表，等等，直到所有部分都匹配。

考虑前一组流。所有的流都存在于表 44 中，并且具有向表 45 重新提交的相同动作。所有的数据流首先在一个端口号上匹配，然后在一组 IP 地址上匹配。我们想要的是两部分的合取匹配。第一部分匹配每个有效端口，第二部分匹配地址集中的每个 IP 地址。每次我们添加一个新端口时，合取匹配的第一部分都会添加一个新的流，但是第二部分保持不变。这样，表 44 中的流数量可以近似为端口数量加上地址集中的地址数量。

为了生成合取匹配，提出了几种想法。最终胜出的是增加了一个名为*端口组*的特性。端口组是一个简单的结构，它允许通过一个集合名称来引用多个逻辑交换机端口。可以在 ACL 中引用端口组，而不是通常引用单个端口。端口组还有其他不错的特性，但是这些超出了本文的范围。

端口组最初是由易贝的周瀚实现的，但是后来许多其他贡献者充实并改进了这个特性。

现在让我们使用端口组重写脚本。主要区别在于，我们将定义一个指向端口组的 ACL，而不是为我们添加的每个端口定义新的 ACL。当我们创建新的逻辑交换机端口时，我们会将该端口添加到端口组中。将所有端口和 IP 地址表示在一个 ACL 中应该允许创建合取匹配。以下是生成的脚本:

```
#!/bin/bash

NUM_PORTS=${1:-100}

ADDR_SET="\"10.1.1.1\""    
for x in 1 2 3 4 ; do                         
    for y in {0..255} ; do
        ADDR_SET="${ADDR_SET},\"10.0.$x.$y\""
    done                                                   
done                      
ovn-nbctl create Address_Set name=set1 addresses=$ADDR_SET                                               
ovn-nbctl list Address_Set                                                                               

ovn-nbctl ls-add ls0                                                                                     
ovn-nbctl create Port_Group name=pg1                                                           
ovn-nbctl acl-add ls0 to-lport 1000 "outport == @pg1 && ip4.src == \$set1" allow                         
ovn-nbctl acl-list ls0                                                                                   
COUNT=0                                                                                                  
while test ${COUNT} -lt ${NUM_PORTS} ; do                                                                
    echo "ovn lsp$COUNT"                                                                                 
    ovn-nbctl --wait=hv lsp-add ls0 lsp${COUNT}                                                          
    port=$(ovn-nbctl get Logical_Switch_Port lsp${COUNT} _uuid)                                          
    ovn-nbctl add Port_Group pg1 ports $port                                                          
    ovs-vsctl add-port br-int port${COUNT} \                                                             
        -- set Interface port${COUNT} external_ids:iface-id=lsp${COUNT}                                  
    COUNT=$[${COUNT} + 1]                                                                                
done   

```

如你所见，构造基本相似。注意，在定义 ACL 时，我们使用`@`符号来指代端口组`pg1`。这让 OVN 知道我们指的是一个端口组，而不是我们 ACL 中的单个端口。让我们看看当我们运行此脚本并创建 1，000 个逻辑交换机端口时会发生什么。

```
$ time ./load_net_pg.sh 1000

real	6m23.696s
user	3m15.920s
sys	0m10.849s

```

现在，该脚本只需大约 6 分钟即可完成。这是 95%的改进！这是循环每次迭代的图表。点击图片放大。

[![Graph of each iteration of the loop](img/20fd96abe44af3adf222a1975e09940f.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/acl_port_groups_resize.png)

每次迭代的时间都在增加，但是如果你看看 y 轴上的刻度，你会发现时间要少得多。客观地说，这是两张相互重叠的图。点击图片放大。

[![Two graphs superimposed on each other](img/10a8be3d68916c2c4b1ec481ba95d96f.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/acl_timing_resize.png)

如果你眯着眼睛看，你可以看到蓝色的条开始出现在图表的右下角...

让我们来看看每一步产生的流程。添加一个交换机端口后，OpenFlow 如下所示:

```
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.1 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.2 actions=resubmit(,45)
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.1.3 actions=resubmit(,45)
...
priority=2000,ip,metadata=0x1,reg15=0x1,nw_src=10.0.4.255 actions=resubmit(,45)

```

到目前为止，和我们不使用端口组的时候完全一样。现在让我们添加第二个端口。

```
priority=2000,conj_id=2,ip,metadata=0x1 actions=resubmit(,45)

priority=2000,ip,metadata=0x1,nw_src=10.0.1.1 actions=conjunction(2,1/2)
priority=2000,ip,metadata=0x1,nw_src=10.0.1.2 actions=conjunction(2,1/2)
priority=2000,ip,metadata=0x1,nw_src=10.0.1.3 actions=conjunction(2,1/2)
...
priority=2000,ip,metadata=0x1,nw_src=10.0.4.255 actions=conjunction(2,1/2)

priority=2000,ip,reg15=0x1,metadata=0x1 actions=conjunction(2,2/2)
priority=2000,ip,reg15=0x2,metadata=0x1 actions=conjunction(2,2/2)

```

这就是我们所追求的合取匹配！第一行创建合取匹配。合取匹配的 ID 是 2，它表示如果合取匹配的所有要求都得到满足，那么操作是重新提交到表 45。其余的几行定义了合取匹配及其`conjunction`动作的要求。逗号前的 2 表示它们是 ID 为 2 的合取匹配的要求。逗号后面的数字表示合取匹配的需求编号和需求总数。所有以`1/2`结尾的流都属于源 IP 地址。所有带有`2/2`的流都属于输出逻辑交换机端口。合取匹配产生相同的逻辑动作集，但以更紧凑的方式表达。

在我们的网络加载了 1，000 个交换机端口后，让我们检查生成的 OpenFlow 流的总数。

```
$ ovs-ofctl dump-flows br-int | wc -l
9098
$ ovs-ofctl dump-flows br-int | table=44 | wc -l
2027

```

这比以前少了三个数量级。现在，表 44 仅占总流量的 22%。这使得`ovn-controller`产生流量所需的时间有天壤之别。

### 混合优化

您可能已经注意到，在我们的 ACL 生成脚本中，我们没有使用后台化的`ovn-nbctl`。我们在第一节中讨论了这是一个多么伟大的优化，现在让我们看看它是如何产生影响的。

```
$ time ./load_net_pg_daemon.sh
real	3m50.670s
user	0m20.340s
sys	0m8.608s

```

还不错！与非守护进程版本相比，我们又获得了 40%的加速。

既然我们在这里跨流，那么如果我们从`ovn-nbctl` daemonization 部分获取脚本并对其使用端口组，会发生什么呢？结果如下:

```
$ time ./scale-daemon-new-pg.sh
real 3m33.457s
user 1m1.614s
sys 1m24.891s

```

有 31%的加速，但这不是由于我们之前看到的合取匹配的原因。由于脚本中没有`--wait=hv`，所以我们不会等待`ovn-controller`生成流。速度加快的主要原因是因为使用端口组需要比以前更少的对`ovn-nbctl`的调用。

## 未来的改进

### 增量处理

您可能已经从 port groups 部分注意到了一个事实，即`ovn-controller`必须在每次读取南行数据库的内容时生成整个 OpenFlow 表。`ovn-northd`的工作方式类似:它总是读取整个北向数据库，以便生成一组完整的新的南向数据。这种操作方式有一些独特的优势。

*   代码很容易推理。
*   在暂时连接失败的情况下，该代码是有弹性的。

掩盖这些优势的是，这种方法速度慢，计算量大，并且随着数据集大小的增加，情况会变得更糟。通过处理对数据库的更改，而不是数据库中的所有内容，可以获得巨大的性能提升。

这在实践中是很难做到的。已经多次尝试重构`ovn-controller`来增量处理结果。这些实现最常见的问题是增加了维护代码的难度。所有尝试中常见的一个问题是，C 没有提供实现增量处理引擎的最简单方法。

在检查了过去尝试的问题并了解了前进的最佳方式后，VMware 的工程师们开始努力用不同于 c 的语言重写 OVN 的部分内容。他们创建了一种称为 [Differential Datalog](https://github.com/ryzhyk/differential-datalog) 的语言，通常缩写为 DDlog。DDlog 的核心是一个增量数据库处理引擎。这正是获得更高性能处理所需要的。本·普法夫向 ovs-discuse 邮件列表发送了一封很好的[电子邮件](https://mail.openvswitch.org/pipermail/ovs-discuss/2018-November/047665.html)，其中包含项目摘要。

那么，我们可以从增量处理中得到什么好处呢？易贝的周瀚为`ovn-controller`组装了一个增量处理的概念验证 C 版本。在我和他进行的测试中，我们发现`ovn-controller`减少了大约 90%的 CPU 使用。我们还发现`ovn-controller`的总体运行速度提高了大约 90%。与前面讨论的优化不同，这并不适用于特定的用例，而是通过`ovn-controller`为*所有的*操作提供优化。DDlog 在`ovn-northd`上的实验结果显示了类似的改进。Leonid Ryzhyk 的这个[演讲](https://ovsorbit.org/episode-58-slides.pdf)提供了一些图表，展示了在所有集群规模下，DDlog 的增量计算相对于当前 C 实现的巨大加速。

向 DDlog 的转换正在进行中。目的是在 2.11 版本发布前完成`ovn-northd`的 DDlog 并集成到 OVN 中。一旦实现下降，我们鼓励每个人部署它，并告诉我们您看到的性能改进。对于那些对在实际环境中部署重写的 OVN 组件犹豫不决的人，不要担心！`ovn-northd`的 C 实现仍然会存在，您可以选择继续使用它。但是您将错过 DDlog 实现的惊人性能改进。

### 其他未来改进

增量处理是所有未来改进的基础。然而，即使使用增量处理，我们也可以看到一些未来的小改进。以下是可能改进的简要列表。

#### 增量流程处理

`ovn-controller`当前创建它想要安装在 OVS 的所有期望的流的集合。它还维护当前安装在 OVS 的所有流的集合。在每次迭代中，`ovn-controller`必须确定两个集合之间的差异，然后向 OVS 发送适当的修改消息。有了增量处理，很自然地，我们也可以增量地计算要安装的流量。当测试周瀚的增量处理的 C 实现时，期望的和安装的流之间的比较是 CPU 时间的新的顶级用户。

#### 直接传递增量更改

一旦增量处理就绪，`ovn-northd`将计算对南行数据库进行的一些更改，并进行该更改。`ovn-controller`将获取南行数据库内容，确定发生了什么变化，并对这些增量变化采取行动。如果`ovn-northd`已经计算出南行数据库的变化，也许有一种方法可以直接在`ovn-northd`和`ovn-controller`之间传递这些变化。这可以消除两个守护进程中的一些重复处理。还可以节省南行数据库将占用的一些硬盘空间。

在可能做出的改进中，这可能是最难做对的。这是因为多个`ovn-controller`守护程序连接到 OVN 南行数据库。因此，试图计算通用的变化增量意味着`ovn-controller`很容易错过更新。

即使不能对所有南行数据都做到这一点，也许可以专门对南行逻辑流表做一些事情。该表往往会变得比任何其他南行表都大。

#### 更好的合取匹配生成

我们之前提到了合取匹配如何极大地帮助减少了由`ovn-controller`安装的流的数量。`ovn-controller`中的表达式解析器在应该生成合取匹配的情况下很难生成合取匹配。这需要对由此产生的流动进行比目前在`ovn-controller`更复杂的分析。如果表达式解析器可以变得更智能，那么可能会节省一些生成的流的数量。

#### 集中表达式解析

目前，`ovn-controller`从南行数据库中读取逻辑流并解析表达式，以便为 OVS 生成流。虽然有一些逻辑流的最终解析形式在不同的虚拟机管理程序之间会有所不同，但是无论在哪里解析，大多数解析表达式都是完全相同的。或许可以通过集中解析表达式来节省一些虚拟机管理程序的计算能力。

## 最后的想法

OVN 的发展正进入一个激动人心的时期。能够如此大规模地提高性能是软件成熟的一个很好的标志。随着增量处理的引入，我相信控制平面性能问题将完全消失。对于任何希望在其环境中使用 OVS 的人来说，使用 OVN 将成为一种自然的选择，只增加很少的开销。

我怀疑随着关于性能改进的消息传出，OVN 的采用会增加更多。越来越多的采用意味着除了不再需要关注性能，我们可以期待在不久的将来看到一系列新功能添加到 OVN。

如果你有兴趣投稿，我强烈建议你现在就参与进来。这可能是一个新的 OVN 特色的黄金时代的开始。

## 额外资源

*   [打开红帽开发者博客上的虚拟网络文章](/blog/2018/11/08/how-to-create-an-open-virtual-network-distributed-gateway-router "How to create an Open Virtual Network distributed gateway router")
*   [打开 Red Hat 开发者博客上的 vSwitch 文章](/blog/2017/06/05/measuring-and-comparing-open-vswitch-performance "Measuring and comparing Open vSwitch performance")
*   [如何创建一个开放的虚拟网络分布式网关路由器](https://developers.redhat.com/blog/2018/11/08/how-to-create-an-open-virtual-network-distributed-gateway-router/)
*   [OVN 的 IP 包缓冲](https://developers.redhat.com/blog/2018/12/07/ip-packet-buffering-in-ovn/)
*   [开放虚拟网络中的动态 IP 地址管理(OVN):第一部分](https://developers.redhat.com/blog/2018/09/03/ovn-dynamic-ip-address-management/)
*   [使用 eBPF (RHEL 8 Beta)进行网络调试](https://developers.redhat.com/blog/2018/12/03/network-debugging-with-ebpf/)
*   [虚拟网络 Linux 接口介绍](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/)
*   [在打开的 vSwitch 中排除 FDB 表缠绕故障](https://developers.redhat.com/blog/2018/09/19/troubleshooting-fdb-table-wrapping-in-open-vswitch/)
*   [故障排除打开 vSwitch DPDK PMD 线程核心关联](https://developers.redhat.com/blog/2018/06/20/troubleshooting-open-vswitch-dpdk-pmd-thread-core-affinity/)
*   [使用 Open vSwitch DPDK 调试内存问题](https://developers.redhat.com/blog/2018/06/14/debugging-ovs-dpdk-memory-issues/)
*   [打开 v switch:802.1 ad(QinQ)支持概述](https://developers.redhat.com/blog/2017/06/06/open-vswitch-overview-of-802-1ad-qinq-support/)

*Last updated: February 17, 2022*