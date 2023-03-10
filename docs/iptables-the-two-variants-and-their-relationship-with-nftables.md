# iptables:两种变体及其与 nftables 的关系

> 原文：<https://developers.redhat.com/blog/2020/08/18/iptables-the-two-variants-and-their-relationship-with-nftables>

在[红帽企业版 Linux](https://developers.redhat.com/topics/linux) (RHEL) 8 中，用户空间实用程序 [`iptables`](https://en.wikipedia.org/wiki/Iptables) 与其继任者 [`nftables`](https://en.wikipedia.org/wiki/Nftables) 关系密切。这两个实用程序之间的联系很微妙，这导致了 Linux 用户和开发人员的困惑。在这篇文章中，我试图阐明`iptables`的两个变体和它的后继程序`nftables`之间的关系。

## 内核 API

一开始，只有`iptables`。在 Linux 历史上，它度过了美好而漫长的一生，但它也不是没有痛点。后来，`nftables`出现了。这提供了一个从`iptables`的错误中吸取教训并加以改进的机会。

在本文的上下文中，最重要的改进是内核 API。内核 API 是用户空间对内核编程的方式。您可以使用`nft`命令或`iptables`命令的变体来访问内核 API。我们将重点关注`iptables`变体。

## iptables 命令的两种变体

`iptables`命令的两种变体是:

*   `legacy`:通常简称为`iptables-legacy.`
*   `nf_tables`:常被称为`iptables-nft`。

较新的`iptables-nft`命令提供了到`nftables`内核 API 和基础设施的桥梁。你可以通过查找`iptables`版本来找到正在使用的变体。对于`iptables-nft`，变量将显示在版本号后的括号中，表示为`nf_tables`:

```
root@rhel-8 # iptables -V
iptables v1.8.4 (nf_tables)

```

对于`iptables-legacy`，变量要么不存在，要么在括号中显示`legacy`:

```
root@rhel-7 # iptables -V
iptables v1.4.21

```

您还可以通过检查`iptables`二进制文件是否是到`xtables-nft-multi`的符号链接来识别`iptables-nft`:

```
root@rhel-8 # ls -al /usr/sbin/iptables
lrwxrwxrwx. 1 root root 17 Mar 17 10:22 /usr/sbin/iptables -> xtables-nft-multi

```

## 使用 iptables-nft

正如我前面提到的，`nftables`实用程序改进了内核 API。`iptables-nft`命令允许`iptables`用户利用这些改进。`iptables-nft`命令使用较新的`nftables`内核 API，但是重用了`legacy`包匹配代码。因此，在使用熟悉的`iptables`命令时，您将获得以下好处:

*   原子规则更新。
*   每网络命名空间锁定。
*   没有基于文件的锁定(例如:/run/xtables.lock)。
*   增量规则集的快速更新。

这些好处对用户来说基本上是透明的。

**注意**:对于`nftables`的用户空间命令是`nft`。它有自己的句法和语法。

### 数据包匹配是相同的

重要的是要理解，虽然有两种`iptables`的变体，但是数据包匹配使用的是*相同的*代码。不管您使用的是哪种变体，相同的数据包匹配功能都是可用的，并且行为相同。内核中数据包匹配代码的另一个术语是`xtables`。两种变体，`iptables-legacy`和`iptables-nft`，使用相同的`xtables`代码。这个图表提供了直观的帮助。为了完整起见，我加入了`nft`:

```
+--------------+     +--------------+     +--------------+
|   iptables   |     |   iptables   |     |     nft      |   USER
|    legacy    |     |     nft      |     |  (nftables)  |   SPACE
+--------------+     +--------------+     +--------------+
       |                          |         |
====== | ===== KERNEL API ======= | ======= | =====================
       |                          |         |
+--------------+               +--------------+
|   iptables   |               |   nftables   |              KERNEL
|      API     |               |     API      |              SPACE
+--------------+               +--------------+
             |                    |         |
             |                    |         |
          +--------------+        |         |     +--------------+
          |   xtables    |--------+         +-----|   nftables   |
          |    match     |                        |    match     |
          +--------------+                        +--------------+

```

## iptables 规则出现在 nftables 规则列表中

`iptables-nft`使用`nftables`基础设施的一个有趣的结果是`iptables`规则集出现在`nftables`规则列表中。让我们考虑一个基于简单规则的例子:

```
root@rhel-8 # iptables -A INPUT -s 10.10.10.0/24 -j ACCEPT

```

通过`iptables`命令显示这条规则会产生我们可能期望的结果:

```
root@rhel-8 # iptables -nL INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  10.10.10.0/24        0.0.0.0/0

```

但是它也会显示在`nft`规则集中:

```
root@rhel-8 # nft list ruleset
table ip filter {
    chain INPUT {
        type filter hook input priority filter; policy accept;
        ip saddr 10.10.10.0/24 counter packets 0 bytes 0 accept
    }
}

```

请注意`iptables`规则是如何被自动转换成`nft`语法的。研究自动翻译是发现`iptables`规则的`nft`等价物的一种方式。然而，在某些情况下，没有直接的对等词。在这种情况下，`nft`会通过显示这样的评论让你知道:

```
table ip nat {
    chain PREROUTING {
        meta l4proto tcp counter packets 0 bytes 0 # xt_REDIRECT
    }
}

```

## 摘要

总而言之，`iptables-nft`变体利用了更新的`nftables`内核基础设施。这给了变体一些优于`iptables-legacy`的好处，同时允许它仍然是传统命令的 100%兼容的替代物。但是，请注意，`iptables-nft`和`nftables`是*而不是*的等价物。他们只是共享基础设施。

同样重要的是要注意，虽然`iptables-nft`可以取代`iptables-legacy`，但是你不应该同时使用它们。

*Last updated: April 7, 2022*