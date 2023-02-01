# 红帽企业版 Linux 8.1 测试版带来了新的开发工具

> 原文：<https://developers.redhat.com/blog/2019/07/24/red-hat-enterprise-linux-8-1-beta-brings-new-development-tools>

五月份，我们宣布红帽企业版 Linux(RHEL)8(T1)全面上市，这是一款智能操作系统，我们认为它是开发人员迄今为止最好的 RHEL。

Red Hat Enterprise Linux 8 的工作仍在继续，我们很高兴地[宣布](http://redhat.com/en/blog/red-hat-enterprise-linux-81-beta-now-available)RHEL 8.1 的测试版已经推出。，可提高开发人员的工作效率，改善可管理性，并增加新的安全性增强功能。此版本还包括更新的驱动程序，为支持的硬件平台提供新功能和错误修复。

## 新的应用程序流

RHEL 8.1 测试版提供了几个新的[应用流](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)，带有新的和/或更新的开发者工具、应用框架和语言。其中包括:

*   GCC 工具集 9
*   Node.js 12
*   Ruby 2.6
*   PHP 7.3
*   Nginx 1.16
*   Go 1.12 更新
*   Clang/LLVM 8 的更新

所有这些包都可以使用 yum 获得，并且包含在所有 Red Hat Enterprise Linux 订阅中。

## 图像生成器

Red Hat Enterprise Linux 8 引入了 Image Builder，这是一个允许您以各种格式创建自定义系统映像的组件。在 RHEL 8.1 测试版中，Image Builder 得到了扩展，以支持更多用于添加用户和 SSH 密钥的配置选项。还增加了新的图片格式，支持谷歌云平台、阿里云等云平台。通过这些新增功能，RHEL 8.1 测试版现在支持所有主要的云基础架构平台，包括 AWS、Microsoft Azure、OpenStack 和 VMware。

## 更多好东西在 RHEL 8.1 测试版

### 提高可管理性

Red Hat Enterprise Linux web 控制台现在在配置防火墙规则和系统服务时支持更精细的粒度，包括:

*   更好的防火墙区域配置。
*   基于服务的日志过滤。
*   基于元数据(如服务名称和状态)的服务过滤。

此外，对于在 RHEL 8.1 测试版上运行的虚拟机(VM)，您现在可以使用 web 控制台来导入现有的 QCOW 映像、管理不同类型的存储池、修改 autostart 配置和内存分配，以及暂停和恢复现有的 VM。

### 增强的安全性

安全性仍然是 Red Hat Enterprise Linux 的一个重要关注点，RHEL 8.1 测试版增加了以容器为中心的 se Linux 配置文件。借助这一新功能，您可以创建更加定制的安全策略，以便更好地控制容器访问主机系统资源(如存储、计算和网络)的方式。

这种方法使客户能够更有效地加强其容器部署，防止安全违规，从而更容易实现和维护法规遵从性。使用新的应用程序白名单功能，管理员还可以更有选择性地选择允许在系统上启动哪些应用程序。此功能降低了运行未知或不可信应用程序的潜在风险。此外，RHEL 8.1 测试版将用于我们寻求平台的额外 FIPS-140 和通用标准认证。

## 资源

*   [了解更多关于 RHEL 8 的信息](https://developers.redhat.com/rhel8/)
*   [rhel-8.1-beta-1-x86 _ 64-DVD . iso](https://developers.redhat.com/download-manager/file/rhel-8.1-beta-1-x86_64-dvd.iso)
*   [rhel-8.1-beta-1-x86 _ 64-boot . iso](https://developers.redhat.com/download-manager/file/rhel-8.1-beta-1-x86_64-boot.iso)
*   [rhel-8.1-beta-1-aach 64-DVD . iso](https://developers.redhat.com/download-manager/file/rhel-8.1-beta-1-aarch64-dvd.iso)
*   [rhel-8.1-beta-1-aarch 64-boot . iso](https://developers.redhat.com/download-manager/file/rhel-8.1-beta-1-aarch64-boot.iso)

*Last updated: August 21, 2019*