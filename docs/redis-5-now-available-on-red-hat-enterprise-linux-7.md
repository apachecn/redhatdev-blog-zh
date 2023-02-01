# Redis 5 现可在 Red Hat Enterprise Linux 7 上使用

> 原文：<https://developers.redhat.com/blog/2019/06/21/redis-5-now-available-on-red-hat-enterprise-linux-7>

[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)通过每年两个发布系列为 [Red Hat Enterprise Linux](https://developers.redhat.com/rhel8/) 提供最新、稳定版本的开发工具。作为最新软件集合 3.3 版本的一部分，我们很高兴地宣布，Redis 5 现已在 RHEL 7 上正式提供并受支持。

新的 Red Hat 软件集合包括 Redis 5.0.3。 [Redis 5](https://redis.io/) 是一个开源的内存数据结构存储，用作数据库、缓存和/或消息代理。此版本提供了多个增强功能，并修复了随早期 Red Hat Software Collections 版本发布的 3.2 版中的错误。最值得注意的是，`redis-trib`集群管理工具已经在 Redis 命令行界面中实现。

Redis 5 的主要新增功能是 Streams——一种新的类似日志的数据结构，用于存储多个字段和带有自动排序的字符串值。有关 Redis 的详细变化，请参见[版本 4.0](https://raw.githubusercontent.com/antirez/redis/4.0/00-RELEASENOTES) 和[版本 5.0](https://raw.githubusercontent.com/antirez/redis/5.0/00-RELEASENOTES) 的上游发行说明。

程序包名称:HR-redis 5

容器图像: [rhscl/redis-5-rhel7](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/redis-5-rhel7)

系统支持:适用于 x86_64、s390x、aarch64、ppc64le 的 RHEL 7

## 关于软件集合

每年两次，Red Hat 发布新版本的编译器工具集、脚本语言、开源数据库、web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)、[Red Hat Developer Toolset with GCC](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview/)，以及最近添加的编译器工具集 [Clang/LLVM、Go 和 Rust](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。

大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

## 资源

*   [红帽软件集合 3.3 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index)
*   [使用红帽软件收藏容器图片](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/using_red_hat_software_collections_container_images/index)
*   [红帽软件集合打包指南](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/packaging_guide/)
*   [软件集合的完整列表](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index#tabl-RHSCL-Components)
*   [迁移至 Redis 5](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index#sect-Migration-redis)

*Last updated: June 19, 2019*