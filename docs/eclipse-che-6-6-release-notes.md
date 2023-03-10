# Eclipse Che 6.6 发行说明

> 原文：<https://developers.redhat.com/blog/2018/06/08/eclipse-che-6-6-release-notes>

【本文交叉转贴自 [Eclipse Che 博客。](https://che.eclipse.org/) ]

# Eclipse Che 6.6 发行说明

Eclipse Che 6.6 来了！自 Che 6.0 发布以来，社区增加了许多新功能:

*   **Kubernetes 支持:**在 Kubernetes 上运行 Che，并使用 Helm 部署它。
*   **热服务器更新:**零宕机升级 Che。
*   **C/C++支持:**增加了 ClangD 语言服务器。
*   **Camel LS 支持:**增加了 Apache Camel 语言服务器协议(LSP)支持。
*   Eclipse Java 开发工具(JDT)语言服务器(ls):为 Eclipse Che 添加了扩展的 LS 功能。
*   **更快的工作空间加载:**图像与新的用户界面并行绘制。

## 快速启动

Che 是一个云 IDE 和容器化的工作空间服务器。您可以使用以下链接开始使用 Che:

*   部署到 Kubernetes ( [单用户](https://www.eclipse.org/che/docs/kubernetes-single-user.html)或[多用户](https://www.eclipse.org/che/docs/kubernetes-multi-user.html))。
*   部署到红帽 OpenShift ( [单用户](https://www.eclipse.org/che/docs/openshift-single-user.html)或[多用户](https://www.eclipse.org/che/docs/openshift-multi-user.html))。
*   安装在 Docker 上([单用户](https://www.eclipse.org/che/docs/docker-single-user.html)或[多用户](https://www.eclipse.org/che/docs/docker-multi-user.html))。

## 立方支撑( [#8559](https://github.com/eclipse/che/pull/8559)

以前 Eclipse Che 主要针对 Docker。然而，随着 Kubernetes 的兴起，我们增加了 OpenShift 和原生 Kubernetes 作为主要部署平台。

自从 6.0.0 发布以来，我们已经做了许多更改，以确保 Che 可以与 Kubernetes 一起工作。这些变化与工作区的卷管理、路由、服务创建等相关。

我们最近还增加了在 Kubernetes 上部署 Che 的头盔图。Helm 是一个流行的应用程序模板系统，用于在 Kubernetes 上部署容器应用程序。舵图首次包含在 6.2.0 版本中，并且在 6.3.0 和 6.4.0 版本中支持有所改进。

使用 Helm 支持 TLS 路由和多用户 Che 部署的大部分工作是由 SAP 的 Guy Daich 贡献的。谢谢你，盖伊！

在[文档](http://www.eclipse.org/che/docs/kubernetes-multi-user.html)中的 Kubernetes 上了解更多关于 Che 的信息。

**突出问题**

请参见以下拉式请求(PRs):

*   Kubernetes-infra:路由，TLS(rebase)[# 9329](https://github.com/eclipse/che/pull/9329)
*   仅使用模板将 Che 部署到 OpenShift [#9190](https://github.com/eclipse/che/pull/9190)
*   Kubernetes 多用户头盔 [#8973](https://github.com/eclipse/che/pull/8973)
*   Kubernetes-infra:服务器路由策略和基本 TLS [种类/增强](https://github.com/eclipse/che/issues?q=is%3Apr+helm+label%3Akind%2Fenhancement)状态/代码-审查 [#8822](https://github.com/eclipse/che/pull/8822)
*   使用舵图 [#8715](https://github.com/eclipse/che/pull/8715) 将 Che 部署到 Kubernetes 的初始支持
*   增加了 Kubernetes 基础设施 [#8559](https://github.com/eclipse/che/pull/8559)

## 热门服务器更新( [#8547](https://github.com/eclipse/che/issues/8547) )

在最近的版本中，我们稳步提高了升级 Che 服务器的能力，而无需停止或重启活动工作区。在 Che 6.6.0 中，现在可以在不停机的情况下升级活动工作区的 Che 服务器，并且只有很短的一段时间不能启动新的工作区。这是我们的企业用户的要求，但它有助于各种规模的团队。

您可以在[文档](http://www.eclipse.org/che/docs/openshift-admin-guide.html#rolling-update)中了解更多信息。

**突出问题**

请参见以下 PRs:

*   执行 OpenShift 工作区启动中断 [#5918](https://github.com/eclipse/che/issues/5918)
*   实施 OpenShift 基础架构的恢复 [#5919](https://github.com/eclipse/che/issues/5919)
*   如果工作区由另一个 Che 服务器实例 [#9502](https://github.com/eclipse/che/issues/9502) 启动，则不会启动服务器检查器
*   记录滚动热更新的程序 [#9630](https://github.com/eclipse/che/issues/9630)
*   使服务终止功能适应工作区恢复 [#9317](https://github.com/eclipse/che/issues/9317)
*   恢复 k8s/os 工作区时，服务器检查程序无法正常工作 [#9453](https://github.com/eclipse/che/issues/9453)
*   添加使用分布式缓存在 workspace 运行时中存储工作区状态的功能 [#9206](https://github.com/eclipse/che/issues/9206)
*   不要使用数据量在 OpenShift/Kubernetes 上存储代理 [#9040](https://github.com/eclipse/che/issues/9040)

## C/C++支持 ClangD LS ( [#7516](https://github.com/eclipse/che/pull/7516) )

[Clang](https://clang.llvm.org/) 为 LLVM 编译器套件提供了 C 和 C++语言前端， [Clangd](http://llvm.org/viewvc/llvm-project/clang-tools-extra/trunk/clangd/) LS 在 Eclipse Che 和其他支持 LSP 的 ide 中实现了对 C 语言的改进支持。非常感谢 Silexica 的 [Hanno Kolvenbach](https://github.com/hkolvenbach) 对该功能的贡献。

 ![](img/001460148cc682311bfbc8a537252c15.png)Code Completion with ClangD ![](img/70be5c724f482df1bd47a0d71e7daabb.png)Go to Definition with ClangD

## Apache Camel LSP 支持( [#8648](https://github.com/eclipse/che/pull/8648) )

![](img/a6f9a10df4af75eed4c7c2ee99c9e41e.png)

Camel-language-server 是一个提供 Camel DSL 智能的服务器实现。服务器遵循[语言服务器协议](https://github.com/Microsoft/language-server-protocol)，并且已经集成到 Eclipse Che 中。服务器使用[阿帕奇骆驼](http://camel.apache.org/)。

**相关 PRs**

请参见以下 PRs:

*   介绍 Apache Camel LSP 支持 [#8648](https://github.com/eclipse/che/pull/8648)
*   [533196]修复骆驼 LSP 人工制品以下载 [#9324](https://github.com/eclipse/che/pull/9324)

## Eclipse JDT LS ( [#6157](https://github.com/eclipse/che/issues/6157) )

Eclipse JDT LS 结合了 Eclipse JDT 的能力(支持 Eclipse 桌面 IDE)和[语言服务器协议](https://github.com/Microsoft/language-server-protocol)。JDT LS 可以与任何支持该协议的编辑器一起使用，当然也包括 Che。该服务器基于:

*   [Eclipse LSP4J](https://github.com/eclipse/lsp4j) ，LSP 的 Java 绑定
*   [Eclipse JDT](http://www.eclipse.org/jdt/) ，它在 Eclipse IDE 中提供 Java 支持(代码完成、引用、诊断等等)
*   [M2Eclipse](http://www.eclipse.org/m2e/) ，提供 Maven 支持
*   [建造船只](https://github.com/eclipse/buildship)，提供梯度支持

Eclipse Che 将很快将其 Java 支持转换为使用 JDT LS。为了支持这种转变，我们一直在努力支持扩展的 LS 功能。Java 是 Che 用户使用最多的语言之一，由于 JDT LS，我们将带来更多的功能。一旦切换完成，您可以期待更多的 Java 版本得到支持，以及 Maven 和 Gradle 支持！

**突出问题**

请参见以下 PRs:

*   jdt.ls 未启动时命令不起作用 [#8673](https://github.com/eclipse/che/issues/8673)
*   允许自定义通知[eclipse/eclipse . JDT . ls # 522](https://github.com/eclipse/eclipse.jdt.ls/issues/522)
*   支持格式化程序选项[eclipse/eclipse . JDT . ls # 623](https://github.com/eclipse/eclipse.jdt.ls/issues/623)
*   使用 [#8880](https://github.com/eclipse/che/issues/8880) 为 jdt.ls 更新 Che 文档
*   实现`textDocument/onTypeFormatting`支持[eclipse/eclipse . JDT . ls # 569](https://github.com/eclipse/eclipse.jdt.ls/issues/569)
*   仅在客户端支持时返回 JDT://URL[eclipse/eclipse . JDT . ls # 649](https://github.com/eclipse/eclipse.jdt.ls/issues/649)
*   将 lsp4j 更新为 0 . 4 . 0[eclipse/eclipse . JDT . ls # 653](https://github.com/eclipse/eclipse.jdt.ls/issues/653)
*   自动完成应该在 Markdown[eclipse/eclipse . JDT . ls # 654](https://github.com/eclipse/eclipse.jdt.ls/issues/654)中返回 Javadoc
*   启用注释处理[eclipse/eclipse . JDT . ls # 128](https://github.com/eclipse/eclipse.jdt.ls/issues/128)

## 更快的工作空间加载( [#8748](https://github.com/eclipse/che/pull/8748) )

在版本 6.2.0 中，我们引入了 Che 通过 SPI 并行提取多个图像的功能。这样，当您在基于多容器的应用程序上工作时，您的工作区的容器映像可以更快地实例化。

**突出问题**

见以下 PR:

*   Che 应该并行提取图像( [#7102](https://github.com/eclipse/che/issues/7102) )

## 即将推出

您可以在[项目路线图页面](https://github.com/eclipse/che/wiki/Roadmap)上跟踪我们对 Eclipse Che 的未来计划。在即将发布的版本中，您可以期待对平台可扩展性的进一步改进，包括 Eclipse Che 插件框架、对调试适配器协议的支持以改进 IDE 中的调试功能、将更多的云原生技术集成到工作区管理中，以及可伸缩性和可靠性工作以使 Eclipse Che 更适合大型企业用户。

机构群体正在这些不同方面努力工作，我们将在接下来的几周内更广泛地讨论这一点。如果您有兴趣了解更多信息，并希望最终参与进来，请不要忘记参加[双周社区电话](https://github.com/eclipse/che/wiki/Che-Dev-Meetings)。

## 入门指南

[在 Kubernetes、OpenShift 或 Docker 上开始](https://www.eclipse.org/che/docs/infra-support.html)。

在我们的[文档](https://www.eclipse.org/che/docs/infra-support.html)中了解更多信息，并立即开始使用共享的 Che 服务器或本地实例。

Eclipse Che 项目一直在寻找用户反馈和新的贡献者！了解如何参与进来，帮助 Che 做得更好。

*Last updated: September 3, 2019*