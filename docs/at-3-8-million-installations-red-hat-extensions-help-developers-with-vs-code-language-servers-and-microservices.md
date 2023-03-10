# 在 380 万次安装中，Red Hat extensions 帮助开发人员使用 VS 代码、语言服务器和微服务

> 原文：<https://developers.redhat.com/blog/2019/04/17/at-3-8-million-installations-red-hat-extensions-help-developers-with-vs-code-language-servers-and-microservices>

在从事 VS 代码扩展三年之后，我的团队[庆祝了 380 万次安装](https://www.redhat.com/en/blog/red-hat-extensions-microsoft-visual-studio-code-receive-38-million-installs)和超过 2000 万次下载——这两个指标表明我们正在提供被其他开发者接受的有价值的 VS 代码扩展。我们还庆祝我们与[语言服务器协议](https://microsoft.github.io/language-server-protocol/) (LSPs)的合作帮助了不同规模的开源社区实现了广泛的 ide(集成开发环境)和编辑器选择，从而使这些社区变得更加强大。那么，我们是怎么走到这一步的？

早在 2016 年初，我的团队和几个主要致力于实现 IDE 的红帽伙伴一起，寻找新的架构，让不同的社区，如编程语言，运行时可以轻松地与 IDE 集成，而无需深入了解 IDE 本身。随着我们实验的继续，微软的开发团队开源了 [Visual Studio 代码](https://code.visualstudio.com/) (VS 代码)，并引入了[语言服务器协议](https://microsoft.github.io/language-server-protocol/) (LSP)。

我们在引入 LSP 和 VS 代码后的第一个实验是 [Eclipse JDT 语言服务器](https://github.com/eclipse/eclipse.jdt.ls)和附带的 VS 代码扩展，Java 的[语言支持](https://marketplace.visualstudio.com/itemdetails?itemName=redhat.java)。我们从最初的工作中得出的一个结论是，LSP 对于不同的开源社区来说是一个很好的使能器，否则它在不同的 ide 和编辑器上不会有一流的表现。我们决定致力于实现对更多 IDE 的 LSP 支持，包括 Eclipse IDE 和 Eclipse Che。今天，你可以使用越来越多的[语言服务器](https://microsoft.github.io/language-server-protocol/implementors/servers/)和许多你喜欢的[编辑器](https://microsoft.github.io/language-server-protocol/implementors/tools/)。

在开发 JDT 语言服务器和 VS 代码扩展时，我们发现 VS 代码提供了一种非常适合开发微服务应用程序的开发体验。这促使我们决定将我们一直在做的一些 LSP 实现引入 VS 代码。最初的几个是 YAML 扩展和 T2 依赖分析扩展。虽然 Dependency Analytics 本身不是一种编程语言，但它确实包含了一个语言服务器，可以通过编辑器内的代码提供关于依赖关系的信息。后来，我们还发布了 [XML 扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml)。

随着 VS 代码和我们的扩展变得越来越流行，我们收到了集成到各种 Red Hat 运行时的请求，比如 OpenShift 或 Wildfly。作为回应，我们为我们的运行时发布了两个连接器扩展:

*   [OpenShift 连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)提供与 OpenShift 平台的集成；它在幕后使用 [odo](https://github.com/openshift/odo) 为内部(开发人员的)循环提供云原生应用组合、部署和调试。
*   [服务器连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-server-connector)提供中间件集成，比如 Wildfly 和 JBoss EAP。它使用[运行时服务器协议](https://github.com/redhat-developer/rsp-server) (RSP)，这是一个本质上类似于 LSP 的协议，旨在帮助中间件社区支持更多的 ide，以实现跨不同中间件风格的功能。

我们也收到了很多关于使用哪些扩展的问题，所以我们策划了不同的[扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-java-pack) [包](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-extension-pack)，我们认为它们是有用的并且质量很好。试试 [Visual Studio 代码扩展](https://developers.redhat.com/products/vscode-extensions/overview)，让我们知道你的想法。

*Last updated: May 1, 2019*