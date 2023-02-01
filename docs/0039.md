# 故障排除-编译器 Hello Worlds

> 原文：<https://developers.redhat.com/HW/compiler-troubleshooting>

### **使用软件集合包**

这些工具被打包成软件集合，旨在允许同时安装多个版本的软件。为了实现这一点，需要使用`scl enable`命令将所需的包添加到您的运行时环境中。当`scl enable`运行时，它修改环境变量，然后运行指定的命令。环境变化只影响由`scl`运行的命令和从该命令运行的任何进程。本教程中的步骤运行命令`bash`来启动一个新的交互式 shell，以便在更新的环境中工作。这些变化不是永久性的。输入`exit`会用原来的环境回到原来的外壳。每次登录或开始新的终端会话时，都需要再次运行`scl enable`。

虽然可以更改系统配置文件，使 RHSCL 软件包成为系统全局环境的一部分，但不建议这样做。这样做可能会导致与其他应用程序的冲突和意外问题，因为路径中首先出现的是 RHSCL 版本，从而掩盖了包的系统版本。

### **了解更多关于 Red Hat 软件集合的信息**

[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/)提供动态语言、开源数据库和 web 开发工具的最新稳定版本，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。有关更多信息:

*   [Red Hat Software Collections 打包指南](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/packaging_guide/)—Red Hat Software Collections 打包指南解释了软件集合的概念，记录了 scl 实用程序，并提供了如何创建自定义软件集合或扩展现有软件集合的详细说明。

*   [Red Hat Software Collections 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/3.0_release_notes/)—Red Hat Software Collections 发行说明记录了内容集发布时已知的问题、可能的问题以及其他可用的重要信息。它们还包含关于安装、重建和迁移的有用信息。

*   [如何使用红帽软件集合(RHSCL)、红帽开发者订阅，或者 Clang/LLVM、Go、Rust 编译器](https://access.redhat.com/solutions/472793) —这篇文章列出了哪些红帽企业 Linux 订阅包括访问红帽软件集合、开发者工具集(带 GCC)、Clang/LLVM、Go 和 Rust。

您可以通过运行以下命令来查看 RHSCL 中可用的软件包列表:

`$ yum --disablerepo="*" --enablerepo="rhel-server-rhscl-7-rpms" list available`

#### **使用 Red Hat Enterprise Linux 开发**

[Red Hat Enterprise Linux 7 开发人员指南](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Developer_Guide/)—《Red Hat Enterprise Linux 7 开发人员指南》提供了应用程序开发工具以及在 Red Hat Enterprise Linux 7 中使用 Git 等源代码管理工具的介绍。

#### 成为红帽开发者，加入红帽开发者计划

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

*Last updated: November 19, 2020*