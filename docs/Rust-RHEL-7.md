# 你好世界-在 RHEL 7 号上安装铁锈

> 原文：<https://developers.redhat.com/HW/Rust-RHEL-7>

## **简介和前提条件**

在本教程中，您将安装 Rust 1.31 工具集并构建一个简单的 Rust Hello World 应用程序。完成本教程应该不到 30 分钟。

在开始之前，您需要一个最新的 Red Hat Enterprise Linux 7 工作站或服务器订阅，以便从 Red Hat 下载软件和获取更新。如果您没有有效的订阅，请从这里注册并获取 Red Hat Developer 订阅(包括 RHEL 服务器)。

注意:Rust 编译器命名约定已经更改，因此版本号反映了社区版本。之前的版本是基于 Rust 1.29 的`Rust-toolset-7`。

如果您在任何时候遇到困难，请参阅疑难解答和常见问题。

## [1。启用必要的软件库](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/#rust_enable-necessary-software-repositories_step_panel_1)

在这一步中，您将配置并“注册”您的系统，以便使用命令行界面从 devtools 存储库中获取软件。

注意:注册订阅(接下来的步骤)不同于注册 Red Hat Developer Program(现在您可能已经完成了)。

您可以 root 用户身份使用 subscription-manager 工具从命令行添加或删除软件存储库。使用- list 选项查看可用的软件存储库，并验证您是否有权访问 devtools 存储库。

```
$ su -
# subscription-manager repos --list | egrep devtools
```

如果您在列表中看不到任何 devtools 存储库，您的订阅可能不包括它。请参阅疑难解答和常见问题了解详情。

如果您使用的是桌面版的 Red Hat Enterprise Linux，请在以下命令中更改-server- to -desktop:

```
# subscription-manager repos --enable rhel-7-server-devtools-rpms
# subscription-manager repos --enable rhel-server-rhscl-7-rpms
```

将 Red Hat 开发人员工具密钥添加到您的系统中:

```
# cd /etc/pki/rpm-gpg
# wget -O RPM-GPG-KEY-redhat-devel https://www.redhat.com/security/data/a5787476.txt
# rpm --import RPM-GPG-KEY-redhat-devel
```

## [2。设置您的开发环境](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/#rust_setup-your-development-environment_step_panel_2)

在下一步中，您将使用一个命令来下载 Rust 编译器。此步骤需要的时间取决于您的互联网连接速度和您的系统。如果连接速度相当快，这一步应该在 5 分钟内完成。

```
$ su -
# yum install rust-toolset-1.31
```

在 Rust Toolset 中， **cargo** 由 rust-toolset-1.31-cargo 包提供，并随 rust-toolset-1.31 包自动安装。

## [3。你好世界和你的第一个 Rust 应用程序](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/#rust_hello-world-and-your-first-rust-application_step_panel_3)

### 创建并运行示例 C++ Hello World 项目

以下步骤将创建并运行 Rust Hello World 项目

从命令行使用 Rust

Rust 编译器(rustc)通常不直接调用，而是通过 cargo 工具调用，比如 cargo build。您需要在终端窗口中使用`scl enable`向您的环境添加 Rust。

$ scl 启用 rust-toolset-1.31 bash

开始一个新项目最简单的方法是使用 cargo new。使用“cargo new - bin hello”创建一个 hello 可执行项目。这将为您的程序创建一个包含 Cargo.toml 项目清单和 src/main.rs 入口点的新目录。

```
hello/Cargo.toml
   [package]
   name = "hello"
   version = "0.1.0"
   authors = ["I.M Coder <imc@redhat.com>"]

   [dependencies]

hello/src/main.rs
   fn main() {
       println!("Hello, world!");
   }

```

现在编译并运行程序:

```
$ cd hello
$ cargo run
Hello, world!
```

让我们自定义该消息。使用文本编辑器(如 vi、nano 或 gedit)编辑 src/main.rs，更改为以下内容:

```
fn main() {
">println!("Hello, Red Hat Developer Program World!");
}
```

在进行修改时，从编译器那里获得代码将被接受的快速反馈通常是有用的。您可以运行 cargo check 进行完整的语法和类型检查，而不会因为代码生成而有更长的延迟。然后当你准备好了，编译并再次运行它。

```
$ cargo run
Hello, Red Hat Developer Program World!
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

*Last updated: January 4, 2022*