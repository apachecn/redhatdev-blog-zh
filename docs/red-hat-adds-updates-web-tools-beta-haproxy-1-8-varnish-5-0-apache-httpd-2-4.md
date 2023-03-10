# 红帽为 beta 版增加/更新了 web 工具:HAProxy 1.8，Varnish 5.0，Apache httpd 2.4

> 原文：<https://developers.redhat.com/blog/2018/04/06/red-hat-adds-updates-web-tools-beta-haproxy-1-8-varnish-5-0-apache-httpd-2-4>

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)(脚本语言、开源数据库、web 工具等。)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

#### 随着新的 Red Hat Software Collections 3.1 beta 发布，这些 web 工具现已可用:

## **新增:清漆 5.0**

这个版本的 Red Hat 软件集合包括 Varnish 5.0。Varnish Cache 是一个 web 应用程序加速器，也称为缓存 HTTP 反向代理。它安装在使用 HTTP 的 web 服务器前面，并被配置为缓存内容。Varnish 具有非常高的性能和高度可扩展的内置配置语言。它通常可以将交付速度提高 300 到 1000 倍。Varnish 只缓存 HTTP，它不支持其他 web 协议。

清漆 5.0 仅适用于 RHEL7。

包装名称:rh-varnish5

Linux 容器镜像:[RHS cl-beta/varnish-5-rhel 7](https://access.redhat.com/containers/#/search/varnish)

## **新增:** **HAProxy 1.8**

这次发布的红帽软件集合增加了一个新的 HAProxy 集合。

最新的稳定上游版本是 1.8，它包含了许多客户感兴趣的和其他红帽产品(如 OpenStack、OpenShift)所需的新特性。由于 RHEL7 中的 rebase 会破坏兼容性，最新的稳定 haproxy 被添加到 RHSCL 中。

HAProxy 是免费的开源软件，为基于 TCP 和 HTTP 的应用程序提供高可用性负载平衡器和代理服务器，这些应用程序将请求分布在多个服务器上。

HAProxy 仅可用于 RHEL7 和 x86_64 架构。目前，RHEL7 中提供了 haproxy-1.5。

包装名称:rh-haproxy18

## Apache **httpd24 已更新，包括 mod_auth_mellon**

此版本的 Red Hat Software Collections 更新了 Apache HTTP Server 版，以包含 mod_auth_mellon。Apache HTTP 是 Apache 软件基金会的一个项目，是互联网上排名第一的 HTTP 服务器。RHEL7 中的 mod_auth_mellon 包目前可用于 httpd 的基本版本。此更新包括 RHSCL httpd24 集合中的 mod_auth_mellon。

使用 Apache 2.4，web 开发人员可以获得其他“快速”web 服务器的性能，而不必切换到更新的 web 服务器，如 Nginx。许多企业利用 SAML 进行认证，也利用 Python。在 RHEL 使用 Python 3 支持的方法是通过 SCL。SCL Python 使用 SCL httpd24 用于 wsgi。SCL httpd24 现在有了 mod_auth_mellon。

现在 RHSCL http24 集合的用户可以迁移到 Django 2.x，它只支持 Python 3。许多应用程序正在放弃对 Django 1.x 的支持，现在 2.x 已经稳定和成熟了。

有哪些版本，在哪里？

*   RHSCL 包含 Apache 2.4
*   RHEL 6 包含 Apache 2.2
*   RHEL7 包含 Apache 2.4

Apache httpd24 适用于 RHEL 6 和 7；mod_auth_mellon 仅适用于 RHEL7。

包装名称:httpd24

Linux 容器映像:rhscl-beta/httpd-24-rhel7

## 参考资料:

*   参见 [Hello World](https://developers.redhat.com/products/softwarecollections/hello-world/) 快速安装软件集合。
*   [红帽集装箱目录](https://access.redhat.com/containers/)

*Last updated: June 24, 2022*