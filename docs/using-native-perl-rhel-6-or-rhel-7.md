# 在 RHEL 6 或 RHEL 7 上构建您的第一个应用原生 Perl

> 原文：<https://developers.redhat.com/articles/using-native-perl-rhel-6-or-rhel-7>

在不到 10 分钟的时间内开始在 Red Hat Enterprise Linux 上使用 Perl 5.16 进行开发。

### 简介和先决条件

在本教程中，您将看到如何通过创建一个简单的 Hello World 应用程序开始在 Red Hat Enterprise Linux 上进行 Perl 开发。完成本教程需要五到十分钟。

在 Red Hat Enterprise Linux 上，默认安装 Perl。您可以直接跳到“您的第一个应用程序”步骤，或者继续阅读以了解更多关于安装和维护软件包的信息。

在您开始之前，您需要一个当前的 Red Hat Enterprise Linux 6 或 7 工作站或服务器订阅，允许您从 Red Hat 下载软件并获取更新。如果您没有有效的订阅，请从[这里](https://developers.redhat.com/downloads/)注册并获得 RHEL 开发者套件(包括 RHEL 服务器)。

Perl 的本地版本有:

*   红帽企业版 Linux 6 -
*   红帽企业版 Linux 7 - 5.16

如果你想要一个新版本的 Perl 用于 RHEL 6 或 7，使用通过[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)获得的更新版本。

如果您有任何问题，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 1.准备好你的系统

*2 分钟*

在这一步中，您将从 Red Hat 为您的系统下载并安装最新的更新。在此过程中，您将验证您的系统具有最新的 Red Hat 订阅，并且能够接收更新。

首先，从*应用*菜单启动一个*终端*窗口。然后，在使用`su`更改为根用户 ID 之后，使用`subscription-manager`验证您可以访问 Red Hat 软件库。

```
$ su -
# subscription-manager repos --list-enabled
```

如果您没有看到任何已启用的存储库，您的系统可能没有注册到 Red Hat，或者可能没有有效的订阅。更多信息参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

现在通过运行`yum update`下载并安装任何可用的更新。如果有可用的更新，`yum`将列出它们并询问是否可以继续。

`# yum update`

## 2.设置您的开发环境

*2 分钟*

Perl 和许多流行的 Perl 模块默认安装在 Red Hat Enterprise Linux 上。要查看已经安装了哪些 Perl 包，在使用`su`成为 root 之后运行下面的`yum`命令。如果您还没有打开终端窗口，请从*应用程序*菜单中打开一个。

`$ su`


如果 Perl 没有安装或者需要更新，您只需要运行一个`yum`命令。

`# yum install perl`

要查看 Red Hat Enterprise Linux 还包括哪些 Perl 模块，请运行以下命令:

`# yum list available perl\*`

您现在已经完成了需要 root 权限的部分。键入`exit`返回到您的普通用户 ID。

```
# exit
$
```

如果您需要帮助，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 3.Hello World 和您的第一个应用程序

*2 分钟*

在这一步中，您将创建并运行一个 Perl 应用程序。如果您没有打开终端窗口，请从*应用程序*菜单启动它。你应该在你的普通用户 ID 下运行，如果你仍然使用 su 作为根用户运行，输入`exit`。

使用您喜欢的文本编辑器，创建 hello.pl:

`$ nano hello.pl`

添加以下案文:

```
#!/usr/bin/env perl
#
print "Hello, Red Hat Developers World from Perl $^V\n";
```

保存它并退出编辑器。然后使脚本可执行并运行它:

```
$ chmod +x hello.pl
$ ./hello.pl
Hello, Red Hat Developers World from Perl
```

### 接下来去哪里？

在 learn.perl.org
[http://learn.perl.org/tutorials/](https://learn.perl.org/tutorials/)阅读 Perl 教程

**查找包含在 Red Hat Enterprise Linux**
`$ yum list available perl\*`中的其他 Perl 模块

#### 想知道更多关于 RHEL 的事情吗？

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**关注红帽开发者博客**
[https://developers.redhat.com/blog/](https://developers.redhat.com/blog)

**了解红帽软件系列**

[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

**了解 Red Hat 开发工具集**

Red Hat Developer Toolset 提供了最新、稳定、开源的 C 和 C++编译器以及包括 Eclipse 在内的补充开发工具。DTS 使开发人员能够一次编译应用程序，并跨多个版本的 Red Hat Enterprise Linux 进行部署。

*   [Red Hat 开发者工具集产品页面](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=6)

*   [Red Hat 开发者工具集 3.1 发行说明](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/11)

*   [Red Hat 开发者工具集 3.1 用户指南](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/11)

## 故障排除和常见问题

1.  我的系统无法从 Red Hat 下载更新？

    我没有当前的红帽订阅，可以获得评估吗？

    如果没有红帽企业版 Linux 订阅，可以免费试用。从位于[https://access . red hat . com/products/red-hat-enterprise-Linux/evaluation](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux/server/trial)的评估开始。开发人员应该选择 Red Hat Enterprise Linux 开发人员工作站选项，以确保您的评估包括来自 Red Hat 开发人员工具集和 Red Hat 软件集合的其他工具。

2.  如何在 Red Hat Enterprise Linux 上获得较新版本的 Perl？

    通过[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)可以获得 Perl 的新版本，它提供了动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

*Last updated: January 9, 2023*