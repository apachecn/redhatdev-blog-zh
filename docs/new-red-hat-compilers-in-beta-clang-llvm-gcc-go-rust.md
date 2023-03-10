# 新的红帽编译器工具集测试版:Clang 和 LLVM，GCC，Go，Rust

> 原文：<https://developers.redhat.com/blog/2018/04/06/new-red-hat-compilers-in-beta-clang-llvm-gcc-go-rust>

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)(脚本语言、开源数据库、web 工具等。)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

**新的/更新的编译器工具集有:**

## **GCC 编译器工具集的新版本:带有 GCC 7.3 的 Red Hat Developer 工具集 7.1**

这是 GCC 7 最新稳定上游更新的更新。鼓励开发人员工具集(DTS)用户更新到 DTS 7.1。

最新的 GNU Compiler Collection 主要版本 GCC 7.3 带来了大量的新功能，包括对当前 C++17 草案的实验性支持、更好的诊断和改进的优化器，以及许多新的过程内和过程间优化。关于诊断，GCC 7 带来了改进的位置、位置范围、拼写错误标识符的建议、选项名称、修复提示和新的警告。

DTS 7 可用于以下体系结构:

*   x86_64 (RHEL 6 & 7 )
*   ppc64le (RHEL 6)
*   aach 64(rhel 6)
*   s390x (RHEL 6)

**在哪里可以找到其他版本的 GCC 和 DTS？**

*   DTS 6.1 的 GCC 版本为 6.3
*   DTS 4.1 具有 GCC 版本 5
*   (没有 DTS 5)
*   RHEL7 具有 GCC 版本 4.8
*   RHEL6 具有 GCC 版本 4.4
*   DTS 7 中更新的 DTS 工具

Linux 容器映像:RHS cl-beta/dev toolset-7-tool chain-rhel 7

## Clang 和 LLVM 编译器工具集的新版本:Clang 和 LLVM 5.0

Clang 是一个“LLVM 原生”C/C++/Objective-C 编译器，旨在提供惊人的快速编译、极其有用的错误和警告消息，并为构建优秀的源代码级工具提供平台。Clang Static Analyzer 是一个工具，它可以自动发现代码中的错误，并且是一个很好的例子，可以使用 Clang 前端作为库来构建这种工具来解析 C/C++代码。LLVM 项目是模块化和可重用的编译器和工具链技术的集合。LLVM 核心库提供了一个现代的独立于源和目标的优化器，以及对 RHEL CPU 架构的代码生成支持。

Clang 和 LLVM 工具集将在 devtools repo 中发布，仅作为 RHEL 7 的技术预览版。我们鼓励客户使用和评估编译器，但不鼓励他们为生产构建应用程序。在 LLVM 工具集被认为对生产支持足够稳定之前，计划进行频繁的更新，但不一定是向后兼容的。目前 RHEL7 中没有 Clang 和 LLVM 工具集。

Clang 和 LLVM 工具集是 RHEL 7 的技术预览版，适用于:

*   x86_64
*   ppc64le
*   aarh64 足球俱乐部
*   s390x

此版本中包括以下软件包:

*   llvm 工具集 7
*   llvm-工具集-7-clang

## **新版 Go 编译器工具集:** **Golang 1.8.7**

此次发布的 Go 工具集引入了 Golang1.8.7 编译器的新版本，供 RHEL 的客户和合作伙伴使用。

Go 富于表现力，简洁，干净，高效。它的并发机制使得编写充分利用多核和联网机器的程序变得容易，而它的新颖类型系统支持灵活和模块化的程序构造。Go 可以快速编译成机器码，同时具有垃圾收集的便利和运行时反射的能力。它是一种快速的静态类型的编译语言，感觉像是一种动态类型的解释语言。

目前，RHEL7 的可选通道中提供了 Golang 编译器。从长远来看，可选的编译器将会被 devtools 中新的 Go 工具集所取代。

RHEL 开发人员现在拥有最新稳定版本的 upstream Go 编译器，用于 RHEL7 上的应用程序开发。Go 工具集将作为技术预览版在 devtools 中发布。我们鼓励客户使用和评估编译器，但不鼓励他们为生产构建应用程序。在 Go 工具集被认为对生产支持足够稳定之前，计划进行频繁的更新，但不一定是向后兼容的。

Go 工具集是 RHEL 7 的技术预览版，可用于:

*   x86_64
*   ppc64le
*   aarh64 足球俱乐部
*   s390x

包名:go-toolset-7-golang

## **新版 Rust 编译器工具集:** **Rust 1.24**

Rust 工具集的第一个版本是基于 Rust 版本 1.20 的。

Rust 是一种开源系统编程语言，由 Mozilla 和一个志愿者社区创建，旨在帮助开发人员创建快速、安全的应用程序，充分利用现代多核处理器的强大功能。它防止分段错误并保证线程安全，所有这些都是通过简单易学的语法实现的。此外，Rust 提供了零成本的抽象、移动语义、有保证的内存安全、没有数据竞争的线程、基于特征的泛型、模式匹配、类型推理和高效的 C 绑定，以及最小的运行时大小。

Cargo 是 Rust 的包管理器和构建工具。它允许 Rust 项目声明与特定版本需求的依赖关系。Cargo 将解析完整的依赖图，根据需要下载包，并构建和测试整个项目。

Rust 随 RHS cl 3.0(2017 年末)添加到 RHEL devtools 频道。

Rust 工具集是 RHEL 7 的技术预览版，可用于:

*   x86_64
*   ppc64le
*   aarh64 足球俱乐部
*   s390x

此版本中包括以下软件包:

*   rust 工具集-7
*   铁锈-工具集-7-铁锈
*   防锈工具集-7-货物

## 参考资料:

*   开发者工具集和 GCC 7.3 [Hello World](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/) 和[发行说明](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/)
*   Clang/LLVM 5.0，Go 1.8.7，Rust 1.24: [Hello World](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/) 和[发行说明](https://access.redhat.com/documentation/en/red-hat-developer-tools?version=2018.2%20Beta)
*   [红帽集装箱目录](https://access.redhat.com/containers/)

*Last updated: November 15, 2018*