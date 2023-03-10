# 获得红帽开发者订阅:RHEL 用户需要知道什么

> 原文：<https://developers.redhat.com/articles/getting-red-hat-developer-subscription-what-rhel-users-need-know>

对于开发人员来说，获得 Red Hat Enterprise Linux 比以往任何时候都更容易，这要归功于免费的 Red Hat Developer Subscription。

## 获取软件

1.  加入红帽开发者计划，并在[developers.redhat.com](/register)注册订阅。每个运行 Red Hat Enterprise Linux 的系统都需要订阅。

2.  [下载](https://developers.redhat.com/products/rhel/download/)红帽企业版 Linux 服务器

3.  [使用我们的指南安装](/products/rhel/hello-world) Red Hat Enterprise Linux Server。对于 Microsoft Windows、Apple Mac OS 或其他 Linux 用户，我们的指南涵盖了使用 VirtualBox、Hyper-V 创建虚拟机(VM)或在物理系统上安装。

4.  安装后，系统会提示您向 Red Hat 客户门户注册您的系统。这将把您的系统附加到您的订阅，使系统能够从 Red Hat 下载更新和附加软件。

## 开发软件和软件仓库

对于开发者来说，红帽企业 Linux 包含了大量的软件，如编译器、动态语言、开发工具、数据库、web 服务器和其他中间件。可用软件被分组到软件存储库中，这些存储库用于根据类型、源代码或支持生命周期来隔离软件包。安装后，只启用基本的 Red Hat Enterprise Linux 软件存储库。通过启用附加存储库，可以安装附加软件。其他存储库中的软件可能不受 Red Hat 支持，或者可能具有不同的生命周期。

## 更新的开发工具:红帽软件系列

red Hat Software Collections(RHS cl)是 RHEL 7 的一部分，它提供了一套开发工具，如编译器、动态语言和数据库服务器，这些工具每年都会更新。这些工具在 RHSCL 软件存储库中以软件包的形式提供，在启用 RHSCL 软件存储库后，它们可以与 Red Hat Enterprise Linux 附带的工具一起安装。

由于 Red Hat Enterprise Linux 包含的基础包支持 10 年，因此在 10 年的生命周期内，将根据需要提供更新包来解决关键的错误修复和安全更新。RHSCL 包仅支持两到三年，具体取决于包。开发人员可以自由选择哪个包适合他们的应用。

## Red Hat Enterprise Linux 版本和生命周期

Red Hat Enterprise Linux 的发行版有一个版本号，比如 7.2，它既表示主发行版号，在本例中为 7，又表示更新(或次版本)号，在本例中为 2。

次要版本(0.1、0.2、0.3 等。)将在整个 10 年期间完成。升级到下一个次要版本很简单，过程基本上与应用其他安全和 bugfix 更新相同。在应用程序二进制接口(ABI)级别，在一个主要版本的整个生命周期中，都非常注意确保兼容性。因此，Red Hat Enterprise Linux 包含的库、编译器、运行时和 Linux 内核的版本在一个主要发行版的 10 年生命周期中变化很小。必要时，对于 Red Hat 支持的包，重要的 bug 和/或安全修复将被反向移植并作为更新提供。

更多信息请参见[红帽企业版 Linux 生命周期](https://access.redhat.com/support/policy/updates/errata)和[红帽企业版 Linux 发布日期](https://access.redhat.com/articles/3078)。

## Hello World 快速入门

使用我们的[Red Hat Enterprise Linux 快速入门](/products/rhel/docs-and-apis)和[Red Hat Software Collections](/products/softwarecollections/hello-world)在几分钟内安装软件并构建一个“Hello World”应用程序。指南适用于 C++、Java、Node.js、Perl、PHP、Python 和 Ruby。

## 关于免费开发者订阅的问题

如果您对通过提供的免费订阅有任何疑问，请参见[常见问题:面向 Red Hat Enterprise Linux 的免费 Red Hat 开发人员订阅](/articles/no-cost-rhel-faq)。

*Last updated: November 30, 2021*