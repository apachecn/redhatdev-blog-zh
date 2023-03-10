# 红帽软件集合 3.5 为红帽企业版 Linux 7 带来了更新

> 原文：<https://developers.redhat.com/blog/2020/05/29/red-hat-software-collections-3-5-brings-updates-for-red-hat-enterprise-linux-7>

[Red Hat Software Collections 3.5 和 Red Hat Developer Toolset 9.1](https://www.redhat.com/en/blog/red-hat-software-collections-35-and-red-hat-developer-toolset-91-now-generally-available) 现可用于 Red Hat Enterprise Linux 7。这对于开发者来说意味着什么。

[Red Hat Software Collections(RHS cl)](https://developers.redhat.com/products/softwarecollections/overview)是我们如何通过[Red Hat Enterprise Linux(RHEL)](https://developers.redhat.com/products/rhel/overview)7 分发各种运行时和语言的最新稳定版本，RHEL 6 中提供了一些组件。RHSCL 还包含 Red Hat Developer 工具集，这是我们为 [C/C++](https://developers.redhat.com/topics/c/) 和 Fortran 设计的工具集。[这些组件支持长达五年](https://access.redhat.com/support/policy/updates/rhscl-rhel7)，这也有助于您构建具有长生命周期的应用。

## 什么变了？

RHSCL 3.5 中更新的集合包括:

*   **Python 3.8** ，它引入了赋值表达式和多项优化，使 Python 3.8 运行速度快于之前的版本，并与之前的版本兼容，以简化升级策略。
*   **Ruby 2.7** ，它提供了大量的新特性，比如模式匹配、读取-求值-打印-循环(REPL)改进，以及针对碎片内存空间的压缩垃圾收集(GC)。
*   Perl 5.30 ，它为开发人员增加了一些新特性，比如有限的可变长度 lookbehinds、Unicode 12.1、更快的字符串插值以及其他性能改进。
*   **Apache httpd 2.4** (更新)，修复了一些 bug，并包含了支持 ACMEv2 的更新版本`mod_md`。
*   **Varnish 6** ，它将 Varnish 缓存更新到版本 6.0.6，这是最新的两年一次的新版本，具有许多错误修复和性能改进。
*   **Java Mission Control 7.1** ，将 JDK Mission Control 更新至 7.1.1 版本，并修复了众多 bug。它还增加了关键的增强功能，包括多种规则优化、基于标准小部件工具包(SWT)的新 JOverflow 视图、新的火焰图视图和使用高动态范围(HDR)直方图的新延迟可视化。
*   **HAProxy 1.8.24** ，提供多个 bug 和安全修复。

Red Hat Software Collections 3.5 的最后一个更新是 Red Hat Developer Toolset(DTS)9.1 版，这是我们为 C/C++和 Fortran 设计的一套工具。对于 DTS，我们更新了编译器、调试器和性能监控工具，以确保使用这些语言的软件开发人员获得最佳体验。DTS 9.1 的核心是 GCC 9.3，它带来了大量的改进，包括改进的诊断和可用性。我们在 DTS 9.1 中更新的[完整工具列表一如既往地在发行说明](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/9/html/9.1_release_notes/dts9.1_release#Features)中提供。

## 我如何得到这些好东西？

有了[红帽开发者订阅](https://developers.redhat.com/articles/getting-red-hat-developer-subscription-what-rhel-users-need-know/)，你就可以访问[红帽企业 Linux 7](https://developers.redhat.com/products/rhel/download) ，在那里你可以更新这些包。如果您已经在 subscription manager 中启用了 Red Hat 软件集合，请遵循以下针对特定软件集合或容器映像的说明。如果您还没有启用 RHSCLs，请[按照我们在线文档](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/3.5_release_notes/chap-installation)中的说明进行操作。

要安装特定的软件集合，请以 root 用户身份在命令行中键入以下内容:

```
$ yum install *software_collection…*
```

将`software_collection`替换为您要安装的软件集合的空格分隔列表。例如，要安装`php54`和`rh-mariadb100`，请以 root 用户身份键入:

```
$ yum install rh-php72 rh-mariadb102
```

这样做将为选定的软件集合安装主元软件包，并安装一组必需的软件包作为其依赖项。有关如何安装其他软件包(如附加模块)的信息，参见[第 2.2.2 节“安装可选软件包”](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/3.5_release_notes/chap-Installation#sect-Installation-Install-Optional)

当然，另一个选择是从这些包的[容器映像开始，这使得为 Red Hat Enterprise Linux 和](https://catalog.redhat.com/software/containers/explore) [Red Hat OpenShift](https://developers.redhat.com/topics/kubernetes/) 平台构建和部署使用这些组件的关键任务应用程序变得更加容易。

红帽软件集合 3.5 和[红帽开发者工具集 9.1](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/9/html/9.1_release_notes/index) 的完整[发行说明可在客户门户获得。](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/3.5_release_notes/index)

## 红帽企业版 Linux 8 怎么样？

软件集合适用于 Red Hat Enterprise Linux 7。Red Hat Enterprise Linux 8 通过[应用流](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)以不同的方式管理，你可以在 RHEL8 `appstream`存储库中找到更新的 RHEL 8 包。对于 Red Hat Enterprise Linux 8 应用流，这些包的更新可能不相同，因此请查看[应用流生命周期](https://access.redhat.com/support/policy/updates/rhel8-app-streams-life-cycle)页面。

*Last updated: June 25, 2020*