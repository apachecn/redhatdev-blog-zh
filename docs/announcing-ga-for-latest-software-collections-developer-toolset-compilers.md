# 宣布面向最新软件集合、开发人员工具集和编译器的正式上市

> 原文：<https://developers.redhat.com/blog/2018/05/03/announcing-ga-for-latest-software-collections-developer-toolset-compilers>

我们很高兴地宣布:正式上市

*   红帽软件集合 3.1(包括 Ruby 2.5、Perl 2.26、PHP 7.0.27、PostgreSQL 10、MongoDB 3.6、Varnish 5、HAProxy 1.8、Apache 2.4 更新)
*   红帽开发者工具集 7.1 (GCC 7.3)
*   Clang/LLVM 5.0，Go 1.8.7，Rust 1.25.0

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些红帽支持的产品被打包成 [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/) (脚本语言、开源数据库、web 工具等。)、 [红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。大多数 Red Hat 软件集合和 Red Hat 开发人员工具集组件也可以作为 Linux 容器映像用于跨 Red Hat 平台的混合云开发，包括但不限于:Red Hat Enterprise Linux 和 Red Hat OpenShift 容器平台。

**红帽软件集合 3.1 带来了以下新/更新版本:**

## 脚本语言:

**新:Ruby 2.5**

*   Ruby 2.5 只适用于 RHEL 7。
*   包装名称:rh-ruby25
*   Linux 容器镜像:rhscl/ruby-25-rhel7

**新:Perl 5.26**

*   Per 5.26 仅适用于 RHEL 7。
*   包装名称:rh-perl526
*   Linux 容器镜像:rhscl/perl-526-rhel7

**PHP 7.0 已更新至 7.0.27**

*   PHP 7.0.27 是针对 RHEL 6 和 RHEL 7 的；仅限 x86_64。
*   包装名称:rh-php70
*   Linux 容器镜像:rhscl/php-70-rhel7
*   注意:PHP 7.1.8 仍然可用

## 开源数据库:

**新:PostgreSQL 10**

*   PostgreSQL 10 仅适用于 RHEL7。
*   包:rh-postgresql10
*   Linux 容器镜像:rhscl/postgresql-10-rhel7

**新:MongoDB 3.6**

*   MongoDB 3.6 只针对 RHEL7。
*   包装名称:rh-mongodb36
*   Linux 容器镜像:rhscl/mongodb-36-rhel7

**MySQL 5.7 已更新，支持 ppc64le、s390x 和 aarch64**

*   包名:rh-mysql57

## 网络工具:

**新:清漆 5.0**

*   清漆 5.0 仅适用于 RHEL7。
*   包装名称:rh-varnish5
*   Linux 容器镜像:rhscl/varnish-5-rhel7

**新:HAProxy 1.8**

*   HAProxy 1.8 仅适用于 RHEL7 和 x86_64。
*   包装名称:rh-haproxy18

**更新了 Apache httpd24，包括 mod_auth_mellon**

*   mod_auth_mellon 是给 RHEL 6 & 7 的。
*   包名:httpd24
*   Linux 容器镜像:rhscl/httpd-24-rhel7

## 带有 GCC 7.3 的 Red Hat 开发工具集 7.1

DTS-7 中包含以下工具链更新:

*   海湾合作委员会:7.3
*   还有大量的附加工具——参见开发者工具集用户指南。
*   月食 4.7.3a
*   Linux 容器映像:RHS cl/dev toolset-7-tool chain-rhel 7

## 互补编译器

以下内容适用于这些 RHEL7 架构:

*   x86_64
*   ppc64le
*   aach 64
*   s390x

**Clang 和 LLVM 编译器工具集 5.0**

*   Clang 和 LLVM 工具集适用于 RHEL 7 号
*   此版本包含以下软件包:
*   llvm-工具集-7-llvm
*   llvm-toolset-7-clang

**Golang 1.8.7**

*   Go 工具集适用于 RHEL 7 号
*   包名:go-toolset-7-golang

Rust 编译器工具集的新版本:Rust 1.25

*   Rust 工具集是为 RHEL 7 号准备的
*   此版本包含以下软件包:
*   rust-toolset-7-rust
*   防锈工具组-7-货物

以上 3 款编译器可作为技术预览版使用。

## **参考文献:**

软件集合

*   参见[Hello World](https://developers.redhat.com/products/softwarecollections/hello-world/)快速安装软件集合。
*   [RHSCL 3.1 发行说明及使用容器镜像](https://access.redhat.com/documentation/en-us/red_hat_software_collections/)

开发工具集、其他编译器、Eclipse IDE

*   开发者工具集和 GCC 7.3[Hello World](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/)和 [产品文档](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/)
*   Clang/LLVM、Go、Rust:[Hello World](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/)和 [产品文档](https://access.redhat.com/documentation/en/red-hat-developer-tools?version=2018.2%20Beta)
*   Eclipse IDE 4 . 7 . 3 a[产品文档 ](https://access.redhat.com/documentation/en/red-hat-developer-tools?version=2018.2%20Beta)

红帽集装箱目录

*   [红帽集装箱目录](https://access.redhat.com/containers/)

*Last updated: November 15, 2018*