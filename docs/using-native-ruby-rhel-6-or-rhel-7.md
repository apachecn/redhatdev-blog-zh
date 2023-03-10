# 在 RHEL 6 或 RHEL 7 上使用原生 Ruby 构建你的第一个应用

> 原文：<https://developers.redhat.com/articles/using-native-ruby-rhel-6-or-rhel-7>

在不到 10 分钟的时间内开始在 Red Hat Enterprise Linux 上使用 Ruby 进行开发。

### 简介和先决条件

在本教程中，您将看到如何通过创建一个简单的 Hello World 应用程序开始在 Red Hat Enterprise Linux 上进行 Ruby 开发。完成本教程需要五到十分钟。

在您开始之前，您需要一个当前的 Red Hat Enterprise Linux 6 或 7 工作站或服务器订阅，允许您从 Red Hat 下载软件并获取更新。如果您没有有效的订阅，请从[这里](https://developers.redhat.com/downloads/)注册并获得 RHEL 开发者套件(包括 RHEL 服务器)。

Ruby 的原生版本有:

*   红帽企业版 Linux 6 - 1.8.7
*   红帽企业版 Linux 7 - 2.0

如果你想在 RHEL 6 或 7 上使用一个新版本的 Ruby，使用通过[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)获得的更新版本。

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

在这一步中，您将使用一个命令下载并安装 Ruby。如果您还没有打开终端窗口，请从*应用程序*菜单中打开一个。您需要使用`su`以 root 权限运行。

```
$ su -
# yum install ruby
```

要查看 Red Hat Enterprise Linux 还包括哪些 Ruby 包，请运行以下命令:

`# yum list available ruby\*`

您现在已经完成了需要 root 权限的部分。键入`exit`返回到您的普通用户 ID。

```
# exit
$
```

如果您需要帮助，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 3.Hello World 和您的第一个应用程序

*2 分钟*

在这一步中，您将首先在交互模式下运行 Ruby。然后，您将创建一个可以从命令行运行的 Ruby 应用程序。如果您没有打开终端窗口，请从*应用程序*菜单启动它。你应该在你的普通用户 ID 下运行，如果你仍然作为根用户运行，输入`exit`。

```
$ irb
irb(main):001:0> puts "Hello, Red Hat Developers World from Ruby " + RUBY_VERSION
Hello, Red Hat Developers World from Ruby
=> nil
irb(main):002:0> quit
```

下一步是创建一个可以从命令行运行的 Ruby 应用程序。使用您喜欢的文本编辑器，创建一个名为`hello.rb`的文件:

`$ nano hello.rb`

将以下文本添加到文件中:

```
#!/usr/bin/env ruby
#
puts "Hello, Red Hat Developers World from Ruby " + RUBY_VERSION
```

保存它并退出编辑器。然后使脚本可执行并运行它:

```
$ chmod +x hello.rb
$ ./hello.rb
Hello, Red Hat Developers World from Ruby 2.0.0
```

### 接下来去哪里？

在 Ruby-lang.org
T3【https://www.ruby-lang.org/en/documentation/quickstart/】20 分钟学会红宝石

**找到额外的红宝石包**
`$ yum list available ruby\*`

#### 想知道更多关于 RHEL 的事情吗？

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**关注红帽开发者博客**
[https://developers.redhat.com/blog/](https://developers.redhat.com/blog)

**了解红帽软件系列**

[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

## 故障排除和常见问题

1.  我的系统无法从 Red Hat 下载更新？

    我没有当前的红帽订阅，可以获得评估吗？

    如果没有红帽企业版 Linux 订阅，可以免费试用。请访问[https://www . red hat . com/en/technologies/Linux-platforms/enterprise-Linux/server/trial](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux/server/trial)开始评估。开发人员应该选择 Red Hat Enterprise Linux 开发人员工作站选项，以确保您的评估包括来自 Red Hat 开发人员工具集和 Red Hat 软件集合的其他工具。

2.  如何在红帽企业版 Linux 上获得较新版本的 Ruby？

    通过[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)可以获得较新版本的 Ruby，它提供了动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

*Last updated: January 9, 2023*