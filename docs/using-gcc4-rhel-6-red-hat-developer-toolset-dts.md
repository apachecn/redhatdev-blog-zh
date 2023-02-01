# 在 RHEL 6 上使用 GCC4 和 Red Hat 开发工具集(DTS)构建您的第一个应用

> 原文：<https://developers.redhat.com/articles/using-gcc4-rhel-6-red-hat-developer-toolset-dts>

在不到 10 分钟的时间内开始在 Red Hat Enterprise Linux 上开发 C++的本地版本。

### 简介和先决条件

在本教程中，您将安装 GNU 编译器集合(GCC)并构建一个简单的 C++ Hello World 应用程序。完成本教程不到 15 分钟。

在您开始之前，您需要一个当前的 Red Hat Enterprise Linux 6 工作站或服务器订阅，允许您从 Red Hat 下载软件并获取更新。如果您没有有效的订阅，请从[这里](https://developers.redhat.com/downloads/)注册并获得 RHEL 开发者套件(包括 RHEL 服务器)。

如果您想要在 Red Hat Enterprise Linux 上安装 GCC 的新版本，请安装并使用属于 [Red Hat Developer 工具集](https://developers.redhat.com/products/developertoolset/overview/)的版本。

如果您在任一点遇到困难，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

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

*5 分钟*

在下一步中，您将使用一个命令来下载和安装 GCC C 和 C++编译器以及其他开发工具。您应该仍然打开先前的*终端*窗口，并且仍然在 su 下运行。

`# yum install @development`

如果您在安装过程中选择了*开发工具*包集合，您的系统可能已经安装了这些工具。在这种情况下，`yum`将返回*无所事事的*。

需要以 root 身份运行的步骤已经完成，所以键入`exit`返回到您的普通用户 ID。

```
# exit
$
```

## 3.Hello World 和您的第一个应用程序

*5 分钟*

在这一步中，您将创建并运行 C++ Hello World 应用程序。您应该仍然有一个*终端*窗口在您的常规用户 ID 下打开运行。

现在使用您喜欢的文本编辑器或简单地使用`cat`来创建`hello.cpp`:

```
$ cat > hello.cpp
#include <iostream>

using namespace std;

int main(int argc, char *argv[]) {
  cout << "Hello, Red Hat Developers World!" << endl;
  return 0;
}
```

键入`control-d`退出`cat`，或者如果您使用了编辑器，保存文件并退出。

现在编译并运行程序:

```
$ pass:[g++] -o hello hello.cpp
$ ./hello
Hello, Red Hat Developers World!
```

### 接下来去哪里？

**用红帽企业版 Linux 开发**
[红帽企业版 Linux 开发者指南](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Developer_Guide/index.html) —《红帽企业版 Linux 6 开发者指南》提供了应用程序开发工具以及在红帽企业版 Linux 6 中使用 Git 等源代码管理工具的介绍。
您可能还想查看 [Red Hat Enterprise Linux 7 开发人员指南](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Developer_Guide/)了解最新信息。红帽企业版 Linux 7 于 2014 年发布。

#### 想知道更多关于 RHEL 的事情吗？

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**了解红帽软件系列**

[Red Hat Software Collections](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=5)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

**了解 Red Hat 开发工具集**

Red Hat Developer Toolset 提供了最新、稳定、开源的 C 和 C++编译器以及包括 Eclipse 在内的补充开发工具。DTS 使开发人员能够一次编译应用程序，并跨多个版本的 Red Hat Enterprise Linux 进行部署。

*   [Red Hat 开发者工具集产品页面](https://access.redhat.com/products/Red_Hat_Enterprise_Linux/Developer/#dev-page=6)

*   [Red Hat 开发者工具集 3.1 发行说明](https://access.redhat.com/documentation/en-US/Red_Hat_Developer_Toolset/3/html-single/3.1_Release_Notes/index.html)

*   [Red Hat 开发者工具集 3.1 用户指南](https://access.redhat.com/documentation/en-US/Red_Hat_Developer_Toolset/3/html/User_Guide/)

## 故障排除和常见问题

1.  我的系统无法从 Red Hat 下载更新。

    我没有当前的红帽订阅，可以获得评估吗？

    如果没有红帽企业版 Linux 订阅，可以免费试用。从位于[https://access . red hat . com/products/red-hat-enterprise-Linux/evaluation](https://access.redhat.com/products/red-hat-enterprise-linux/evaluation)的评估开始。开发人员应该选择 Red Hat Enterprise Linux 开发人员工作站选项，以确保您的评估包括来自 Red Hat 开发人员工具集和 Red Hat 软件集合的其他工具。

2.  我使用的是哪个版本的 GCC？

    Red Hat Enterprise Linux 包括 GNU 编译器集合的一个版本，该版本在与 Red Hat Enterprise Linux 发布相同的生命周期内受支持。Red Hat Enterprise Linux 的主要版本支持长达 10 年。

    使用`g++ -v`查看您安装的版本。

    ```
    $ pass:[g++] -v
    gcc version 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC)
    ```

3.  如何获得 GCC/G++的新版本？

    Red Hat Developer Toolset 提供了最新、稳定、开源的 C 和 C++编译器以及包括 Eclipse 在内的补充开发工具。DTS 使开发人员能够一次编译应用程序，并跨多个版本的 Red Hat Enterprise Linux 进行部署。Red Hat Developer 工具集使用软件集合在`/opt/rh`中安装一组并行的包，它们不会覆盖 Red Hat Enterprise Linux 附带的系统包。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

*Last updated: November 19, 2020*