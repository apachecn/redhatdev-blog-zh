# Clang/LLVM 6.0、Go 1.10 和 Rust 1.29 现已进入 Red Hat Enterprise Linux 测试版

> 原文：<https://developers.redhat.com/blog/2018/10/24/clang-llvm-6-0-go-1-10-rust-1-29-beta-rhel>

我们很高兴地宣布，这三个编译器工具集现已在 Red Hat Enterprise Linux 7 的测试版中推出。正式发布后，这些版本将成为官方支持的 Red Hat 产品:

*   Clang/LLVM 6.0
*   Go 1.10
*   铁锈 1.29

这些工具集可以从 Red Hat Enterprise Linux 7 Devtools 频道安装。请参阅下面的“新编译器详细信息”,了解新特性。

## **关于 Red Hat Enterprise Linux 的 Red Hat 编译器工具集**

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具，为应用程序开发人员提供最新、稳定的版本。这些红帽支持的产品被打包成 [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/) (脚本语言、开源数据库、web 工具等。)、 [红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是可安装的，并且包含在所有 Red Hat Enterprise Linux 开发人员订阅和大多数 Red Hat Enterprise Linux 订阅中。大多数组件也可以作为 Linux 容器映像用于跨 Red Hat 平台的混合云开发，包括:Red Hat Enterprise Linux、Red Hat OpenShift、Red Hat OpenStack 等。 **编译器工具集详情**

## **新的编译器细节**

### **铁锈 1.29**

Rust 是一种开源系统编程语言，由 Mozilla 和一个志愿者社区创建，旨在帮助开发人员创建快速、安全的应用程序，充分利用现代多核处理器的强大功能。它防止分段错误并保证线程安全，所有这些都是通过简单易学的语法实现的。

此外，Rust 提供了零成本的抽象、移动语义、有保证的内存安全、无数据竞争的线程、基于特征的泛型、模式匹配、类型推理和高效的 C 绑定，以及最小的运行时大小。

Cargo 是 Rust 的包管理器和构建工具。它允许 Rust 项目声明与特定版本需求的依赖关系。Cargo 将解析完整的依赖图，根据需要下载包，并构建和测试整个项目。

此版本包含以下组件:

*   防锈工具组-1.29
*   防锈工具组-1.29-货物 1.28
*   容器图像:dev tools-beta/rust-toolset-1.29-rhel 7

Rust 1.29 运行于 RHEL 7 (x86_64，Power LE，aarch64，S390x)

### **Clang/LLVM 6.0**

这个版本是基于 LLVM 版本 6 的，它将成为第一个全面支持的版本。

LLVM 项目是模块化的、可重用的编译器和工具链技术的集合。LLVM 核心库提供了一个现代的独立于源和目标的优化器，以及对 RHEL CPU 架构的代码生成支持。

Clang 是一款“LLVM 原生”C/C++/Objective-C 编译器，旨在提供惊人的快速编译、极其有用的错误和警告消息，并为构建优秀的源代码级工具提供一个平台。Clang Static Analyzer 是一个工具，它可以自动发现代码中的错误，并且是一个很好的例子，可以使用 Clang 前端作为库来构建这种工具来解析 C/C++代码。

此版本包含以下组件:

*   llvm-工具集-6.0-llvm-6.0.1
*   Llvm-toolset-6.0-clang-6 . 0 . 1
*   容器镜像:dev tools-beta/llvm-toolset-6.0-rhel 7

Clang/LLVM 6.0 运行于 RHEL 7 (x86_64，Power LE，aarch64，S390x)

### **Golang 1.10**

这个 Go 工具集基于 Golang 1.10，将成为第一个全面支持的版本。

围棋富有表现力、简洁、干净、高效。它的并发机制使得编写充分利用多核和联网机器的程序变得容易，而它的新颖类型系统支持灵活和模块化的程序构造。Go 可以快速编译成机器码，同时具有垃圾收集的便利和运行时反射的能力。它是一种快速的静态类型的编译语言，感觉像是一种动态类型的解释语言。

此版本包含以下组件:

*   go-toolset-1.10
*   Go-toolset-1.10-golang-1.10
*   容器镜像:dev tools-beta/go-toolset-1.10-rhel 7

Golang 1.10 运行于 RHEL 7 (x86_64，Power LE，aarch64，S390x)

## **更多信息:**

*   **[进入 Hello World w/这些编译器](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world)**
*   红帽开发者工具 [文档](https://access.redhat.com/documentation/en-us/red_hat_developer_tools)

*Last updated: October 23, 2018*