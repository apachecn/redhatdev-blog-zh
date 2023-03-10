# 如何使用 Subscription Manager 或 Yum-Utils 启用/禁用资料档案库

> 原文：<https://developers.redhat.com/blog/2017/09/29/enabledisable-repository-using-subscription-manager-yum-utils>

本博客旨在解决以下问题/回答以下问题。

1.  如何使用 Red Hat Subscription Manager/yum 启用存储库？
2.  需要使用 Red Hat Subscription Manager/yum 访问存储库？
3.  如何使用 rhn classing the Red Hat Subscription Manager/yum 禁用存储库 usisystem 注册？
4.  如何使用红帽订阅管理器/yum 订阅子频道？

要使用 Subscription-Manger 或 Yum-Utils 启用/禁用存储库，您需要:

1.  红帽企业版 Linux 6 或更高版本。
2.  红帽订阅管理(RHSM)。
3.  红帽订阅经理。
4.  向 RHN 经典注册的系统(参见向 rhn 经典注册的系统)。

如果你使用的是最新版本的 Red Hat Enterprise Linux，那么它会非常有用(我建议定期更新你的系统)。

### 解决办法

在此之前，我们需要知道什么是存储库。

存储库是基于交付网络的产品和内容的内容。系统订阅产品，并在您系统的 rhsm.conf 文件中定义。

要启用存储库，您必须是根用户。

*   [username@localhost ~]$ su -

然后检查存储库列表

*   [root@localhost 用户名]# subscription-manager repos-list

要启用存储库

*   [root@localhost 用户名]# subscription-manager repos-enable = repos name

将 ReposName 更改为您想要的存储库名称。

要禁用存储库

*   [root@localhost 用户名]# subscription-manager repos-disable = repos name

将 ReposName 更改为您想要的存储库名称。

在一些大学/企业中，订阅管理器被阻止，因为他们可以使用 yum 来启用或禁用任何存储库。

要使用 yum 来启用. disable repos，您需要使用 yum-utils 安装 config-manager 属性。

*   [username@localhost ~]$ su -
*   [root@localhost 用户名]# yum 安装 yum-utils

在启用存储库之前，要确保所有存储库都处于稳定状态。

*   [root@localhost 用户名]# yum 全部清除

要检查已启用的存储库

*   [root@localhost 用户名]# yum 报告列表已启用

要启用存储库

*   [root@localhost 用户名]# yum-config-manager-enable repos name

将 ReposName 更改为您的存储库名称。

要禁用存储库

*   [root@localhost 用户名]# yum-config-manager -禁用仓库名

将 ReposName 更改为您的存储库名称。

一些安装包使用- enablerepo 来安装包

*   [root@localhost 用户名]# yum 安装包-名称-enable repo repo 名称

以及禁用订阅管理器

*   [root@localhost 用户名]# subscription-manager config-rhsm . manage _ repos = 0

当使用 subscription manager 注册系统时，会创建一个名为 redhat.repo 的文件，它是一个特殊的 yum 存储库。在某些环境中维护 redhat.repo 可能并不理想。如果回购协议不是用于订阅实际回购协议，它可能会在内容管理操作中创建静态。您可以通过将 rshm.manage repo 的值设置为零(0)来禁用它。

任何疑问/问题请在下面评论。

*Last updated: December 20, 2021*