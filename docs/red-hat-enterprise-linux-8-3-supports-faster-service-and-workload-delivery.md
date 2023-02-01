# Red Hat Enterprise Linux 8.3 支持更快的服务和工作负载交付

> 原文：<https://developers.redhat.com/blog/2020/11/06/red-hat-enterprise-linux-8-3-supports-faster-service-and-workload-delivery>

Red Hat Enterprise Linux (RHEL) 8.3 于上周发布，现已正式上市。我们鼓励 [Linux](https://developers.redhat.com/topics/linux) 开发者[下载这个更新并尝试一下](https://developers.redhat.com/products/rhel/download)。我们还建议将开发和生产系统更新到新的 8.3 版本。本文概述了 RHEL 8.3 的开发者亮点，包括针对 [Node.js](https://developers.redhat.com/blog/category/node-js/) 14、Ruby 2.7、 [PHP](https://developers.redhat.com/blog/category/php/) 7.4、 [GCC 工具集 10](https://developers.redhat.com/blog/2020/09/24/new-c-features-in-gcc-10/) 等的新应用流。

## RHEL 8.3 有什么新功能？

RHEL 8.3 通过为开放混合云提供安全一致的基础，兑现了 RHEL 8 的承诺。此更新中的功能增强有助于开发人员更快、更轻松地交付服务和工作负载，适用于任何时间、任何地点的任何应用。

以下是 RHEL 8.3 的一些新特性，旨在帮助开发人员实现云计算、[edge](https://developers.redhat.com/topics/edge-computing)以及其他:

*   **应用流**帮助开发人员使用最新工具的支持版本进行创新，而不会牺牲应用维护所需的早期版本。RHEL 8.3 包括用于 [Node.js](https://developers.redhat.com/topics/nodejs) 14、 [Ruby](https://developers.redhat.com/blog/category/ruby/) 2.7、 [PHP](https://developers.redhat.com/blog/category/php/) 7.4、GCC 工具集 10 等等的应用流。
*   **Podman Remote API 2.0** 让组织能够轻松保留之前依赖于 RHEL 7 中 Docker 容器引擎的代码和工具。
*   **容器化的 RHEL 容器工具**通过让开发人员使用 Buildah、Skopeo 和 [Podman](https://developers.redhat.com/blog/2020/09/25/rootless-containers-with-podman-the-basics) 来构建和运行符合标准(开放容器倡议)的容器映像，增加了灵活性。
*   **Image Builder push-to-cloud**允许管理员构建定制的机器映像，并自动上传到云提供商的清单中。
*   **云的就地升级**通过支持从 RHEL 7 到 RHEL 8 的就地升级，简化云部署的生命周期管理。

## RHEL 8.3 中的新应用程序流

RHEL 8.3 包括以下开发人员工具更新，这些更新以应用程序流的形式提供:

*   GCC 工具集 10 :编译器、工具链、调试器和其他关键开发工具的精选集合。[红帽开发者工具集 10](https://developers.redhat.com/blog/2020/11/04/red-hat-software-collections-3-6-now-available-in-beta/) 的基础，GNU 编译器集合(GCC) 10.2.1 是流行的[开源](https://developers.redhat.com/topics/open-source)编译器集合的最新更新。GCC 工具集 10 还为 C/C++ 和 Fortran 更新了调试和性能工具。
*   **Go 工具集、LLVM 工具集和 Rust 工具集**:这些流行的编译器在最新版本更新中提供了新的特性。
*   **Node.js 14** :引入了一个[升级的 V8 引擎](https://developers.redhat.com/blog/2020/10/20/get-started-with-node-js-14-on-red-hat-openshift/)，一个新的实验性 WebAssembly 系统接口(WASI)，实验性异步本地存储 API，以及众多的 bug 和安全修复。
*   Nginx 1.18 :这个流行的 web 和代理服务器版本提供了对 Nginx 1.16 的错误修复、安全修复、新特性和增强。Nginx 1.18 包括对 HTTP 请求速率和连接限制的增强，以及新的代理协议变量。
*   **Git 2.27** :为开发人员增加了许多特性，比如切换和恢复命令、配置变量以及配置 SSL 与代理通信的选项。
*   **Ruby 2.7** :带来了性能提升、bug 和安全修复，以及新特性，包括压缩垃圾收集器和交互式 Ruby Shell 中的多行编辑。
*   **PHP 7.4** : Bug 修复和增强包括新的外来函数接口(FFI)，这是一个实验性的扩展，能够调用本机函数，访问本机变量，创建和访问 C 库中定义的数据结构。
*   Perl 5.30 :增加新特性，取消或删除几个模块。

所有新的 RHEL 8.3 应用流都可以通过[红帽生态系统目录](https://catalog.redhat.com/software/containers/explore)作为[红帽认证容器](https://connect.redhat.com/explore/red-hat-container-certification)获得。我们的目标是使用 Red Hat Enterprise Linux 和 [Red Hat OpenShift](https://developers.redhat.com/topics/kubernetes/) 环境支持的应用程序流，轻松构建和部署任务关键型应用程序。

有关更多信息，请阅读全面的 [RHEL 8.3 发行说明](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/)。

## 立即开始使用 RHEL 8.3

Red Hat Enterprise Linux 8.3 在底层计算架构方面继续为客户提供选择，可用于 x86_64、ppc64le、s390x 和 aarch64 硬件。如果您是一名拥有有效订阅的开发人员，您可以通过 [Red Hat Enterprise Linux 下载页面](https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.3/x86_64/product-software)访问 RHEL 8.3。如果你是红帽企业版 Linux 的新手，你可以使用下载页面注册并下载[红帽企业版 Linux for developers](https://developers.redhat.com/topics/linux) 。

[点击](https://www.redhat.com/en/blog/rhel-83-and-edge)了解更多关于 Red Hat Enterprise Linux 8.3 的信息。

*Last updated: November 17, 2020*