# 红帽企业 Linux 8 测试版来了

> 原文：<https://developers.redhat.com/blog/2018/11/15/red-hat-enterprise-linux-8-beta-is-here>

## 红帽企业版 Linux 8 测试版来了

它的构建考虑到了生产稳定性和开发灵活性。

关于 RHEL 8 Beta 版有太多要说的了，但我想着重谈谈公司 [公告](http://redhat.com/en/blog/powering-its-future-while-preserving-present-introducing-red-hat-enterprise-linux-8-beta) 中的几点，该公告强调 Red Hat Enterprise Linux 8 Beta 版是一个开发者平台:

*   简化应用开发——通过更少的设置和配置工作，您可以更快地开始编写代码
*   对于刚接触 Linux 的开发者来说,是最简单的 RHEL
*   适用于传统应用和云/容器应用，其中包含许多新工具
*   已经交付了许多工具来构建和测试应用

现在让我们来放大一下这些是什么意思。

点击此处下载 RHEL 8 测试版。

## “太慢，太快”

在讨论开发包的可用性和支持时，我们得到了很多这样的反馈。为了解决这种二分法，Red Hat Enterprise Linux 8 Beta 内置了[应用流](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)的概念来交付用户空间包(编程语言、编译器、数据库等)。)更简单、更灵活——这解决了“太慢”的问题。对于“太快”的需求，也有“核心”组件的生命周期与操作系统相同——10 年。用户通常在每个分组中都有一个版本。应用程序流——把它们想象成“软件集合之子”——是一种更简单的方式，通过改进的安装、使用和重用来实现这种模块化

## 操作系统架构

为了简化您的开发体验，Red Hat Enterprise Linux 8 已经过简化，因为它基于两个预启用的存储库:

*   BaseOS——“大多”OS 内容
*   应用流(AppStream) -大部分开发者工具都在这里

**BaseOS** 中的内容旨在提供底层操作系统功能的核心集，为所有安装提供基础。这些内容以传统的 RPM 格式提供。有关 BaseOS 包的列表，请参见附录 A 中的 [，BaseOS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/) 中的 *包。*

**应用流** 中的内容旨在提供 BaseOS 所不具备的附加功能。该内容集包括额外的用户空间应用程序、运行时语言和数据库，支持各种工作负载和用例，例如“太慢”。AppStream 中的内容有两种格式——熟悉的 RPM 格式和称为 *模块* 的 RPM 格式的扩展，它们只是打包使用/重用。构建为 *模块* (例如 PHP)的组件被称为 *流* (例如 PHP 7.2)。有关 AppStream 包的列表，请参见附录 B 中的 [和 AppStream](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/) 中的 *包。*

## Linux 容器

Linux 容器是数字化转型(或云原生或你想怎么称呼它)的关键组件，因此 Red Hat 的轻量级、基于开放标准的容器工具包现在完全受支持并包含在 Red Hat Enterprise Linux 8 中。考虑到企业 IT 安全需求， [**Buildah**](https://www.redhat.com/en/blog/daemon-haunted-container-world-no-longer-introducing-buildah-10) (容器构建)、[**Podman**](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)(运行容器)和[**Skopeo**](https://blog.openshift.com/ting-container-images-between-registries-with-skopeo/)(共享/查找容器)帮助开发人员更快、更高效地查找、运行、构建和共享容器化的应用程序，这要归功于这些工具的分布式和无后台特性。

## 开发者工具

Red Hat Enterprise Linux 8 包括几十种工具，我们已经为您准备了一些入门指南。在[【developers.redhat.com/rhel8】](https://developers.redhat.com/rhel8)上读到它们。我们将在整个测试版中添加更多的操作方法，所以请访问

## 下载它

有三种方法可以访问 RHEL 8 测试版，它们基于您与 Red Hat 的现有关系:

1.  当前红帽开发者会员可以在 [红帽开发者](https://developers.redhat.com/rhel8) 登录并下载今日 。
2.  如果你不是 Red Hat 开发者会员，注册(免费且简单)并下载 RHEL 8 测试版。
3.  如果你是当前红帽企业 Linux 企业客户，登录 [客户门户](https://access.redhat.com/products/red-hat-enterprise-linux/beta) 参与 RHEL 8 客户测试。

## 资源

[阅读](http://redhat.com/en/blog/powering-its-future-while-preserving-present-introducing-red-hat-enterprise-linux-8-beta) 关于更多红帽企业 Linux 8 Beta 版关于新功能的公告:

*   联网
*   安全
*   系统管理
*   文件系统和存储

点击此处下载 RHEL 8 测试版。

参考文献:

*   [面向应用开发者的 RHEL 8 Beta 信息页面](https://developers.redhat.com/rhel8/)
*   [蟒蛇上 RHEL 8](https://wp.me/p8e0as-2fsJ)
*   [RHEL 上的 node . js 8](http://developers.redhat.com/rhel8/hw/nodejs)
*   [T1。网芯上 RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8-beta)
*   [Java 8 和 11 上 RHEL 8](https://developers.redhat.com/blog/2018/12/10/install-java-rhel8/)
*   [红帽企业版 Linux 8 beta 文档页](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8-beta/)

*Last updated: October 4, 2022*