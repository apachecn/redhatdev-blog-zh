# 分析 Linux 内核及其模块之间二进制接口的变化

> 原文：<https://developers.redhat.com/blog/2018/03/28/analyzing-binary-interface-changes-linux-kernel>

本文面向对长期 Linux 内核维护感兴趣的人。它向您介绍了一些工具，这些工具有助于在假定稳定的内核的整个生命周期中，在修改代码时，保持内核及其可加载模块之间的二进制接口稳定。由于这些工具本质上是分析工具，它们不仅可以被内核开发人员使用，也可以被质量保证工程师和高级内核用户(系统程序员)使用。

### 上游树内内核模块:理想情况

在 Linux 内核的规范开发模型中，所有动态加载模块的源代码都与核心内核的源代码放在一起。在这种模型中，每当核心内核改变它向其模块公开的接口时，编译器会检测到接口发生了变化，从而很容易相应地调整模块的代码。

这个模型被称为 *in-tree 内核模块*模型，内核与其模块之间的二进制接口被称为*内核模块接口*，或 KMI。在树内内核模块开发模型中，当 KMI 改变时，不需要做任何事情。模块源代码的修改是自然发生的。我相信这是最好的模式。内核模块的源代码应该就在上游。

### 树外内核模块:生活有时会很艰难

内核模块的源代码应该就在上游。我喜欢那句话。如果现实生活一直如此，那就太好了。

但事实并非如此。

有些情况下，有些人可能会开发一个内核模块，而不把它的源代码上传，至少在一段时间内是这样。我把这种情况称为*树外内核模块*模型。

在这个模型中，假设内核模块最初是针对版本号为 3.10 的上游内核所公开的接口开发的。当上游内核迁移到版本 3.11 时，模块的作者可能想知道开发该模块所针对的二进制 KMI 是否发生了变化。如果是的话，有什么变化。有了这些知识，模块作者可以评估是否需要重新编译他们的模块，使其与新的 3.11 内核兼容。

同样，Linux 发行商可能希望确保他们发布的稳定内核的更新不会以不兼容的方式修改初始 KMI。为此，帮助可视化和审查二进制 KMI 变更的工具非常有用。

最终，稳定内核的用户可能会开发私有的、非分布式的模块供自己使用。他们可能会非常感激对稳定内核的更新不会破坏他们的私有模块。

### 可视化 KMI 的变化

