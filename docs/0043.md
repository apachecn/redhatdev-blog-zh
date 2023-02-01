# 你好世界-在 RHEL 7 号上安装 GCC

> 原文：<https://developers.redhat.com/HW/gcc-RHEL-7>

### 简介和先决条件

在本教程中，您将从 Red Hat 开发工具集(DTS)安装 GNU 编译器集合 8.2，并构建一个简单的 C++ Hello World 应用程序。完成本教程应该不到 30 分钟。

在开始之前，您需要一个最新的 Red Hat Enterprise Linux 7 工作站或服务器订阅，以便从 Red Hat 下载软件和获取更新。如果您没有有效的订阅，请从这里注册并获取 RHEL 开发者套件(包括 RHEL 服务器)。

如果您在任一点遇到困难，请参见[故障排除和常见问题解答](https://developers.redhat.com/products/developertoolset/hello-world/#Troubleshooting)。

## [1。启用必要的软件库](https://developers.redhat.com/products/developertoolset/hello-world/#rhel-7_enable-necessary-software-repositories_step_panel_1)

在此步骤中，您将配置您的系统，以便通过使用命令行界面从 Red Hat Software Collection 存储库中获取软件，包括 Red Hat DTS、最新动态语言和开源数据库。

作为根用户，您可以使用`subscription-manager`工具从命令行添加或删除软件仓库。使用`--list`选项查看可用的软件存储库，并验证您是否有权访问 RHSCL，其中包括 DTS:

```
$ su -
# subscription-manager repos --list | egrep devtools
```

如果您在列表中没有看到任何 RHSCL 存储库，则您的订阅可能不包括它。更多信息参见[故障排除和常见问题解答](https://developers.redhat.com/products/developertoolset/hello-world/#Troubleshooting)。

如果您使用的是桌面版的 Red Hat Enterprise Linux，请在以下命令中将`-server-`更改为`-desktop-`:

```
# subscription-manager repos --enable rhel-server-devtools-7-rpms
# subscription-manager repos --enable rhel-7-server-optional-rpms
```

## [2。设置您的开发环境](https://developers.redhat.com/products/developertoolset/hello-world/#rhel-7_setup-your-development-environment_step_panel_2)

在下一步中，您将使用一个命令来下载和安装 GCC 8.2，以及 Red Hat Developer Toolset 中的其他开发工具。此步骤需要的时间取决于您的互联网连接速度和您的系统。如果连接速度相当快，这一步应该在 5 分钟内完成。

要安装所有组件，请键入:

```
$ su -
# yum install devtoolset-8

```

如果您想安装组件的子集，请参阅此处的说明。

注意:在所有 scl 命令中，您仍将使用 devtoolset-8 作为软件集合的名称。只有 yum 要安装的元包的名称会发生变化。

## [3。你好世界和你的第一个应用程序](https://developers.redhat.com/products/developertoolset/hello-world/#rhel-7_hello-world-and-your-first-application_step_panel_3)

**从命令行使用 DTS c++**

GNU C++编译器是用 g++命令运行的。您需要将 DTS 添加到您的环境中，并在`Terminal`窗口中启用 scl。更多信息请参见[将 DTS 永久添加到您的开发环境](https://developers.redhat.com/products/developertoolset/hello-world/#permanently-enable)。

`$ scl enable devtoolset-8 bash`

现在使用文本编辑器创建`hello.cpp`，比如`vi`、`nano`或`gedit`，内容如下:

hello.cpp

```
#include <iostream>

using namespace std;

int main(int argc, char *argv[]) {
  cout << "Hello, Red Hat Developer Program World!" << endl;
  return 0;
}
```

现在编译并运行程序:

```
$ g++ -o hello hello.cpp
$ ./hello
Hello, Red Hat Developer Program World!
```

有关更多信息，请参见 Red Hat Developer Toolset 8 用户指南的 GNU C++编译器部分。

**Docker 格式的容器图像**

`Docker-formatted container`映像可用于在虚拟软件容器中运行 Red Hat Developer Toolset 组件，从而将它们与主机系统隔离开来，并允许它们快速部署。有关 Red Hat 开发工具集 docker 格式的容器映像和 Red Hat 开发工具集`dockerfiles`的详细描述，请参见[使用 Red Hat 软件集合容器映像](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/using_red_hat_software_collections_container_images/)。

### 使用 Red Hat 开发工具集和软件收集包

Red Hat 开发工具集是作为 Red Hat 软件集合中的一组包提供的。RHSCL 中的软件包旨在允许同时安装多个版本的软件。为了实现这一点，需要使用`scl enable`命令将所需的包添加到您的运行时环境中。当`scl enable`运行时，它修改环境变量，然后运行指定的命令。环境变化只影响由`scl`运行的命令和从该命令运行的任何进程。本教程中的步骤运行命令`bash`来启动一个新的交互式 shell，以便在更新的环境中工作。这些变化不是永久性的。输入`exit`会用原来的环境回到原来的外壳。每次登录或开始新的终端会话时，都需要再次运行`scl enable`。

虽然可以更改系统配置文件，使 RHSCL 软件包成为系统全局环境的一部分，但不建议这样做。这样做可能会导致与其他应用程序的冲突和意外问题，因为路径中首先出现的是 RHSCL 版本，从而掩盖了包的系统版本。

## 将 DTS 永久添加到开发环境中

要使 DTS 成为开发环境的永久组成部分，可以将其添加到特定用户 ID 的登录脚本中。这是推荐的开发方法，因为只有在您的用户 ID 下运行的进程才会受到影响。

使用您喜欢的文本编辑器，将下面一行添加到`~/.bashrc`的末尾:

```
source scl_source enable devtoolset-8
```

注销并重新登录后，您可以通过运行`which g++`或`g++ --version`来验证 DTS GCC 是否在您的路径中。

```
$ which g++
/opt/rh/devtoolset-4/root/usr/bin/g++
```

```
$ g++ -v
g++ (GCC) 8.2 20181112 (Red Hat 8.2.1-0)
```

## **使用软件集合包**

这些工具被打包成软件集合，旨在允许同时安装多个版本的软件。为了实现这一点，需要使用`scl enable`命令将所需的包添加到您的运行时环境中。当`scl enable`运行时，它修改环境变量，然后运行指定的命令。环境变化只影响由`scl`运行的命令和从该命令运行的任何进程。本教程中的步骤运行命令`bash`来启动一个新的交互式 shell，以便在更新的环境中工作。这些变化不是永久性的。输入`exit`会用原来的环境回到原来的外壳。每次登录或开始新的终端会话时，都需要再次运行`scl enable`。

虽然可以更改系统配置文件，使 RHSCL 软件包成为系统全局环境的一部分，但不建议这样做。这样做可能会导致与其他应用程序的冲突和意外问题，因为路径中首先出现的是 RHSCL 版本，从而掩盖了包的系统版本。

### **了解更多关于 Red Hat 软件集合的信息**

[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。有关更多信息:

*   [Red Hat Software Collections 打包指南](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/packaging_guide/)—Red Hat Software Collections 打包指南解释了软件集合的概念，记录了 scl 实用程序，并提供了如何创建自定义软件集合或扩展现有软件集合的详细说明。

*   [Red Hat Software Collections 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/3.0_release_notes/)—Red Hat Software Collections 发行说明记录了内容集发布时已知的问题、可能的问题以及其他可用的重要信息。它们还包含关于安装、重建和迁移的有用信息。

*   [如何使用红帽软件集合(RHSCL)、红帽开发者订阅，或者 Clang/LLVM、Go、Rust 编译器](https://access.redhat.com/solutions/472793) —这篇文章列出了哪些红帽企业 Linux 订阅包括访问红帽软件集合、开发者工具集(带 GCC)、Clang/LLVM、Go 和 Rust。

您可以通过运行以下命令来查看 RHSCL 中可用的软件包列表:

`$ yum --disablerepo="*" --enablerepo="rhel-server-rhscl-7-rpms" list available`

#### **使用 Red Hat Enterprise Linux 开发**

Red Hat Enterprise Linux 7 开发人员指南——Red Hat Enterprise Linux 7 开发人员指南介绍了应用程序开发工具以及在 Red Hat Enterprise Linux 7 中使用源代码管理工具(如 Git)。

#### 成为红帽开发者，加入红帽开发者计划

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

## 故障排除和常见问题

1.  作为一名开发人员，我如何获得包含 Clang/LLVM、Go 或 Rust 编译器的 Red Hat Enterprise Linux 订阅？

开发者通过【developers.redhat.com】[注册并下载](https://developers.redhat.com/products/rhel/download/)，可以获得一份用于开发目的的免费红帽企业版 Linux 开发者订阅。我们建议您遵循我们的[入门指南](https://developers.redhat.com/products/rhel/get-started/)，该指南涵盖了使用您选择的 VirtualBox、VMware、Microsoft Hyper-V 或 Linux KVM/Libvirt 在物理系统或虚拟机(VM)上下载和安装 Red Hat Enterprise Linux。更多信息请参见[常见问题:免费红帽企业版 Linux 开发者订阅](https://developers.redhat.com/articles/no-cost-rhel-faq/)。

2.  我在系统上找不到 devtools 或 RHSCL 存储库。

一些 Red Hat Enterprise Linux 订阅不包括对软件集合或开发工具的访问。关于包括哪些订阅的列表，请参见[如何使用 Red Hat 软件集合(RHSCL)、Red Hat 开发工具集(DTS)等。)](https://access.redhat.com/solutions/472793)。

存储库的名称取决于您是否安装了 Red Hat Enterprise Linux 的服务器或工作站版本。您可以使用 subscription-manager 查看可用的软件存储库，并验证您是否有权访问 RHSCL 和 devtools:

```
$ su -
# subscription-manager repos --list | egrep rhscl
# subscription-manager repos --list | egrep devtools
```

3.  当我运行 yum install *包* (go-toolset-7，llvm-toolset-7，rust-toolset-7)时，由于缺少依赖项而失败。

一些软件集合需要 devtools RPMs 存储库中的包，默认情况下不启用。请参见上面的步骤 1，了解如何启用适用的 rpm 和存储库。

4.  如何查看 Go 手册页面？

Go 编译器帮助命令提供了有关其用法的信息。要显示帮助索引:

```
$ scl enable go-toolset-7 'go help'
```

5.  **如何才能知道安装了哪些 RHSCL 软件包？**

`scl --list`将显示已安装的 RHSCL 软件包列表，无论它们是否已启用。

```
$ scl --list
```

*Last updated: November 15, 2021*