# 优化 iptables-nft 大型规则集在用户空间的性能

> 原文：<https://developers.redhat.com/blog/2020/04/27/optimizing-iptables-nft-large-ruleset-performance-in-user-space>

在检查 Linux 防火墙性能时，数据包处理还有第二个方面——即防火墙设置操作的成本。在容器的世界中，不同的网络节点产生得足够快，以至于防火墙规则集调整延迟成为一个重要因素。与此同时，考虑到即使是中等规模的服务器也可能托管大量的容器，规则集往往会变得非常庞大。

在过去，为了加速大型规则集的处理，在 legacy `iptables`上投入了相当大的努力。随着最近上游和下游将`iptables-nft`确立为标准变型，对这种质量的重新评估是适当的。为了看看事情到底有多糟糕，我创建了[一堆基准](http://nwl.cc/cgi-bin/git/gitweb.cgi?p=ipt-sbs-bench.git;a=summary)来运行两种变体并比较结果。

## 用于测试的基准

**免责声明:**除了一个测试之外，接下来的所有测试都是由不经常处理大型高性能设置的人创建的。相反，我的重点是测试代码的各个方面，因为我知道不同命令背后的实现。

这些基准中的每一个都运行多次，并且记录平均变化以建立对结果的信心。运行相同的基准测试来增加规则集的大小会给结果增加第二个维度—比例因子。以下测试描述使用“N”表示被缩放的值:

1.  Measure `iptables-restore`运行时加载转储文件:
    1.  规则
    2.  链条
    3.  200 条链，每条链包含 N 条规则
2.  在一个在`filter`表的链 X 中有 N 个规则的规则集中，测量:
    1.  将规则附加到`filter`表的链 x 上。
    2.  删除`filter`表的链 X 中的最后一条规则(根据规则规范)。
    3.  在`filter`表的链 x 的顶部插入一条规则。
    4.  将规则附加到`filter`表的空链 y。
    5.  再次从`filter`表的链 Y 中删除该规则。
    6.  将规则附加到`nat`表的链 x 上。
    7.  再次从`nat`表的链 X 中删除该规则。
    8.  列出了`filter`表的链 x 中的第一条规则。
    9.  列出`filter`表的链 x 中的最后一条规则。
    10.  列出`filter`表的空链 y。
    11.  列出`nat`表的空链 x。
    12.  冲洗`filter`工作台的空链 y。
    13.  清空`nat`工作台的空链 x。
    14.  冲洗`filter`工作台的链条 x。
3.  在具有 N 个链的规则集中，测量:
    *   冲洗所有链条。
4.  在一个有 200 个链的规则集中，每个链包含 N 个规则，测量:
    *   冲洗所有链条。
5.  在包含 N 组规则集加载项的规则集中，这 N 组加载项由两个用户定义的
    链(每个链包含一个和两个规则)以及附加到公共链的两个规则组成，测量:
    *   如上所述添加另一个 ruleset 附加组件。

这最后一个基准应该是一个“真实世界的例子”它源于 Kubernetes，据称类似于容器启动时发生的防火墙配置更改。不那么无聊的方面，在我看来就是使用了`iptables-restore`的`--noflush`选项。这种策略是批处理几个`iptables`命令的常用技巧，因此可以减少程序启动和缓存开销，通常用于性能关键的情况。但是，与此同时，优化它的性能是一个更大的挑战，因为在程序启动时，不仅内核规则集的内容是未知的，用户输入也是未知的。

## 基线结果

最初的测试结果令人清醒。传统的`iptables`性能在几乎所有情况下都优于`iptables-nft`，但是常规的`iptables-restore`测试(1.1 到 1.3)都在很近的范围内，如图 1 所示。

[![test1.1 initial results](img/279558c3e7f4f36588e93c425e09c972.png "test1.1_initial")](/sites/default/files/blog/2020/02/test1.1_initial.png)initial results of test 1.1

Figure 1: The initial results from test 1.1.

正如您在图 2 和图 3 中看到的，其余的从“表现稍差”到“糟糕”不等。

[![test2.1 initial results](img/72eb0c62405a514867096857868f16f5.png "test2.1_initial")](/sites/default/files/blog/2020/02/test2.1_initial.png)initial results of test 2.1

Figure 2: The initial results from test 2.1.

[![test2.6 initial results](img/c69e105721aaaa4159062006dc7ec812.png "test2.6_initial")](/sites/default/files/blog/2020/02/test2.6_initial.png)initial results of test 2.6

Figure 3: The initial results from test 2.6.

我怀疑为什么`iptables-nft`表现如此糟糕的主要原因是内核规则集缓存和从`libnftnl`数据结构中的`nftables`规则到`libxtables`数据结构中的`iptables`规则的内部转换。后者是难以避免的，因为`iptables-nft`与传统的`iptables`共享大部分解析器，所以我专注于改进缓存算法。

## 最大化接收缓冲区

内核规则集通过`netlink`一次获取 16KB 的块，由用户空间缓冲区决定。然而，内核支持一次转储两倍的数据，所需要做的只是将用户空间接收缓冲区大小增加一倍——这是一个简单的修复，在涉及缓存的大规模测试中提高了性能，在理想情况下提高了 50%。

图 4 是一个很好的例子，展示了随着规则集的增加，性能如何不断提高。蓝色曲线倾斜度的降低也表明缩放比例略有改善，尽管总体上仍然比传统的`iptables`差得多。

[![](img/8a43f405e27ca57681d13231ecaf8468.png "test2.1_v1.8.3-059-gb5cb6e631c828")](/sites/default/files/blog/2020/02/test2.1_v1.8.3-059-gb5cb6e631c828.png)results of test2.1 in v1.8.3-059-gb5cb6e631c828

Figure 4: The results from test2.1 in v1.8.3-059-gb5cb6e631c828.

## 缓存链而不是规则

第一个真正的缓存优化是在不需要的时候跳过获取规则。以测试 2.1 和 2.3 作为简单的例子，没有理由规则`append`或`insert`(在链的开始)的运行时间应该依赖于该链中已经存在的规则的数量。

实现这个特性是一项简单的任务。`iptables-nft`中的后端代码具有函数`nft_chain_list_get`,该函数应该返回一个给定表包含的链表。对`nft_chain_list_get`函数的调用隐式触发了完全缓存填充(除非缓存已经完全填充)。我修改了这个函数，只获取表中的链表，并在需要时让调用者请求全缓存更新。

令我惊讶的是，这一变化显著地改善了八个基准测试的结果。其中一半开始显示 O(1)复杂度(即，不考虑缩放值的恒定运行时间)。具体而言，结果如下:

*   **测试 2.1、2.3、2.14 和 4.1:** 运行时仍然依赖于缩放，但性能始终优于传统`iptables`。在这些情况下，瓶颈在内核端，用户空间已经很完善了。
*   **测试 2.6 和 2.13:** 恒定运行时间，性能与传统相当。这表明传统的`iptables`在某些情况下可以避免缓存，并且它的缓存是针对每个表的。
*   **测试 2.4 和 2.12:** 恒定运行时间，遗留性能取决于缩放值。这些是真正的宝石，在用户和内核空间都是无用的，只是被遗留系统中不灵活的缓存延迟了。

## 每个链的选择性规则缓存

以前的变化的结果是一个很好的动机，甚至进一步向那个方向推进，即不仅要决定是否需要一个规则缓存，还要决定哪个(哪些)链需要。这样，大型链就不会像在测试 2.5、2.7 和 2.11 中那样减慢其他链上需要规则缓存的操作。

实现这种改变感觉很自然，因为有代码路径接受可选的链名，比如`--list`命令。我的方法是扩展`nft_chain_list_get`函数的签名来接受这个链名。如果为非 NULL，被调用的代码将只从内核中获取特定的链，而不是每个表的完整链转储。因此，返回的链表只包含一个条目。除此之外，我还更改了完整的缓存更新例程(`nft_build_cache`)来接受可选的链名，因此规则获取也可以在每个链中进行。

上面的缺点是代码必须知道部分缓存。如果在部分缓存更新之后进行完全缓存更新，我们最终会得到重复的条目。因此，将链插入缓存的例程必须跳过已经存在的链。这个逻辑对于规则是不可能的，所以对于已经包含规则的链，规则缓存更新被简单地跳过。

## 简化兼容性检查

一个有趣的问题是，以前的优化未能改善 test 2.10，尽管这应该包括在内。原因隐藏在为“list”命令执行的健全性检查中，因为`iptables-nft`需要它所关心的规则集部分处于兼容状态:基链需要有正确的名称和优先级，规则不能包含未知的表达式，等等。

这个规则集“preview”有点笨拙，因为它应用于所有的表格内容，而不仅仅是相关的内容。在进行了更细粒度的测试之后，test 2.10 也开始给出预期的结果，如图 5 所示。

[![](img/3a429b1807b6e9d66369e40760f9c779.png "test2.10_v1.8.3-064-g48a21d5c7af07")](/sites/default/files/blog/2020/02/test2.10_v1.8.3-064-g48a21d5c7af07.png)results of test 2.10 in v1.8.3-064-g48a21d5c7af07

Figure 5: The results of test 2.10 in v1.8.3-064-g48a21d5c7af07.

## 优化 flush 命令

正如所料，刷新表中的每个链的性能很差，因为有大量的链，因为`iptables-nft`必须获取完整的链列表并为每个链创建刷新请求。与其说是有意的，不如说是巧合，我注意到内核已经支持一次刷新一个表的所有链，也就是通过从各自的`netlink`消息中省略链名属性。

如果不是详细模式，这将是一个微不足道的修复。但是由于冗余模式下的传统`iptables`打印每个刷新的链的名称，所以在用户通过`--verbose`的情况下，`iptables-nft`必须回退到旧的逻辑。

如图 6 所示，测试 3.1 中的缩放看起来仍然有点“滑稽”，但这可能是由于内核代码缩放不如用户空间。

[![](img/c342944844940d01d26d287d7aa3f445.png "test3.1_v1.8.3-065-gc41b98babd55f")](/sites/default/files/blog/2020/02/test3.1_v1.8.3-065-gc41b98babd55f.png)results of test 3.1 in v1.8.3-065-gc41b98babd55f

Figure 6: The results of test 3.1 in v1.8.3-065-gc41b98babd55f.

## iptables-nft-restore - noflush

与其他工具相比，`iptables-nft-restore`中的排序缓存管理需要更多的努力。这个问题主要是由输入未知这一事实引起的。假设在处理了几个不需要规则缓存的命令(例如简单的`--append`命令)之后，输入包含一个需要规则缓存的命令(例如，- insert with index)。这种情况意味着之前添加的规则必须与事后从内核获取的规则合并。有了插入规则(在开头)或附加规则(在结尾)的选项，提取的规则可能会位于之前添加的规则之间。

为了避免这个问题以及可能与不一致的缓存相关的其他问题，我决定尝试一种不同的方法，将输入缓冲到一定的量。可能会在 commit 09cb 517949 e 69(" xtables-restore:提高- noflush 操作的性能")中找到相关的更改。在这种情况下，在测试输入行的缓存需求后，使用 64KB 的缓冲区来存储输入行。这个过程一直持续到:

*   缓冲区空间耗尽。
*   输入流表示文件结束。
*   读取了需要规则缓存的命令。

有了输入缓冲，test 5.1 开始显示完美的结果，如图 7 所示。

[![results of test 5.1 in v1.8.3-094-g09cb517949e69](img/c0c6509453c19ee05a2973e1c01a3fd0.png "test5.1_v1.8.3-094-g09cb517949e69")](/sites/default/files/blog/2020/02/test5.1_v1.8.3-094-g09cb517949e69.png)results of test 5.1 in v1.8.3-094-g09cb517949e69

Figure 7: The results of test 5.1 in v1.8.3-094-g09cb517949e69.

## 零碎材料

一些测试用例仍然没有优化，这意味着在那些情况下`iptables-nft`的性能比 legacy 的性能差。原因如下:测试 2.2、2.8 和 2.9 很慢，因为`nftables`通过句柄识别规则，并且在任何情况下都需要从 rulespec 或 index 进行转换。由于`nftables`不支持获取特定的规则(例如，通过索引)，用户空间在这里没有优化的余地。一个部分的解决方法是在内核中通过索引实现规则识别，但是在过去，这种方法由于其固有的并发性问题而不被认可。

所有的(刷新)恢复测试(1.1、1.2 和 1.3)仍然很慢，但是这听起来比实际情况更糟，请看图 8 中 1.3 的初始测试结果。

[![test1.3 initial results](img/6f26e9096ee1f35eaa5c5d456531756f.png "test1.3_initial")](/sites/default/files/blog/2020/02/test1.3_initial.png)initial results of test 1.3

Figure 8: The initial results from test 1.3.

这些结果表明`iptables-nft`慢了大约 1.2 到 1.7 倍。换句话说，恢复包含 200k 多一点的行的转储需要 1.7 秒而不是 1.0 秒。除了非常相似的缩放，我怀疑罪魁祸首是稍微不太优化的代码，或者数据必须打包到`netlink`消息中，而不是通过`ioctl`一次性复制到内核。

## 摘要

好消息是，在大多数情况下,`iptables-nft`的表现远远超过了传统的`iptables`,在其他情况下也没有太大的损失。这个测试也显示了`nftables`内核代码为高负载做了多么充分的准备，尽管使用内核的`xtables`扩展的`nft_compat`模块有些不雅。因此，虽然迁移到`nftables`并利用额外的好处仍然是获得最高性能的最佳途径，但在遗留应用程序背后交换`iptables`变体可能是一种有效的性能方案。

当然，所有这些结果都不能全信。一方面，我肯定忘记了具体的用例——我所有的测试加起来可能都没有达到完整的代码覆盖率。另一方面，一个复杂的规则集利用了遗留的`iptables`所能提供的(包括`ipset`)将有可能从一开始就防止用户空间工具性能成为问题。

*Last updated: June 29, 2020*