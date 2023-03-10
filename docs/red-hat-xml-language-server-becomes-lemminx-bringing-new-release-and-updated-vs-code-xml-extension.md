# Red Hat XML 语言服务器成为 LemMinX，带来了新的版本和更新的 VS 代码 XML 扩展

> 原文：<https://developers.redhat.com/blog/2020/03/27/red-hat-xml-language-server-becomes-lemminx-bringing-new-release-and-updated-vs-code-xml-extension>

红帽的 XML 语言服务器已经开始了一个新的时代，它被迁移到 Eclipse 基金会，并有了一个新的项目名称: [Eclipse LemMinX](https://projects.eclipse.org/projects/technology.lemminx) (参考了 [Lemmings 视频游戏](https://en.wikipedia.org/wiki/Lemmings_(video_game)))。Eclipse LemMinX 项目可以说是目前功能最丰富的 XML 语言服务器。它的迁移为未来的开发和利用打开了更多的大门。此外，在其迁移后不久，Eclipse LemMinX 项目和 Red Hat 也发布了更新:Eclipse LemMinX 版本 0.11.1 和 [Red Hat VS Code XML 扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml)。

## Eclipse LemMinX 版本 0.11.1

Eclipse LemMinX 版本 0.11.1 主要关注于这里的变更日志中概述的[错误修复。在一些历史上，Eclipse LemMinX 是由](https://github.com/eclipse/lemminx/blob/master/CHANGELOG.md) [Angelo ZERR](https://twitter.com/angelozerr) 在 2018 年年中创建的一个开源项目。Angelo 的 XML 语言服务器实现在特性和代码基础设施方面遥遥领先。随着 Red Hat 对 XML 语言服务器的兴趣不断增长，Red Hat 与 Angelo(他后来正式加入 Red Hat，担任高级软件工程师)联手创建了功能最丰富、最易于使用的 XML 语言服务器。

由于 XML 语言服务器的流行和功能，像 Eclipse(带有 Wild Web Developer)、VS Code(带有 Red Hat 的 XML 语言支持)和 Vim/Neovim(带有`coc-xml`)这样的客户端开始使用 XML 语言服务器。此外，所有 LSP 特性(完成、验证、快速修复等。)很容易扩展。这有助于激励其他项目扩展 LSP 特性，而不是自己从头开始实现它们。

例如，有专门针对 Maven 和 Liferay 的扩展。Maven 扩展扩展了完成特性以管理高级依赖项完成，Liferay 扩展扩展了悬停特性以适应特定的用例。我们希望对 Eclipse Foundation 的贡献有助于相关项目的消费，并吸引 Red Hat 以外的新贡献者。

## 红帽 VS 代码 XML 扩展

此外，我们发布了 [Red Hat VS Code XML 扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml)(当然，它消耗了 Eclipse LemMinX XML 语言服务器来提供语言特性)。这个扩展为在 VS 代码中编辑 XML、XSD 和 DTD 文件提供了一个优秀的一体化包，但是使这个扩展脱颖而出的是对 XML 文件的 XSD 和 DTD 模式验证的支持。

这个新版本也集中在 bug 修复上，在这里的变更日志中有[的概述。](https://github.com/redhat-developer/vscode-xml/blob/master/CHANGELOG.md)

*Last updated: June 29, 2020*