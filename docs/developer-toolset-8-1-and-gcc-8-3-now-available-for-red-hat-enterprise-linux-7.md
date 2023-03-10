# 开发者工具集 8.1 和 GCC 8.3 现可用于 Red Hat Enterprise Linux 7

> 原文：<https://developers.redhat.com/blog/2019/06/20/developer-toolset-8-1-and-gcc-8-3-now-available-for-red-hat-enterprise-linux-7>

[Red Hat Developer Toolset](https://developers.redhat.com/products/developertoolset/) 通过每年两个发布系列为 [Red Hat Enterprise Linux](http://developers.redhat.com/rhel8/) 提供 GCC、GDB 和一套补充开发工具。我们很高兴地宣布，带有 GCC 8.3 的开发人员工具集 8.1 现已推出，并在 Red Hat Enterprise Linux 7 上得到支持。

Red Hat Developer Toolset 8.1 版本包括许多[增强和变化](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/8/html/user_guide/appe-changes_in_version_8.1)，但这里有一些亮点:

*   GCC 8.3.1
*   GDB 8.2
*   binutils 2.30
*   埃尔夫蒂斯 0.176
*   Valgrind 3.14.0

包名:`devtoolset-8`

容器镜像:[RHS cl/dev toolset-8-tool chain-rhel 7](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/devtoolset-8-toolchain-rhel7)；[RHS cl/dev toolset-8-perf tools-rhel 7](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/devtoolset-8-perftools-rhel7)

系统支持:适用于 x86_64、IBM Z、aarch64、ppc64le 的 RHEL 7

## **关于软件集合**

每年两次，Red Hat 发布新版本的编译器工具集、脚本语言、开源数据库、web 工具等。，以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)、[Red Hat Developer Toolset with GCC](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview/)，以及最近添加的编译器工具集 [Clang/LLVM、Go 和 Rust](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview/) 。所有这些都是可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。

大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

## **资源**

*   [Red Hat 开发者工具集用户指南](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/8/html-single/user_guide/index)
*   [红帽软件集合 3.3 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index)
*   [使用红帽软件收藏容器图片](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/using_red_hat_software_collections_container_images/index)
*   [红帽软件集合打包指南](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/packaging_guide/)
*   [软件集合的完整列表](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index#tabl-RHSCL-Components)

*Last updated: June 19, 2019*