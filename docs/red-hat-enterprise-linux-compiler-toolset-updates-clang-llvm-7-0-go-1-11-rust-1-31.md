# Red Hat Enterprise Linux 编译器工具集更新:Clang/LLVM 7.0、Go 1.11、Rust 1.31

> 原文：<https://developers.redhat.com/blog/2019/03/29/red-hat-enterprise-linux-compiler-toolset-updates-clang-llvm-7-0-go-1-11-rust-1-31>

我们很高兴地宣布， [Red Hat Enterprise Linux 7:](https://developers.redhat.com/topics/linux/) 的这三个编译器工具集正式上市

*   Clang/LLVM 7.0
*   Go 1.11
*   铁锈 1.31

这些工具集可以从[Red Hat Enterprise Linux 7 dev tools](https://developers.redhat.com/products/rhel/download/)通道安装。请参阅本文的“编译器工具集细节”一节，了解新特性。

从上一版本开始，这些工具集成为官方支持的 Red Hat 产品。

## 关于 Red Hat Enterprise Linux 的编译器工具集

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和 web 工具，为应用程序开发人员提供最新、稳定的版本。这些红帽支持的产品被打包成[红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/)(脚本语言、开源数据库、网络工具等。)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview) 。所有这些产品都是可安装的，并且包含在所有 Red Hat Enterprise Linux 开发人员订阅和大多数 Red Hat Enterprise Linux 订阅中。大多数组件也可以作为 Linux 容器映像用于跨 Red Hat 平台的混合云开发，包括 Red Hat Enterprise Linux、 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview/) 、 [Red Hat OpenStack](https://www.redhat.com/en/technologies/linux-platforms/openstack-platform) 等。

## **编译器工具集详情**

### **Clang/LLVM 7.0**

![](img/6785d601e99177f0996ceddc56cfff00.png)

该版本基于 LLVM 版，并完全受 Red Hat 支持。

LLVM 项目是模块化和可重用的编译器和工具链技术的集合。LLVM 核心库提供了一个现代的独立于源和目标的优化器，以及对 Red Hat Enterprise Linux CPU 架构的代码生成支持。

Clang 是一个“LLVM 原生”C/C++/Objective-C 编译器，旨在提供惊人的快速编译和极其有用的错误和警告消息，并为构建优秀的源代码级工具提供一个平台。Clang Static Analyzer 是一个工具，它可以自动发现代码中的错误，并且是一个很好的例子，说明可以使用 Clang 前端作为库来构建这种工具来解析 C/C++代码。

此版本包括以下组件:

*   llvm-toolset-7.0(自动安装 Clang)

Clang/LLVM 7.0 在 Red Hat Enterprise Linux 7 (x86_64、Power LE、aarch64、S390x)上运行

**注意:**自 2018 年 11 月发布以来，红帽的 Clang/LLVM 包命名约定发生了变化，因此红帽版本号现在反映了上游版本。`llvm-toolset-7`包(没有点零)基于 Clang/LLVM 5.0。Clang/LLVM 7.0 的新包名是`llvm-toolset-7.0`(七点零)。虽然这现在看起来很混乱，但希望它能让每个人都更容易前进。

以下是版本号和相应软件包名称的列表:

*   LLVM 版本 7.x，包名 llvm-toolset-7.0

*   LLVM 版本 6.x，包名 llvm-toolset-6.0
*   LLVM 版本 5.x，包名 llvm-toolset-7

For more information, see the [LLVM 7.0.0 Release Notes](https://releases.llvm.org/7.0.0/docs/ReleaseNotes.html) and [Clang 7.0.0 Release Notes](http://releases.llvm.org/7.0.0/tools/clang/docs/ReleaseNotes.html).

### **Golang 1.11**

![](img/2a7342cb8a00180060cce5412dcd5291.png)

这个 Go 工具集是基于 Golang 1.11 的，完全受 Red Hat 支持。

Go 富于表现力，简洁，干净，高效。它的并发机制使得编写充分利用多核和联网机器的程序变得容易，而它的新颖类型系统支持灵活和模块化的程序构造。Go 可以快速编译成机器码，同时具有垃圾收集的便利和运行时反射的能力。它是一种快速的静态类型的编译语言，感觉像是一种动态类型的解释语言。

此版本包括以下组件:

*   go-toolset-1.11
*   go-toolset-1.11-golang-1.11

Golang 1.11 运行在 Red Hat Enterprise Linux 7 (x86_64、Power LE、aarch64、S390x)上

有关更多详细信息，请参见 [Go 1.11 发行说明](https://golang.org/doc/go1.11)。

### **铁锈 1.31.1**

![](img/96806c4d1ab6434bcbbebe8cfae00c22.png)

这个 Rust 工具集基于 Rust 1.31，并且完全受 Red Hat 支持。

Rust 是一种开源系统编程语言，由 Mozilla 和一个志愿者社区创建。它旨在帮助开发人员创建快速、安全的应用，充分利用现代多核处理器的强大功能。它防止分段错误并保证线程安全，所有这些都是通过简单易学的语法实现的。

此外，Rust 提供了零成本的抽象、移动语义、有保证的内存安全、无数据竞争的线程、基于特征的泛型、模式匹配、类型推理和高效的 C 绑定，以及最小的运行时大小。

Cargo 是 Rust 的包管理器和构建工具。它允许 Rust 项目声明与特定版本需求的依赖关系。Cargo 将解析完整的依赖图，根据需要下载包，并构建和测试整个项目。

此版本包括以下组件:

*   rust-toolset-1.31(自动安装货物)
*   防锈工具套件-1.31-货物

Rust 1.31 运行在 Red Hat Enterprise Linux 7 (x86_64、Power LE、aarch64、S390x)上

*   定义程序宏的新功能
    *   属性宏允许您定义自定义`#[name]`注释。
    *   功能宏的工作方式类似于那些由`macro_rules!`定义的宏，但是在 Rust 中实现起来更加灵活。
    *   现在可以在 use 语句中导入宏，不再需要`#[macro_use]` crate 属性。
    *   为了帮助编写这些新的宏，这个箱子现在已经稳定了。
*   模块改进
    *   外部板条箱现在在前奏中，它允许一个板条箱名称作为来自任何地方的路径的根。
    *   crate 关键字现在在 paths 和 use 语句中充当您自己的 crate 的根。
    *   2018 版
        *   新的 2018 版标志着 Rust 过去三年发展的一个集体里程碑，同时也做出了一些突破性的改变。现有代码将默认为 2015 版，没有重大变化，不同版本的板条箱完全可以互操作。`cargo new`将在`Cargo.toml`中为新项目指定 edition = "2018 "。
        *   async、await、try 是 2018 年的保留关键字，dyn 现在是严格关键字。
        *   非词法生存期是对以前基于块的生存期系统的改进，允许在许多情况下更快地释放借用的值，以便在其他地方重用。这最初是 2018 年版独有的，但也计划在 2015 年推出。
        *   模块变化:2018 年大多数情况下不需要显式`extern crate`声明。使用路径现在可以相对于当前范围，而不是像 2015 年那样总是从根范围开始。
    *   现在，在更多情况下，尤其是使用新的“_ 占位符”时，可以将生存期保持为隐式。
    *   const fn —函数可以声明为常量，这允许它们在受限的上下文中使用，如常量或静态值的初始化。
    *   稳定的工具:clippy、rls 和 rustfmt。我们已经发布了这些工具的预览版，但是现在它们得到了官方支持。
        *   clippy 增加了额外的代码/样式问题。
        *   rls 为 IDE 集成实现语言服务器协议。
        *   rustfmt 格式化您的代码，并且还与 cargo fmt 子命令集成在一起。
    *   工具链接允许您为自定义链接添加警告注释，尤其是那些由 clippy 添加的注释。例如，`#[allow(clippy::bool_comparison)]`将使您认为可以接受的项目的警告静音。
*   货物的显著变化包括:
    *   货物现在显示一个进度条，因为它建立你的板条箱和依赖。
    *   Cargo 现在允许重命名`Cargo.toml`中的依赖项，这会影响它们在源代码中的引用方式。以前，你只能在源代码中重命名，比如`extern crate published_name as new_name;`。

**注意:**Rust 编译器的命名约定已经改变，因此版本号反映了社区版本。之前的版本是`Rust-toolset-7`，基于 Rust 1.29。

### 欲了解更多信息

*   [开始使用这些编译器](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview)
*   红帽开发者工具[文档](https://access.redhat.com/documentation/en-us/red_hat_developer_tools/2019.1/)

*Last updated: April 1, 2019*