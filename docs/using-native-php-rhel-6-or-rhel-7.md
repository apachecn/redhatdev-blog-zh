# 在 RHEL 6 或 RHEL 7 上使用原生 PHP 构建您的第一个应用程序

> 原文：<https://developers.redhat.com/articles/using-native-php-rhel-6-or-rhel-7>

在不到 10 分钟的时间内开始在 Red Hat Enterprise Linux 上使用原生 PHP 进行开发。

**简介和前提条件**

在本教程中，您将看到如何通过创建一个简单的 Hello World 应用程序开始在 Red Hat Enterprise Linux 上进行 PHP 开发。完成本教程需要 5 到 10 分钟。

在您开始之前，您需要一个当前的 Red Hat Enterprise Linux 6 或 7 工作站或服务器订阅，允许您从 Red Hat 下载软件并获取更新。如果您没有有效的订阅，请从[这里](https://developers.redhat.com/downloads/)注册并获得 RHEL 开发者套件(包括 RHEL 服务器)。

PHP 的本地版本有:

*   红帽企业版 Linux 6 - 5.3
*   红帽企业版 Linux 7 - 5.4

如果你想要一个新版本的 PHP 用于 RHEL 6 或 7，使用通过[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)获得的更新版本。

如果您有任何问题，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## [1。准备好你的系统](http://developers.redhat.com/products/rhel/get-started-rhel7-php/#Step1)

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

在这一步中，您将使用一个命令下载并安装 PHP。如果您还没有打开终端窗口，请从*应用程序*菜单中打开一个。您需要使用`su`以 root 权限运行。

```
$ su -
# yum install php
```

要查看 Red Hat Enterprise Linux 还包括哪些 PHP 模块，请运行以下命令:

`# yum list available php\*`

您现在已经完成了需要 root 权限的部分。键入`exit`返回到您的普通用户 ID。

```
# exit
$
```

如果您需要帮助，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 3.Hello World 和您的第一个应用程序

*2 分钟*

在这一步中，您将首先在交互模式下运行 PHP。然后，您将创建一个可以从命令行运行的 PHP 应用程序。如果您没有打开终端窗口，请从*应用程序*菜单启动它。你应该在你的普通用户 ID 下运行，如果你仍然作为根用户运行，输入`exit`。

```
$ php -a
Interactive shell

php > printf("Hello, Red Hat Developer's World from PHP %s\n",PHP_VERSION);
Hello, Red Hat Developer's World from PHP
php > quit
```

下一步是创建一个可以从命令行运行的 PHP 应用程序。使用您喜欢的文本编辑器，创建一个名为`hello.php`的文件:

`$ nano hello.php`

将以下文本添加到文件中:

```
#!/usr/bin/env php
<?php
print "Hello, Red Hat Developers World from PHP " . PHP_VERSION . "\n";
?>
```

保存它并退出编辑器。然后使脚本可执行并运行它:

```
$ chmod +x hello.php
$ ./hello.php
Hello, Red Hat Developers World from PHP 5.4.16
```

### 接下来去哪里？

**php.net PHP 教程**
[http://php.net/manual/en/tutorial.php](http://php.net/manual/en/tutorial.php)

**找到额外的 PHP 模块**
`$ yum list available php\*`

#### 想知道更多关于 RHEL 的事情吗？

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](http://developers.redhat.com/)免费注册。

**关注红帽开发者博客**
[http://developerblog.redhat.com/](http://developerblog.redhat.com/)

**了解红帽软件系列**

[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

## 故障排除和常见问题

1.  我的系统无法从 Red Hat 下载更新？

    我没有当前的红帽订阅，可以获得评估吗？

    如果没有红帽企业版 Linux 订阅，可以免费试用。从位于[https://access . red hat . com/products/red-hat-enterprise-Linux/evaluation](https://access.redhat.com/products/red-hat-enterprise-linux/evaluation)的评估开始。开发人员应该选择 Red Hat Enterprise Linux 开发人员工作站选项，以确保您的评估包括来自 Red Hat 开发人员工具集和 Red Hat 软件集合的其他工具。

2.  如何在 Red Hat Enterprise Linux 上获得较新版本的 PHP？

    通过[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)可以获得 PHP 的新版本，它提供了动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

*Last updated: November 19, 2020*