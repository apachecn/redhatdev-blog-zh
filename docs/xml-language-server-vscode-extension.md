# XML 语言服务器和 VSCode 扩展

> 原文：<https://developers.redhat.com/blog/2018/12/04/xml-language-server-vscode-extension>

作为 Red Hat 的一名实习生，我的第一个也是正在进行的项目是与[安吉洛·泽尔](https://twitter.com/angelozerr) 和[弗雷德·布里肯](https://twitter.com/fbricon) 一起开发 XML 的[语言服务器协议](https://developers.redhat.com/blog/2016/06/27/a-common-interface-for-building-developer-tools/) (LSP) 的实现。通过 XML 语言服务器，像 VSCode 和 Eclipse 这样的开发工具接收 XML 语法高亮显示和检查、代码完成、文档折叠等。目前，我们似乎拥有功能最丰富的 XML 语言服务器实现，包括我们基于模式的支持，这是我们最引以为豪的一个基本 XML 功能。结合起来，所有这些特性使得开发人员可以更容易地在他们最喜欢的编辑器或 IDE 中处理任何类型的涉及 XML 的项目。

[观看 XML VS 代码扩展的演示](https://www.youtube.com/watch?v=2T4g9R56Cc0)

https://www.youtube.com/watch?v=2T4g9R56Cc0

XML 语言服务器目前在 VSCode(通过 Red Hat 的 [XML 扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml))、 [Eclipse LSP4E](https://github.com/angelozerr/lsp4e-xml) 中可用，很快在 [Eclipse Che、](https://www.eclipse.org/che/)中可用，应该在 2019 年初可用。

## 支持的功能:

*   语法错误报告
*   通用代码完成
*   自动关闭标签
*   自动节点缩进
*   符号高亮显示
*   文件折叠
*   文档链接
*   文件符号和轮廓
*   重命名支持
*   文档格式化
*   DTD 验证
*   XSD 验证
*   XSD 基地悬停
*   基于 XSD 的代码完成
*   XSL 支持
*   XML 目录
*   文件关联
*   代码动作
*   模式缓存

功能的数量只会继续增长， 我们最近发布的第二个版本有了很大的改进，例如本地模式缓存可以加快加载速度。一些即将到来的增加将包括对 DTD 和 XSD 支持的主要改进和修复，这将补充整个 XML 套件。

这个语言服务器由 Angelo 发起，是我们用 Java 开发的微软容错 HTML 语言服务器的一个端口。因为 XML 是 HTML 的一个子集，所以我们能够在 HTML 语言服务器的底层结构上进行构建，这反过来使我们能够快速地开始 XML 特定功能的工作。

如果你想为语言服务器做贡献，可以在这里找到这个库。

与 Red Hat 和开源社区一起参与这个项目是一次很棒的经历，尤其是在一个如此有潜力的工具上。 随着轻量级和基于云的 IDE 如 [VSCode](https://code.visualstudio.com/) 或 [Eclipse Che](https://www.eclipse.org/che/) 的发展，语言服务器将是这些编辑器成功的关键。令人兴奋的是，即使作为一名实习生，我的工作也将成为开发者空间中一场非常重要的运动的一部分。

我希望你尝试一下 VSCode 中的 XML 语言服务器，或者你最喜欢的支持语言服务器协议的编辑器，并在这里随意留下反馈或报告任何 bug[。](https://github.com/angelozerr/lsp4xml/issues)

[![XML Language support for VS Code by Red Hat](img/c4cbc72ac154b090d2f319de9945ba56.png)](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml)

## 额外资源

*   [开发者工具的公共接口—](https://developers.redhat.com/blog/2016/06/27/a-common-interface-for-building-developer-tools/)Gorkem er can 的语言服务器协议介绍
*   由 Red Hat 提供的对 Java for Visual Studio Cod 的语言支持—这个 Java LSP 实现是 Visual Studio 代码市场中最流行的扩展之一。至今已下载 1000 万次。
*   [宣布 Visual Studio 代码的 Red Hat OpenShift 扩展](https://developers.redhat.com/blog/2018/11/28/announcing-red-hat-openshift-extension-for-visual-studio-code-public-preview/)—从您的 IDE 中控制您的 OpenShift/Kubernetes 开发环境。
*   [Red Hat 对 Visual Studio Marketplace 的扩展](https://marketplace.visualstudio.com/publishers/redhat)包括
    *   [Java 语言支持](https://marketplace.visualstudio.com/items?itemName=redhat.java) —Java 林挺、智能感知、格式化、重构、Maven/Gradle 支持等等
    *   [YAML 支持](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)—内置 Kubernetes 和 Kedge 语法支持的 YAML 支持
    *   [OpenShift 连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)—使用 Visual Studio 代码与 Red Hat OpenShift 集群交互的简化体验
    *   [依赖性分析](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics)—洞察您的应用依赖性:安全性、许可证兼容性和基于人工智能的指导，为您的应用选择合适的依赖性。

*Last updated: December 11, 2018*