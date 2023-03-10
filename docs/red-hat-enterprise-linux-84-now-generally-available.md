# Red Hat Enterprise Linux 8.4 现已正式上市

> 原文：<https://developers.redhat.com/articles/2021/05/19/red-hat-enterprise-linux-84-now-generally-available>

[红帽企业版 Linux 8.4](https://developers.redhat.com/products/rhel/overview) ，于 4 月 27 日在[红帽峰会](https://www.redhat.com/en/summit)上预发布，现已全面上市。我们鼓励 [Linux](https://developers.redhat.com/topics/linux) 开发者下载这个最新版本并试用新软件。我们还建议将开发和生产系统升级到新的[Red Hat Enterprise Linux](/products/rhel/overview)(RHEL)8.4 版本。

## RHEL 8.4 有什么新功能？

RHEL 8.4 提供了从开发到部署的简化路径，将团队统一在一个开放平台上，包括在任何足迹上构建和管理这些系统所需的工具和分析，从数据中心到云到边缘，等等。通过使用最新的工具、编程语言和增强的容器功能，开发团队可以在生成新代码时更快地实现价值。你可以在这里了解更多关于 RHEL 8.4 提供[的内容。](https://www.redhat.com/en/blog/rhel-84-brings-continuous-stability-plus-innovation)

如果你是一个开发者，这里有一些你需要知道的关于 Red Hat Enterprise Linux 8.4 的关键亮点。

### 利用最新的数据库技术

现在，您可以在应用程序中利用最新的数据库技术:

*   PostgreSQL 13 ，现在可以通过 RHEL 应用流获得，提高了数据库性能，让开发人员能够更新他们的应用程序，尤其是在云中。
*   Redis 6 现已在 RHEL 应用流中推出，允许开发人员构建能够利用新的数据库安全增强和客户端缓存功能来提升性能的现代应用。
*   **MariaDB 10.5** 现已在 RHEL 应用流中推出，它允许您构建能够利用附加数据库功能的应用，包括 IPv6 (INET 6)数据类型支持、更细粒度的权限以及使用 Galera 插件的集群。

### 利用最新的应用运行时为您的应用提供动力

应用程序运行时更新为您的应用程序带来了新功能:

*   Python 3.9 带来了一些新的增强，包括时区感知时间戳、新的字符串前缀和后缀方法，以及字典联合操作，因此开发人员可以更新他们的应用程序。
*   **Go 1.15** 带来了小对象内存分配的改进，Go 链接器的改进，以及其他几个核心库的改进。
*   **Rust 1.49** 允许开发人员编写在低内存占用下运行的高性能应用，使其非常适合边缘用例。此外，Rust 是一种静态类型语言，很容易在编译时捕捉错误并进行维护。
*   有了最新的 **LLVM 工具集**，开发者可以利用更新的工具，以及与其他代码的兼容性，这些代码是用兼容版本的 [LLVM/Clang](/search?t=clang+llvm) 构建的。

### 与您的朋友和客户一起构建、共享和协作 RHEL 应用程序

容器更新使得在 RHEL 应用程序上构建、共享和协作变得更加容易:

*   **Red Hat Universal Base Image(**[**UBI**](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)**):**认证语言运行时容器已经更新，包含了上面列出的一些语言的容器化环境。此外，在 RHEL 8.4 中，目录中提供了一种新的微型 UBI 容器产品，以提供比最小基本容器映像更小的容器。
*   寻找特定的容器图像？通过[红帽生态系统目录](https://catalog.redhat.com/software/containers/explore)查看 [**红帽认证集装箱**](https://connect.redhat.com/explore/red-hat-container-certification) 。这使得使用 Red Hat Enterprise Linux 和 [Red Hat OpenShift](https://developers.redhat.com/topics/kubernetes/) 环境支持的应用程序流构建和部署任务关键型应用程序变得更加容易。

### 立即开始使用 RHEL 8.4

Red Hat Enterprise Linux 8.4 延续了 Red Hat 在底层计算架构方面对客户选择的承诺，可用于 x86_64、ppc64le、s390x 和 aarch64 硬件。

拥有有效订阅的开发者可以访问 [Red Hat Enterprise Linux 下载](https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.4/x86_64/product-software)。如果您是使用 Red Hat 产品的新手，请注册 Red Hat developer program 以获得针对 RHEL 的 [**个人开发者订阅**](https://developers.redhat.com/rhel8) ，该订阅可用于多达 16 个系统的生产。如需了解关于团队订阅 的 [**Red Hat 开发者的信息，请联系您的 Red Hat 客户代表。**](https://www.redhat.com/en/blog/new-year-new-red-hat-enterprise-linux-programs-easier-ways-access-rhel#Bookmark%202)

有关更多信息，请阅读完整的[发行说明。](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/)

*Last updated: August 15, 2022*