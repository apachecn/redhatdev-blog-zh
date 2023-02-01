# 防火墙:未来是不可预测的

> 原文：<https://developers.redhat.com/blog/2018/08/10/firewalld-the-future-is-nftables>

Firewalld 是 Red Hat Enterprise Linux 和 Fedora 中的默认防火墙管理工具，它获得了对 nftables 的长期支持。这在 firewalld 的项目[博客](https://firewalld.org/2018/07/nftables-backend)上详细公布了。该特性在 firewalld 0.6.0 版本中作为新的默认防火墙后端发布。

Red Hat 开发人员的博客中概述了 nftables 的好处:

*   iptables 之后是什么？当然，它的继任者是:nftables
*   [基准测试 nftables](https://developers.redhat.com/blog/2017/04/11/benchmarking-nftables/)
*   [将我的 iptables 设置迁移到 nftables](https://developers.redhat.com/blog/2017/01/10/migrating-my-iptables-setup-to-nftables/)

firewalld 有许多长期存在的问题，我们可以用 nftables 来解决，这在旧的 iptables 后端是不可能的。nftables 后端允许以下改进:

*   使用一个底层工具 nft 即可查看所有防火墙信息
*   适用于 IPv4 和 IPv6 的单一规则，而不是重复的规则
*   不承担防火墙后端的完全控制
*   不会删除由其他工具或用户安装的防火墙规则
*   规则优化(在同一规则中记录和拒绝)

最重要的是，新的后端几乎 100%兼容先前存在的配置。大多数用户甚至不会注意到一些变化。这意味着即使速度较慢的发行版也应该能够获得新版本。

您可以立即开始使用 firewalld 和 nftables！firewalld 0.6.0 已经可以在 Fedora rawhide 中使用，并将在即将发布的 Fedora 29 版本中使用。现有的 Fedora 安装将在升级到 Fedora 29 时自动升级到 nftables 后端。

不幸的是，firewalld 的 nftables 后端不太可能找到通往 Red Hat Enterprise Linux 7 的道路。好消息是，由于 Fedora 是 RHEL 的上游，nftables 后端很可能最终会在未来的 RHEL 版本中发布。

欲知详情，请参考上游[在 firewalld.org](https://firewalld.org/2018/07/nftables-backend)的[上的博文](https://firewalld.org/)。防火墙快乐！

*Last updated: August 9, 2018*