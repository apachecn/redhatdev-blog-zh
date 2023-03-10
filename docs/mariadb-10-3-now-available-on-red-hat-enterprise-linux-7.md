# MariaDB 10.3 现已在 Red Hat Enterprise Linux 7 上提供

> 原文：<https://developers.redhat.com/blog/2019/06/25/mariadb-10-3-now-available-on-red-hat-enterprise-linux-7>

[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)每年通过两个发布系列为 Red Hat Enterprise Linux 提供最新、稳定版本的开发工具和组件。作为红帽软件集合 3.3 发布的一部分，我们很高兴地宣布 [MariaDB](https://mariadb.org) 10.3 现可用于[红帽企业 Linux 7](https://developers.redhat.com/products/softwarecollections/download/) 。

新的 MariaDB 软件集合提供了 MariaDB 10.3.13，它引入了几个新功能和错误修复，包括:

*   一个新的`rh-mariadb103-mariadb-java-client`包，它为 MariaDB 和 MySQL 数据库服务器提供了 Java 数据库连接(JDBC)连接器。该连接器支持 MariaDB 和 MySQL 版本 5.5.3 和更高版本，以及 JDBC 版本 4.2，并且它需要 Java 运行时环境(JRE)版本 8 或 11。
*   [系统版本化表](https://mariadb.com/kb/en/library/system-versioned-tables/)，使您能够存储变更历史。
*   [不可见列](https://mariadb.com/kb/en/library/invisible-columns/)，除非明确调用，否则不会列出。
*   一个新的`InnoDB` 的[即时`ADD COLUMN`操作，不需要重建整个表。](https://mariadb.com/kb/en/library/instant-add-column-for-innodb/)

包装名称:rh-mariadb103

容器图像: [rhscl/mariadb-103-rhel7](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/mariadb-103-rhel7)

系统支持:适用于 x86_64、s390x、aarch64、ppc64le 的 RHEL 7

## 关于软件集合

每年两次，Red Hat 发布新版本的编译器工具集、脚本语言、开源数据库、web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)、[Red Hat Developer Toolset with GCC](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview/)，以及最近添加的编译器工具集 [Clang/LLVM、Go 和 Rust](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview/) 。所有这些都是可安装的，包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发者订阅中。

大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、 [Red Hat OpenShift 容器平台](https://developers.redhat.com/openshift/)等的混合云开发。

## 资源

*   [红帽软件集合 3.3 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index)
*   [使用红帽软件收藏容器图片](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/using_red_hat_software_collections_container_images/index)
*   [红帽软件集合打包指南](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/packaging_guide/)
*   [软件集合的完整列表](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index#tabl-RHSCL-Components)
*   [迁移到 MariaDB 10.3](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.3_release_notes/index#sect-Migration-MariaDB)

*Last updated: June 24, 2019*