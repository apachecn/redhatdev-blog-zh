# 红帽发布新的开发工具

> 原文：<https://developers.redhat.com/blog/2017/08/15/red-hat-releases-new-development-tools>

我非常高兴地宣布我们的 Red Hat 开发工具的最新版本，可在多个平台上使用。此次发布的主题是扩展可用性、产品集成、扩展对开发套件中中间件产品的支持，以及全新添加的 Kompose 和用于 Red Hat Enterprise Linux 的 [DevTools 频道](https://access.redhat.com/documentation/en-us/red_hat_jboss_developer_studio/11.0/html/installation_guide/rpm#enabling_the_red_hat_developer_tools_repositories) 。

这些工具集合已被组装成一个易于使用的安装程序，帮助软件开发人员快速、轻松地构建开发环境，通过在桌面上安装 OpenShift 来创建容器化的企业 Java 应用。开发者工具安装程序将在 macOS、Windows 和 Red Hat Enterprise Linux 上自动下载、安装和配置选定的工具。开发套件还简化了 EAP、Fuse 和 [Kompose](https://developers.redhat.com/blog/2017/08/02/getting-started-with-kompose/) 的安装和配置。一如既往，它可以从 developers.redhat.com/downloads 的[](https://developers.redhat.com/downloads/)免费获得。

今天，红帽发布了以下新版本:

*   [红帽开发套件 2](https://developers.redhat.com/products/devsuite/overview/)
*   [红帽 JBoss 开发者工作室 11](https://developers.redhat.com/products/devstudio/overview/)
*   [红帽容器开发套件 3.1 (CDK)](https://developers.redhat.com/products/cdk/overview/)

### **开发套件 2 (DevSuite)的主要新特性:**

*   DevStudio 11.0
*   CDK 3.1
*   OpenJDK 8 u141
*   Kompose 1.0 技术预览版——现在是一个 Kubernetes 社区项目(yum install 来自新的 DevTools 频道——在这篇 [博客文章](https://developers.redhat.com/blog/2017/08/02/getting-started-with-kompose/) 中了解更多关于 Kompose 的信息)。)
*   对 Red Hat Enterprise Linux 7 的完全 RPM 支持(从新的 DevTools 渠道进行 yum 安装)
*   保险丝工具 10.0.0 GA
*   在 JBoss EAP 6.4.0 上融合集成平台 6 . 3 . 0
*   在 Apache Karaf 6.3.0 上融合集成平台 6.3.0
*   JBoss EAP 7.0.0.GA 可选安装

### **开发者工作室 11 (DevStudio)的主要新特性:**

*   基于 Eclipse IDE 4.7 (Oxygen) (yum 从新的 DevTools 通道安装)
*   适用于 Red Hat Enterprise Linux 7 的 rpm(从新的 DevTools 渠道进行 yum 安装)
*   CDK 3.1 服务器适配器
*   包含保险丝工具(通过 DevSuite 或 Central 可选安装)

### **Red Hat 容器开发套件 3.1 (CDK)**

*   基于 Minishift 社区发布版本 1.3.1
*   【OpenShift 图像的本地容器图像缓存
*   附加功能的改进功能
*   RHEL 开发套件 7 安装程序的 CDK RPM
*   要求:本机虚拟机管理程序(Hyper-V/xyve/KVM)或 VirtualBox

一如既往，我鼓励你下载并试用这些新工具，告诉你的朋友，并向我们提供任何反馈。非常感谢扩展的 Red Hat 开发工具和程序团队，他们再一次在我们 12 周的周期内按时发布了版本。

*Last updated: March 19, 2020*