# 宣布更新的 Red Hat Developer Studio 和容器开发工具包

> 原文：<https://developers.redhat.com/blog/2018/07/18/announcing-updated-cdk-devstudio>

我非常高兴地宣布[红帽容器开发工具包(CDK) 3.5](https://developers.redhat.com/products/cdk/overview/) 和[红帽开发者工作室 12](https://developers.redhat.com/products/devstudio/overview/) 的发布。无论您是开发传统的还是基于云的应用和微服务，您都可以在 Windows、macOS 或 Red Hat Enterprise Linux 笔记本电脑上运行这些工具来简化开发:

*   Red Hat Container Development Kit 提供了一个预构建的容器开发环境，帮助您使用 Red Hat OpenShift 和 Kubernetes 快速开发基于容器的应用程序。
*   Red Hat Developer Studio(以前称为 JBoss Developer Studio)为您的整个开发生命周期提供了一个桌面 IDE。它包括广泛的工具功能和对多种编程模型和框架的支持。Developer Studio 为使用 Red Hat 产品和技术提供了广泛的支持，包括中间件、业务自动化和集成，特别是 Camel 和 Red Hat Fuse。Developer Studio 基于 Eclipse 4.8(光子)。

许多 Red Hat Enterprise Linux (RHEL)开发工具已经更新。其中包括 Rust 1.26.1，Go 1.10.2，Cargo 1.26，Eclipse 4.8(光子)。

我们的目标是为开发人员提高我们工具的可用性，同时增加对 Red Hat 平台和技术的用户最重要的新功能。

新功能概述:

## 发布亮点

[**CDK 3.5**](https://developers.redhat.com/products/cdk/overview/) 已更新，包括以下更新:

*   包含了最新的 Red Hat OpenShift 容器平台，当新发布的版本可用时，可以使用`minishift start --ocp-tag <version>`命令将其更新到新版本。
*   你可以在现有的 RHEL 7 远程机器上运行 CDK。
*   SSHFS 是在虚拟机内部运行的容器环境中共享主机上的文件夹的默认方法。
*   包含一个[本地 DNS 服务器](https://docs.openshift.org/latest/minishift/using/experimental-features.html#local-dns-server)以减少对 nip.io 的依赖，并更充分地支持离线工作。
*   CDK 3.5 基于 Minishift 1.21.0。

[**红帽开发者工作室**](https://developers.redhat.com/products/devstudio/overview/) **12** 已更新至最新[日蚀光子](https://www.eclipse.org/eclipse/news/4.8/)发布。它支持 CDK 3.5 服务器适配器和 Red Hat OpenShift.io 登录提供程序。这个版本支持用 Java 9 和 10 开发应用程序。一如既往，它可以在 Linux、Windows 和 macOS 上运行。

此外，Red Hat Developer Studio Central 插件已更新，包括:

*   红帽保险丝工具 11
*   红帽应用程序迁移工具包 4.1
*   CDK 3.5 服务器适配器
*   野花 13 支持
*   open shift spring boot-增强支撑
*   [连接器](https://issues.jboss.org/browse/JBIDE-26065)用于[节点夹子](http://www.nodeclipse.org/updates/)支持 Node.js

参见 Jeff Maury 在 Red Hat Developer blog 上发布 Red Hat Developer Studio 12 和 JBoss Tools 4.6T3 的文章[。插件信息可以在](https://developers.redhat.com/blog/2018/07/18/announcing-devstudio-12-jboss-tools-46/) [JBoss 工具文档](https://tools.jboss.org/documentation/whatsnew/jbosstools/4.6.0.Final.html)中找到。日食光子信息可在 [JAXenter](https://jaxenter.com/eclipse-photon-is-out-146242.html) 获得。

## Red Hat Enterprise Linux 工具的更新

Go 工具集中的 Go 语言包已从版本 1.8.7 更新到版本 1.10.2。此版本修复了一些最近发现的安全问题。值得注意的变化包括:

*   构建和测试运行的结果现在被缓存，从而提高了这些操作的性能。
*   增加了包中函数的并发编译。
*   Go 编程语言中添加了类型别名。要创建类型别名，请使用以下格式:`type B = A`。
*   Go 标准库中添加了用于位计数和无符号整数类型操作的 math/bits 包。
*   添加了并发访问的`sync.Map`类型。
*   添加了`testing.B.helper()`和`testing.T.helper()`函数，以启用测试助手函数的标记。
*   由`time`包中的`Time`类型跟踪的时间现在透明地总是单调的。

Rust 已从 1.25.0 版更新到 1.26.2 版。值得注意的变化包括:

*   添加了用`impl Trait`构造描述类型而不给出类型名的功能。在实际类型未知的情况下，例如使用闭包，或者在类型的实现应该保持私有的情况下，此构造对于返回未命名类型很有用。在必须在语句中的多个位置提供类型的情况下，仍然需要类型参数。
*   对自动模式引用的支持已添加到`match`、`let`和其他语句中。当使用模式匹配被引用对象的内部部分时，编译器现在可以自动解引用对象并引用内部部分。
*   以前，Rust 程序的`main()`函数只能返回`()`单元类型。Rust 已经被扩展，允许从`main()`返回一个`Result`类型的值，比如`Result<(), E>`，并处理以这种方式提供的错误值。
*   添加了包含范围。要指定包括最后指定值的范围，使用`a..=b`。
*   对切片模式的支持已被添加到`match`语句中。
*   添加了 128 位整数类型 i128 和 u128。

其他 Red Hat Enterprise Linux 工具更新包括:

*   `cargo`工具已经从版本 0.26.0 更新到版本 1.26.0。
*   `cargo-vendor`工具已经从版本 0.1.13 更新到版本 0.1.15。
*   rust Language Server(RLS)0 . 126 . 0 版已添加到 Red Hat 开发人员工具中。这个工具支持 Rust 与集成开发环境的集成。RLS 由`rust-toolset-7-rls-preview`包提供。

关于这些工具和其他工具的更多信息可以在我们的文档中找到。

一如既往，我鼓励您下载并试用这些新工具，告诉您的朋友，并向我们提供您的任何反馈。非常感谢扩展的 Red Hat 开发工具和程序团队，他们再一次在我们 12 周的周期内按时发布了版本。

*Last updated: March 19, 2020*