本文向您展示了如何可视化和分析 Linux 发行版中可用的两个二进制内核包(即 [CentOS](https://www.centos.org/) )之间的 KMI 变化。

实际上，一组函数和全局变量(以及它们的类型)定义了要在两个二进制内核包之间进行比较的二进制 KMI。

请注意，Red Hat 在名为“ *kabi 白名单文件*”的文本文件中定义了二进制 KMI。如果您在运行 x86_64 机器的 [RHEL](https://access.redhat.com/products/red-hat-enterprise-linux/) 或 [CentOS](https://www.centos.org/) 7 系统上，您可以在*/lib/modules/kabi-rhel 70/kabi _ 白名单 _x86_64* 找到它。在这些系统上，该文件包含在名为[的 RPM 内核-ABI-白名单](http://vault.centos.org/7.0.1406/os/x86_64/Packages/kernel-abi-whitelists-3.10.0-123.el7.noarch.rpm)中。

为了执行实际的 KMI 比较，这个例子使用了 [abipkgdiff 工具](https://sourceware.org/libabigail/manual/abipkgdiff.html)，它基于 [Libabigail 框架](http://sourceware.org/libabigail)。这个框架最近得到了改进，可以分析 Linux 内核 ELF 二进制文件。结果， *abipkgdiff* 工具得到了改进，可以支持 RPM 格式的 Linux 内核包。

## 为 KMI 变化分析设置您的环境

### 获取 Libabigail 工具

这个例子假设你要么使用的是 CentOS 6 或 7、[RHEL 6 或 7，要么使用的是受支持版本的](https://access.redhat.com/products/red-hat-enterprise-linux/) [Fedora Linux](https://getfedora.org/) 。

要在 [CentOS](https://www.centos.org/) 或 [RHEL](https://access.redhat.com/products/red-hat-enterprise-linux/) 上安装 Libabigail 工具，您需要键入:

> ```
> $ yum install libabigail
> ```

在受支持的 Fedora 发行版上，您需要键入:

> ```
> $ dnf install libabigail
> ```

### 获取要比较的内核包

[abipkgdiff](https://sourceware.org/libabigail/manual/abipkgdiff.html) 需要两个内核包来比较 KMI，还有一些辅助包；即相关的调试信息和 KMI 定义包。

下面介绍如何比较 [CentOS 7.0](http://vault.centos.org/7.0.1406/) 和 [CentOS 7.1](http://vault.centos.org/7.1.1503) 内核的二进制 kmi。

首先获取 7.0 内核包:

> ```
> $ mkdir 7.0
> $ pushd . 
> $ cd 7.0 
> $ wget http://vault.centos.org/7.0.1406/os/x86_64/Packages/kernel-3.10.0-123.el7.x86_64.rpm \ 
>        http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-3.10.0-123.el7.x86_64.rpm \ 
>        http://vault.centos.org/7.0.1406/os/x86_64/Packages/kernel-abi-whitelists-3.10.0-123.el7.noarch.rpm
> $ popd
> ```

接下来，获取 7.1 内核包:

> ```
> $ mkdir 7.1
> $ pushd . 
> $ cd 7.1
> $ wget http://vault.centos.org/7.1.1503/os/x86_64/Packages/kernel-3.10.0-229.el7.x86_64.rpm \
>        http://vault.centos.org/7.1.1503/os/x86_64/Packages/kernel-abi-whitelists-3.10.0-229.el7.noarch.rpm \
>        http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-3.10.0-229.el7.x86_64.rpm
> $ popd
> ```

## 比较 kmi

现在比较这两个内核的 KMI，并将其保存到一个名为 *kernel.kmidiff.txt* 的文件中。为此，您需要键入以下命令:

> ```
> $ abipkgdiff --d1 7.0/kernel-debuginfo-3.10.0-123.el7.x86_64.rpm \
>              --d2 7.1/kernel-debuginfo-3.10.0-229.el7.x86_64.rpm \
>              --wp 7.1/kernel-abi-whitelists-3.10.0-229.el7.noarch.rpm \
>                   7.0/kernel-3.10.0-123.el7.x86_64.rpm \
>                   7.1/kernel-3.10.0-229.el7.x86_64.rpm \
>                   > kernel.kmidiff.txt
> ```

这个命令需要一点时间，因为它需要解包两个内核包，分析两个 vmlinux 二进制文件以及相关联的二进制模块的 *t* *housands* ，比较接口，并生成一个有意义的报告。根据记录，在我的机器上，大约需要两分钟。您的里程可能会有所不同。

## 分析结果

为了更好地衡量和保持这篇文章足够简洁，您可以通过[点击这里](http://people.redhat.com/~dseketel/kabidiff/kernel-7.0-7.1-2.abidiff.txt)获得比较的文本文件报告。让我们浏览一下那份报告中有趣的片段并加以分析。

首先，这里是报告的摘要:

> ```
> == Kernel ABI changes between packages '7.0/kernel-3.10.0-123.el7.x86_64.rpm' and '7.1/kernel-3.10.0-229.el7.x86_64.rpm' are: ===
> Leaf changes summary: 38 artifacts changed (16 filtered out)
> Added/removed functions summary: 0 Removed, 0 Added functions
> Added/removed variables summary: 0 Removed, 0 Added variable
> ```

所以，38 个工件的改变是改变了的类型。根据定义，这些类型可以从组成 KMI 的函数和全局变量的类型中获得。

工件也可能是函数或变量，但是我们看到没有函数或变量被删除或添加。我们看到 16 个工件变更被过滤掉了，大概是因为工具认为这些变更从 KMI 的角度来看没有意义。工具中有一些选项可以让你看到所有的变化，没有过滤。您可以在在线手册中了解这些选项。

以下是报告的一些类型变化:

> ```
> 'struct netns_nftables at nftables.h:8:1' changed:
> type size hasn't changed
> 1 data member insertion:
>  'unsigned int netns_nftables::base_seq', at offset 608 (in bits) at nftables.h:19:1
> there are data member changes:
>  'unsigned long int netns_nftables::__rht_reserved1' size changed from 64 to 32 (in bits) (by -32 bits)
> 
> ```

您可以看到一个名为“ *base_seq* ”、类型为 *int* (32 位)的数据成员被添加到了*结构 netns_nftables* 中。同时，一个名为 *__rht_reserved1* 的数据成员的大小发生了变化，减少了 32 位。结果，报告说 *struct netns_nftables* 的大小没有改变。因此可以推测，使用类型实例并期望其大小保持不变的代码将继续工作。

现在，让我们看看另一种报告的变化:

> ```
> 'struct swap_info_struct at swap.h:182:1' changed:
>  type size changed from 1152 to 1792 bits
>  2 data member insertions:
>  'plist_node swap_info_struct::list', at offset 1152 (in bits) at swap.h:225:1
>  'plist_node swap_info_struct::avail_list', at offset 1472 (in bits) at swap.h:226:1
> ```

请注意，结构的大小发生了变化。以前是 1152 位，现在是 1792 位。它的增长是因为在其末端添加了两个新的数据成员——从偏移量 1152 开始。

回顾这一变化后，可以公平地认为内核开发人员认为使用这一结构的代码都不希望它的大小保持在 1152。请注意，任何现有数据成员的偏移量都没有改变，这正是因为数据成员是在结构的末尾添加的。

还有另一种有趣的变化:

> ```
> struct cfg80211_connect_params at cfg80211.h:1587:1'
>  changed: type size changed from 1728 to 1856 bits
>  2 data member insertions:
>   'ieee80211_channel* cfg80211_connect_params::channel_hint', at offset 64 (in bits) at cfg80211.h:1775:1
>   'const u8* cfg80211_connect_params::bssid_hint', at offset 192 (in bits) at cfg80211.h:1777:1
>  there are data member changes:
>   'u8* cfg80211_connect_params::bssid' offset changed from 64 to 128 (in bits) (by +64 bits)
>   'u8* cfg80211_connect_params::ssid' offset changed from 128 to 256 (in bits) (by +128 bits)
>   'size_t cfg80211_connect_params::ssid_len' offset changed from 192 to 320 (in bits) (by +128 bits)
>  ...
>  ...
> ```

在这个变更报告中(为了简单起见进行了删减)，这个 *struct cfg80211_connect* 类型的大小发生了变化，因为在它的中间插入了两个数据成员。这导致后续数据成员的偏移量也发生了变化。

很明显，这个结构，即使可以从位于*/lib/modules/kabi-rhel 70/kabi _ whitelist _ x86 _ 64，*的 *kabi whitelis* t 文件中定义的函数和变量到达，也不意味着是 KMI 的一部分。

换句话说，任何使用这种结构的代码都将与文件 *cfg80211.h* 一起被(重新)编译。而且，据推测，这种变化已经被内核开发人员有意识地审查和接受，因为它在实践中不会对二进制 KMI 稳定性产生负面影响。

所以我们在这里看到，这个基于 libabigail 的工具**并没有消除对二进制 KMI 变更**进行同行评审的需要。相反，它为这一审查取得成果提供了一个有希望的有用基础。

它还展示了如何使用 libaigail 库构建一些定制的自动 KMI 稳定性验证工具。

## 可能的改进

这种 Linux 内核支持是一种新的发展。因此，仍有改进的余地。

### 从报告中过滤掉非公共类型的更改

如上所示，有些类型可以从组成二进制 KMI 的函数和全局变量集合中访问，但是它们对于定义它们的模块(或 vmlinux 二进制文件)来说是私有的。

想要移除这些类型的用户可以定义一个[抑制规范文件](https://sourceware.org/libabigail/manual/libabigail-concepts.html#suppression-specifications)，在其中他们基本上定义了一个黑名单，该黑名单中的类型的变化不应该被工具报告。然后可以通过使用 *-抑制*选项将该抑制规范提供给 [abipkgdiff](https://sourceware.org/libabigail/manual/abipkgdiff.html) 。

什么类型被放入黑名单的细节是定义一个特定的审查过程的问题，在这个过程中，事情被彻底地讨论和权衡。

### 过滤掉不影响任何大小或偏移的更改

每当类型变化不影响任何数据成员的偏移量或结构的大小时，默认情况下，我们可以避免显示这种变化。

这个特殊的过滤特性还没有实现，但是我相信它会是一个有趣的特性。

### 支持其他打包格式

目前， *abipkgdiff* 一般支持 rpm 和 DEB 包格式。注意，对 DEB 格式的支持是由 GCC 包的 Debian 维护者添加的。啊，自由和开源软件的美妙之处。好了，我们不要跑题太多。

但是，对于这个特定的 Linux 内核分析特性， *abipkgdiff* 只支持 RPM 格式。即使对于 rpm，它也只支持 RHEL 和 Fedora 内核使用的特定文件布局。我们很乐意接受添加支持其他发行版内核的补丁。

### 表演

今天，libabigail 库的性能是可以接受的，但我们一直在寻找更快(但更健壮)的方法来做事，尤其是现在它必须分析大量的二进制文件，如 Linux 内核及其模块。这也是我们计划改进的地方。

## 结论

libabigail 中最近增加的 Linux 内核二进制支持为它所支持的工具集提供了一组新的分析特性；abipkgdiff 是这些工具中的一个例子，现在可以分析 Linux 内核包。

还有一个新工具(名为 kmidiff)旨在比较来自*源树*的 kmi，而不是 RPM 包。开发人员编译他们自己的源代码树可能会使用 *kmidiff* 而不是 *abipkgdiff* 。

如上所述，这是全新的开发，我们希望在性能、过滤和易用性方面做进一步的改进。

*Last updated: May 26, 2022*