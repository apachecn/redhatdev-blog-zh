# 在 RHEL 6 或 RHEL 7 上使用原生 Python 构建您的第一个应用

> 原文：<https://developers.redhat.com/articles/using-native-python-rhel-6-or-rhel-7>

在不到 10 分钟的时间内开始在 Red Hat Enterprise Linux 上使用 Python 进行开发。

### 简介和先决条件

在本教程中，您将看到如何通过创建一个简单的 Hello World 应用程序，在 Red Hat Enterprise Linux 上开始 Python 开发。完成本教程需要 5 到 10 分钟。

在 Red Hat Enterprise Linux 上，Python 是默认安装的。你可以直接跳到 [Hello Word 和你的第一个应用](https://developers.redhat.com/products/rhel/get-started-rhel7-python/#Hello%20Word%20and%20your%20first%20application)，或者继续阅读了解更多关于安装和维护软件包的信息。

在您开始之前，您需要一个当前的 Red Hat Enterprise Linux 6 或 7 工作站或服务器订阅，允许您从 Red Hat 下载软件并获取更新。如果您没有有效的订阅，请从[这里](https://developers.redhat.com/downloads/)注册并获得 RHEL 开发者套件(包括 RHEL 服务器)。

Python 的本地版本有:

*   红帽企业版 Linux 6 - 2.6

*   红帽企业版 Linux 7 - 2.7

如果你想要一个新版本的 Python 用于 RHEL 6 或 7，使用通过[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)获得的更新版本。

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

Python 2.7 和许多流行的 Python 模块默认安装在 Red Hat Enterprise Linux 上。要查看已经安装了哪些 Python 包，在使用`su`成为 root 之后运行下面的`yum`命令。如果你还没有打开*终端*窗口，从*应用*菜单启动一个。

```
$ su -
# yum list installed python\*
```

如果 Python 没有安装或者不需要更新，您只需要运行一个`yum`命令。

`# yum install python`

要查看 Red Hat Enterprise Linux 中还包括哪些 Python 模块，请运行以下命令:

`# yum list available python\*`

您现在已经完成了需要 root 权限的部分。键入`exit`返回到您的普通用户 ID。

```
# exit
$
```

如果您需要帮助，请参见[故障排除和常见问题解答](https://developers.redhat.com/products/rhel/get-started-rhel7-python/#troubleshooting)。

## 3.Hello World 和您的第一个应用程序

*2 分钟*

在这一步中，您将首先在交互模式下运行 Python。然后，您将创建一个可以从命令行运行的 Python 应用程序。如果没有打开*终端*窗口，从*应用*菜单启动。你应该在你的普通用户 ID 下运行，如果你仍然作为根用户运行，输入`exit`。

```
$ python
Python 2.7.5 (default, Apr  9 2015, 11:03:32)
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello, Red Hat Developers World")
Hello, Red Hat Developers World
>>> quit()
```

下一步是创建一个可以从命令行运行的 Python 应用程序。使用您喜欢的文本编辑器，创建一个名为`hello.py`的文件:

`$ nano hello.py`

将以下文本添加到文件中:

```
#!/usr/bin/python

import sys

version = "Python %d.%d" % (sys.version_info.major, sys.version_info.minor)
print "Hello, Red Hat Developers World from",version
```

保存它并退出编辑器。然后使脚本可执行并运行它:

```
$ chmod +x hello.py
$ ./hello.py
Hello, Red Hat Developers World from Python
```

### 接下来去哪里？

**Python 教程 Python.org**
[https://docs.python.org/](https://docs.python.org/2.7/tutorial)

**找到额外的 Python 模块**
`$ yum list available python\*`

#### 想知道更多关于 RHEL 的事情吗？

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**关注红帽开发者博客**
[https://developers.redhat.com/blog/](https://developers.redhat.com/blog)

**了解红帽软件系列**

[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

## 故障排除和常见问题

1.  我的系统无法从 Red Hat 下载更新。

    我没有当前的红帽订阅，可以获得评估吗？

    如果没有红帽企业版 Linux 订阅，可以免费试用。请访问[https://www . red hat . com/en/technologies/Linux-platforms/enterprise-Linux/server/trial](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux/server/trial)开始评估。开发人员应该选择 Red Hat Enterprise Linux 开发人员工作站选项，以确保您的评估包括来自 Red Hat 开发人员工具集和 Red Hat 软件集合的其他工具。

2.  我尝试过的一些 Python 代码/例子不能与 Red Hat Enterprise Linux 的 Python 2 一起工作。

    Python 3.x 是 Python 语言的新版本，与之前的 2.x 系列不兼容。`/usr/bin/python`中 Red Hat Enterprise Linux 包含的 Python 版本来自 Python 2.x 系列。有大量为 Python 2.x 编写的代码，如果不进行修改，将无法在 Python 3.x 上运行。同样，为 Python 3 编写的代码与 Python 2 不兼容。

    更多信息请参见[“我的开发活动应该使用 Python 2 还是 Python 3？”](https://wiki.python.org/moin/Python2orPython3)在 Python.org[的](https://python.org/)

3.  如何在红帽企业版 Linux 上获得 Python 3？

    Python 3 可以通过[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)获得，它提供了动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

*Last updated: January 9, 2023*