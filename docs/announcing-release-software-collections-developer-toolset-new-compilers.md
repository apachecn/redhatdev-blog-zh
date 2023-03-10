# 宣布发布软件集合、开发工具集、新编译器

> 原文：<https://developers.redhat.com/blog/2017/10/25/announcing-release-software-collections-developer-toolset-new-compilers>

I am pleased to announce the general availability of numerous Red Hat curated collections of the latest, stable application development tools, languages, compilers, databases, and more. Created for Red Hat Enterprise Linux, developers can access these via the following open source offerings:

*   红帽软件集合
*   Red Hat 开发工具集
*   新 RHEL 编译器:Clang/LLVM、Go 和 Rust

作为 Linux 容器交付的组件也可以在 Red Hat OpenShift 容器平台上使用。

## **新的和更新的 Red Hat 软件集合组件**

Red Hat Software Collections 3 在将无处不在的 Apache HTTP server*更新到版本 2.4.27 的同时，增加了许多新组件。Red Hat 软件集合 3 的新功能有:

*   MariaDB 10.2*
*   Maven 3.5
*   MongoDB 3.4*
*   nginx 1.12*
*   Node.js 8.6*
*   PHP 7.1*
*   PostgreSQL 9.6*
*   Python 3.6*

## **主要的 Red Hat 开发者工具集更新**

Red Hat Developer Toolset* 7 为 **GCC 7.2** 增加了 GNU 编译器集合(GCC)的重大更新，使开发人员能够编译一次并跨多个版本的 Red Hat Enterprise Linux 部署应用。Red Hat 开发工具集还更新了其他支持组件，包括 GDB 调试器、SystemTap 跟踪和探测工具、Valgrind(一个动态分析框架和工具)、strace 系统调用跟踪器等等。

## **新的开源编译器**

此外，随着编程语言需求和消费的扩大，开发人员现在可以访问三种新的受支持的开源编译器[技术预览](https://access.redhat.com/support/offerings/techpreview):

*   Clang/LLVM 4 . 0 . 1 *-LLVM 编译器后端和核心库提供了一个独立于源代码和目标的现代优化器，而 Clang 是一个 LLVM 原生 C/C++/Objective-C 编译器前端，有助于提供更快的编译、直观的错误和警告消息，以及一个用于构建源代码级工具的平台。它还包括 Clang Static Analyzer，这是一个帮助自动发现编写代码中的 bug 的工具。

*   **Go 1 . 8 . 3 ***-Go 编译器以其非常快速的编译而闻名，它支持 Go 语言，这是一种富有表现力、简洁、干净和高效的语言，有助于构建针对多核和联网机器优化的灵活和模块化程序。

*   **Rust 1.20*** - Rust 旨在帮助创建针对现代多核处理器优化的更快、更安全的应用。它旨在通过简单易学的语法来防止分段错误和提高线程安全性。

### **新架构支持**

认识到不同的 CPU 架构解决不同的 IT 问题，Red Hat**现在支持额外的架构**作为 Red Hat 软件集合 3 的一部分。除了 x86_64 之外，Red Hat Software Collections 3 和 Red Hat Developer Toolset 7 中管理和支持的组件现在可以在 **64 位 ARM 体系结构(aarch64)、IBM z Systems 和 IBM Power little endian 上使用。**这扩展了 IT 组织在满足其独特需求的架构上使用最新稳定的开源工具构建受支持的企业应用的能力。

### **Linux 容器支持和打包**

随着现代应用开发发展到包括微服务开发和复合工作负载，开发人员越来越需要容器化的工具和支持组件，以与传统工具相同的方式支持，来为云原生世界构建企业应用。考虑到这一点，除了 rpm 之外，通过 [Red Hat 容器目录](https://access.redhat.com/containers)，这些软件组件中的许多都可以作为容器映像获得。

### **可用性**

Red Hat Software Collections 3、Red Hat Developer Toolset 7 和三个技术预览版编译器现已提供受支持的 Red Hat Enterprise Linux 订阅、所有 Red Hat Enterprise Linux 开发人员订阅以及作为 Red Hat Developer Program 一部分的免费 Red Hat Enterprise Linux 开发人员订阅。

*打包成 Linux 容器的组件也可以在 Red Hat OpenShift 容器平台上使用。

### **关于软件集合**

这些产品在独立于 Red Hat Enterprise Linux 的生命周期中交付，发布频率更高，将开发灵活性与生产稳定性联系起来，帮助 IT 组织创建可以更放心地部署到生产中的企业应用程序。

### **附加资源**

*   阅读更多关于最新版本的[红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/)，以及新的 [Clang/LLVM、Go 和 Rust 编译器](https://developers.redhat.com/products/clang-llvm-go-rust/overview/)。
*   了解关于[红帽开发者计划](http://developers.redhat.com)的更多信息
*   了解更多关于免费 Red Hat Enterprise Linux 开发人员订阅的信息
*   [使用 Red Hat 软件集合 3 作为容器映像](https://access.redhat.com/documentation/en/red-hat-software-collections/)

*Last updated: November 15, 2018*