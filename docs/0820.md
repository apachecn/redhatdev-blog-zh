# Red Hat Enterprise Linux 8 现已正式上市

> 原文：<https://developers.redhat.com/blog/2019/05/07/red-hat-enterprise-linux-8-now-generally-available>

我认为 Red Hat Enterprise Linux 8 是我们交付的对开发人员最友好的 Red Hat Enterprise Linux，我希望你同意这一点。让我们言归正传，或者说是编码，这样你就可以自己看了。可以看一下[红帽公司新闻稿](https://www.redhat.com/en/about/press-releases/red-hat-enterprise-linux-8-every-enterprise-every-cloud-every-workload)。

在本文中，我将快速回顾 Red Hat Enterprise Linux 8 的特性(架构、容器)，介绍非常新颖和酷的 Red Hat Universal Base Image (UBI ),并提供一个方便的开发人员资源列表，帮助您开始使用 Red Hat Enterprise Linux 8。

**TL；博士**

[立即下载 RHEL 8](http://developers.redhat.com/rhel8)

[下载 RHEL 8 图像](https://access.redhat.com/containers/#/search/ubi8)

## 红帽企业 Linux 8 架构

为了简化您的开发体验，Red Hat Enterprise Linux 8 有三个预启用的存储库:

*   **BaseOS**——“大部分”有操作系统内容
*   **应用流**(AppStream)——大多数开发者工具都在这里
*   **CodeReady Builder** — 附加库和开发工具

BaseOS 中的内容旨在提供底层操作系统功能的核心集，为所有安装提供基础。这些内容以传统的 RPM 格式提供。有关 BaseOS 软件包的列表，请参见 RHEL 8 软件包清单。

应用流，本质上是下一代软件集合 **，** 旨在提供 BaseOS 所不具备的额外功能。这个内容集包括附加的用户空间应用程序、运行时语言、数据库、web 服务器等。支持各种工作负载和使用案例。对你来说，网就是简单地使用你想要的组件和版本。一旦有市场需求，就会添加更新的稳定版本的组件。

## Linux 容器

Linux 容器是云原生开发和微服务的关键组件，因此 Red Hat 的轻量级、基于开放标准的容器工具包现在完全受支持并包含在 Red Hat Enterprise Linux 8 中。考虑到企业 IT 安全需求， [Buildah](https://www.redhat.com/en/blog/daemon-haunted-container-world-no-longer-introducing-buildah-10) (构建容器)、 [Podman](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/) (运行容器)和 [Skopeo](https://blog.openshift.com/promoting-container-images-between-registries-with-skopeo/) (共享/查找容器)帮助开发人员更快、更有效地查找、运行、构建和共享容器化的应用程序——这要归功于这些工具的分布式特性，更重要的是，它们的无后台特性。

## 介绍通用基础映像

源自 Red Hat Enterprise Linux，Red Hat Universal Base Image (UBI)提供了一个可自由再分发的企业级基本容器映像，开发人员可以在其上构建和交付他们的应用程序。这意味着你可以在 UBI 中封装你的应用程序，并将其部署到任何地方。当然，当部署在 Red Hat Enterprise Linux 上时，它将更加安全并支持 Red Hat，但现在您有了选择。红帽企业版 Linux 7 和 8 分别有单独的 UBI 7 和 UBI 8 版本。在 [Red Hat Universal Base Image 简介](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image-0)中了解更多信息。

## Red Hat Enterprise Linux 8 开发者资源

在过去的几个月里，我们专门为 Red Hat Enterprise Linux 8 制作了许多操作指南文档。如果你错过了它们，这里有一个列表:

*   [应用流简介](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)—这是一本关于 Red Hat Enterprise Linux 8 如何以开发人员为中心进行重新架构的初级读本
*   [Red Hat Enterprise Linux 8 备忘单](https://developers.redhat.com/cheat-sheets/red-hat-enterprise-linux-8/)—新 Red Hat Enterprise Linux 8 命令的快速参考，以及更常用的开发人员工具列表
*   [Builder Repo 简介](https://developers.redhat.com/blog/2018/11/15/introducing-codeready-linux-builder/)—阅读它是什么以及为什么你会觉得它很方便
*   [安装 Java 8 和 11](https://developers.redhat.com/blog/2018/12/10/install-java-rhel8/)—不再多说
*   [使用 Apache、MySQL 和 PHP 建立 LAMP 堆栈](https://wp.me/p8e0as-2tQZ)
*   [构建没有守护进程的容器](https://developers.redhat.com/blog/2018/11/20/buildah-podman-containers-without-daemons/)—介绍如何使用 Podman、Buildah 等等。
*   XDP [第一部](https://developers.redhat.com/blog/2018/12/06/achieving-high-performance-low-latency-networking-with-xdp-part-1/)T4[第二部](https://developers.redhat.com/blog/2018/12/17/using-xdp-maps-rhel8/)
*   [使用 eBPF 进行网络调试](https://developers.redhat.com/blog/2018/12/03/network-debugging-with-ebpf/)
*   [在 VirtualBox 上快速安装](https://developers.redhat.com/rhel8/install-rhel8-vbox/)
*   [在裸机上快速安装](https://developers.redhat.com/rhel8/install-rhel8/)
*   [蟒蛇在 RHEL 8](https://developers.redhat.com/blog/2018/11/14/python-in-rhel-8-3)
*   [快速安装:Node.js](https://developers.redhat.com/rhel8/hw/nodejs/)
*   [什么，RHEL 8 里没有蟒蛇？](https://wp.me/p8e0as-2tAd)
*   [快速安装:Python](https://developers.redhat.com/rhel8/hw/python/)
*   [红帽通用基础映像(UBI)简介](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image-0)

## Red Hat 开发者订阅

红帽开发者会员已经享受了 3 年多的免费开发者订阅，RHEL 8 现在自动成为其中的一部分。如果你的公司需要开发者支持，有几个 [Red Hat Enterprise Linux 开发者订阅](https://www.redhat.com/en/store/linux-platforms)选项也有 Red Hat 支持。

## RHEL 八号文件

*   [安装指南](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/performing_a_standard_rhel_installation/index)
*   [发行说明](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/8.0_release_notes/index)
*   [包裹清单](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/package_manifest/index)
*   [构建、运行和管理容器](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index)

*Last updated: February 24, 2022*