# GCC 8 和 Red Hat Enterprise Linux 6 和 7 的测试版工具

> 原文：<https://developers.redhat.com/blog/2018/10/24/gcc-8-and-tools-now-in-beta-for-red-hat-enterprise-linux-6-and-7>

我们很高兴地宣布 [Red Hat 开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) 8 beta 版将立即面向 [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/) 6 和 7 推出。此版本的关键新组件是:

*   GCC 8.2.1
*   GDB 8.2
*   更新的组件，如 SystemTap、Valgrind、OProfile 等等

要入门，请看: *[如何在红帽企业版 Linux](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)* 上安装 GCC 8。有关更多详细信息，请参见下面的“新功能”部分。

## ***关于红帽开发者工具集***

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和 web 工具，为应用程序开发人员提供最新、稳定的版本。这些红帽支持的产品被打包成 [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/) (脚本语言、开源数据库、web 工具等。)、 [红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在所有 Red Hat Enterprise Linux 开发人员订阅和大多数 Red Hat Enterprise Linux 订阅中。大多数组件也可以作为 Linux 容器映像用于跨 Red Hat 平台的混合云开发，包括:Red Hat Enterprise Linux、Red Hat OpenShift、Red Hat OpenStack 等。

## ***开发者工具集 8 新增功能:***

开发者工具集(DTS)版本 8 基于 GCC 版本 8.2.1，鼓励 DTS 用户更新到 DTS 8。GCC 8 中包含以下新的编译器特性:

*   过程间优化改进
*   性能分析驱动优化(PGO)的改进
*   链接时间优化的改进(LTO)
*   循环嵌套优化标志的变化
*   增加程序安全性的新代码生成选项(-fstack-clash-protection)
*   各种新的警告，以检测具有安全隐患的潜在错误代码
*   对 GCOV 的各种改进
*   杀毒程序已扩展到检测更多无效案例
*   此外，增加了各种前端警告。一些现有的警告已被延长。诊断信息得到了改进

GCC 8 稳定版相对于 GCC 7 系列的新变化包括:

*   英特尔 Cannonlake 支持。GCC 8 的最初目标也是 Cannonlake 的继任者英特尔 Icelake。
*   改进了 AMD Zen“Zn ver 1”微体系结构的调整。
*   ARMv8.4-A 支持以及现在正式支持 ARM Cortex-A55 和 Cortex-A75 CPU。
*   高通 Saphira CPU 支持，高通新型 ARM 服务器 CPU 核心。
*   对于 C 代码，默认为 C17，而在 C++方面，是 C++2A 的初始工作。
*   Libstdc++改进了对 C++17 的实验支持和对 C++2A 的实验支持。
*   【Fortran 2018 的准备工作。
*   围绕不同架构的 Spectre 缓解的各种工作。
*   正确-March = ARM/AAR ch 64 上的原生处理。
*   英特尔 Cilk Plus 支持已被取消，而单独的英特尔 MPX(内存保护扩展)已被弃用，可能会在 GCC 9 中被取消。

开发者工具集 8 还包括调试、优化和性能工具的更新:

*   比努蒂尔斯:2.30
*   dwz: 0.12
*   elfutils: 0.174
*   作品集:1.3.0
*   systemtap: 3.3
*   valgrind: 3.14.0
*   动态:9.3.2
*   损失:4.24
*   memstomp: 0.1.5
*   ltrace: 0.7.91
*   制造:4.2.1

带 GCC 8 的开发人员工具集在 RHEL 6 (x86_64)和 7 (x86_64、ppc64、ppc64le、aarch64、s390x)上运行

包名:`devtoolset-8`

集装箱图像:`rhscl-beta/devtoolset-8-toolchain-rhel7`和`rhscl-beta/devtoolset-8-perftools-rhel7`

## **更多信息:**

*   [如何在红帽企业 Linux](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/) 上安装 GCC 和 Clang/LLVM 6
*   [最快捷的方式你好世界](https://developers.redhat.com/products/developertoolset/hello-world/)
***   **[红帽开发者工具集页面](https://developers.redhat.com/products/developertoolset/overview/)***   [生命周期文档](https://access.redhat.com/support/policy/updates/rhscl)

    *   红帽开发者工具集 [文档](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/)**

***Last updated: March 5, 2019***