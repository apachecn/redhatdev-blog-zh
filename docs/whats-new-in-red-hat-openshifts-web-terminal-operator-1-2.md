# 红帽 OpenShift 的 Web 终端操作器 1.2 有什么新功能

> 原文：<https://developers.redhat.com/blog/2021/03/08/whats-new-in-red-hat-openshifts-web-terminal-operator-1-2>

[Red Hat OpenShift](/products/openshift/overview) 的 web 终端操作器是一种用户访问预装通用集群工具的 Web 终端的方式。这为您提供了直接通过 OpenShift web 控制台处理产品的能力和灵活性，消除了在本地安装所有工具的需要。

本文概述了 Web Terminal Operator 1.2 中引入的新功能。这些改进包括允许集群管理员安全地访问终端，当终端由于不活动而关闭时为用户提供更多信息，以及与 OpenShift 4.7 保持一致的工具更新。

## 更轻松地访问 OpenShift web 控制台

在 web 终端操作员 1.2 中，集群管理员可以直接访问 Web 终端，如图 1 所示。

[![Opening Web Terminal as cluster admin](img/a323012355dd7228f7e9639e0b9ba3d3.png "Figure 1")](/sites/default/files/blog/2021/02/Figure-1.gif)

Figure 1: Opening the web terminal as a cluster administrator.

现在，集群中的每个人都可以访问他们自己的 web 终端，无论他们的权限级别如何。注意，出于安全原因，集群管理员将自动绕过名称空间选择器，在`openshift-terminal`名称空间中创建他们的 web 终端。除此之外，其功能与没有集群管理员角色的用户相同。

## 了解终端停止的原因

为了节省资源，web 终端会在 15 分钟不活动后自动关闭。在 Web Terminal Operator 1.2 中，我们添加了更多信息来帮助用户理解终端停止的原因(参见图 2)。

[![OpenShift Console showing why the web terminal stopped](img/f2cb056674e9fe370703923726c976ed.png "Figure 2")](/sites/default/files/blog/2021/02/Figure-2.png)

Figure 2: The terminal window indicating why a terminal has shut down.

Web Terminal Operator 1.2 还提供了一种更简单的方式，只需点击一个按钮就可以重启终端，如图 3 所示。

[![OpenShift Console showing how restart terminal button](img/a341ecd19b55040374cbcb0263abc84c.png "Figure 3")](/sites/default/files/blog/2021/02/Figure-3.gif)

## 更新的工具

我们已经更新了 Web Terminal Operator 1.2 中的默认二进制文件，以包含内置命令行工具的最新版本，如表 1 所示。

**Table 1: Command-line tools in Web Terminal Operator 1.2**

| 二进制的 | 旧版本 | 新版本 |
| `oc` | 4.6.1 | 4.7.0 |
| `kubectl` | 1.19.0 | 1.20.1 |
| `odo` | 2.0.0 | 2.0.4 |
| `helm` | 3.3.4 | 3.5.0 |
| `kn</code` | 0.16.1 | 0.19.1 |
| `tkn` | 0.11.0 | 0.15.0 |

## 额外资源

要想一窥网络终端操作员是如何工作的，请看安吉尔·米塞夫斯基的《网络终端操作员 》一文。你也可以看看 Joshua Wood 最初发布的文章: [*用 Red Hat OpenShift 的新 web 终端*](/blog/2020/10/01/command-line-cluster-management-with-red-hat-openshifts-new-web-terminal-tech-preview/) 进行命令行集群管理。

*Last updated: March 5, 2021*