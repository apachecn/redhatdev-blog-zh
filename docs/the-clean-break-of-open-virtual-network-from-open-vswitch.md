# 开放式虚拟网络与开放式虚拟交换机的彻底决裂

> 原文：<https://developers.redhat.com/blog/2019/08/30/the-clean-break-of-open-virtual-network-from-open-vswitch>

经过大量的电子邮件和 IRC 讨论，[开放虚拟网络(OVN)源代码](https://github.com/ovn-org/ovn)已经从[开放虚拟交换机(OVS)源代码](https://github.com/openvswitch/ovs)中分离出来，两个项目现在独立运作。在这篇文章中，我们将解释把 OVN 从 OVS 分离出来的原因，分离的技术方面，以及 OVN 项目即将面临的挑战。

## 背景

OVN 最初是在 2015 年 1 月公布在 OVS 开发者邮件列表上的。从最初的[电子邮件公告](https://mail.openvswitch.org/pipermail/ovs-dev/2015-January/293922.html):

> “OVN 不需要对 ovs-vswitchd 或 ovsdb-server 进行 OVS 或 OVN 特定的特殊构建。OVN 组件将成为开放式 vSwitch 源代码和二进制版本的一部分。”

将 OVN 纳入 OVS 同一来源分布的决定主要是出于方便。通过将它包含在同一个源分布中，OVN 可以毫无问题地利用 OVS 的前沿特性。OVS 不需要修改就可以导出库供外部程序使用。所需要做的就是在 OVS 项目中创建一个 OVN 子目录，并在其中放入一些代码。也就是说，从一开始就有关于 OVN 存在于一个单独的回购协议中的讨论，而且人们预计最终 OVN 会分拆成自己的回购协议。因此，问题变成了:什么时候有必要这样做，谁愿意投入这项工作？

## 现状

自 2015 年最初创建 OVN 以来，该项目已经成熟，云管理服务(CMSes)已经开始采用它。在 Red Hat， [OpenStack](https://www.openstack.org/) 和 [Openshift](https://developers.redhat.com/openshift/) 都利用 OVN 来定义他们的虚拟网络。从他们的角度来看，他们直接与 OVN 互动，而 OVS 进行更多的“幕后”工作。OVN 不断获得重要的新功能，而 OVS 则获得了他们几乎不太关心的变化。此外，从 CMS 的角度来看，OVS 被视为“稳定”没有太多的动力去升级他们运行的 OVS 版本。OVS 以六个月为一个发布周期，但 CMSes 对更快地获得 OVN 的新功能感兴趣。CMSes 对 OVS 目前的功能很满意，如果没有必要，他们宁愿不更新 OVS。相反，他们不得不等待六个月的 OVN 功能可用，然后他们被迫更新 OVS 超出他们想要使用的。

## 分离的三个阶段

基于这种情况，关于将 OVN 从 OVS 分离出去的讨论已经持续了很长时间。2019 年早些时候，红帽公司决定进行跑腿工作，将 OVN 与 OVS 分开。第一步是解决拆分的技术问题。这里有一个我创建的[邮件列表帖子](https://mail.openvswitch.org/pipermail/ovs-dev/2018-December/354513.html)，概述了执行分离的潜在策略。最后，我们提出了一个分三个阶段将 OVN 从 OVS 分离出去的计划。

*   第一阶段:将 OVN 和 OVS 的包装分开。
*   阶段 2:创建一个单独的 OVN 源 repo，包括 OVS 作为一个 Git 子树。
*   第 3 阶段:消除 OVS 子树，允许使用远程安装的 OVS 编译 OVN。

我们在一月份完成了第一阶段。一个新的规范文件被添加到 OVS 发行版中，OVN 的 rpm 被移动到这个规范文件中。此时，包名也发生了变化。当 OVS 2.11 发布时，我们现在创建了 *ovn-central* 和 *ovn-host* 包，而不是 *openvswitch-ovn-central* 和*openv switch-ovn-host*。除了包名的变化，这一步对用户来说可能是不明显的。新软件包的设计使得升级可以透明地安装新软件包。

对于第二阶段，我们在今年早些时候进行了几次[试验](https://mail.openvswitch.org/pipermail/ovs-dev/2019-February/356365.html) [运行](https://mail.openvswitch.org/pipermail/ovs-dev/2019-March/357586.html)将 OVN 和 OVS 分开。然而，我们选择在 OVS 2.12 分支创建之后才进行这一更改。这样，我们就有了一个明确的分离点，让 OVN 发展以自己的回购方式发生。OVS 2.12 分支于 7 月 22 日创建，我们在接下来的一周进行了拆分。自分拆以来，所有 OVN 开发商都将其新功能瞄准了新的 OVN 回购协议。当 OVS 2.12.0 发布时，它将是 OVS 的最终版本，还包括一个配套的 OVN 版本。随着第二阶段的完成，OVN 可以自由地改变它的发布节奏；但是，它也有责任保持与多个版本的 OVS 的兼容性。

第三阶段目前正在审核中，可能很快就会被合并。为了方便起见，将 OVS 作为 Git 子树是一个很好的权宜之计，但是这使得构建和测试 OVN 有点奇怪。具体来说，保持 Git 子树的更新并不像保持系统中的一个单独的 repo 更新那样简单。当运行单元测试时，由于 GNU autotest 的工作方式，这些测试将从 OVS 子树和 OVN 的 tests/目录中运行。当试图运行特定的 OVN 测试时，这可能会导致麻烦。随着 OVS 的分离，这不再是一个问题，这使得对 OVN 的测试更快。

一个未说明的(也是显而易见的)第四步是从 OVS 回购协议中删除 OVN 代码。我们的邮件列表上有一个[补丁系列](https://mail.openvswitch.org/pipermail/ovs-dev/2019-August/361704.html)可以做到这一点，但它仍在等待批准。

## 今后

将 OVN 从 OVS 分离的“物理”方面已经完成。然而，我们在前进的道路上仍然面临许多挑战。最大的问题是提出保持 OVN 和 OVS 之间兼容性的政策。我已经开始了一个邮件列表讨论，其中有一个文档列出了一个潜在的解决方案。

从现在开始，我们如何计划 OVN 的版本也是一个问题。鉴于 CMSes 希望能够更快地使用新的 OVN 功能，比每六个月更快地发布新版本的 OVN 可能是有意义的。以前，OVN 是每个 OVS 版本的一部分。因此，它需要有 OVN 和 OVS 的匹配版本。然而，如果我们比 OVS 更频繁地发布 OVN，版本号将会扭曲，潜在地导致混乱。因此，我们可能会完全改变 OVN 的版本方案。我已经开始了一个[邮件列表讨论](https://mail.openvswitch.org/pipermail/ovs-dev/2019-August/361439.html),其中有一个文档提出了一个更短的发布周期和一个新的版本编号方案。

还需要完成许多较小的清理任务。例如，文档文件夹仍然包含许多对“打开虚拟交换机”的引用，而不是“OVN”为了方便起见，OVS 准备了一些补充材料。例如，它包含了建立快速流浪环境的文件。OVN 可以从类似的便利设施中受益。

其他行政工作正在进行中。例如，正在为 OVN 创建单独的邮件列表，这样众多的 OVN 讨论就不会“污染”OVS 列表。还努力创建一个独立于 openvswitch.org 网站的 ovn.org 网站。

OVN 继续成长，成为创建虚拟网络的领先解决方案。OVN 与 OVS 的分离标志着一个酝酿了近五年的里程碑。随着 OVN 与 OVS 的分离，这是一个伟大的新功能的起点，如果你有兴趣，也是一个加入项目开发的绝佳时机。

*Last updated: August 29, 2019*