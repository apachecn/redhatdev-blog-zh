# 对 Apache Camel 0.0.20 版本的 VS 代码语言支持

> 原文：<https://developers.redhat.com/blog/2019/12/16/vs-code-language-support-for-apache-camel-0-0-20-release>

在过去的几个月里，基于 [Apache Camel](https://camel.apache.org/) 的应用程序增加了几个引人注目的新特性来改善开发者的体验。这些更新可以在 Visual Studio (VS)代码扩展的 [0.0.20 版本中获得。](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-apache-camel)

在详细讨论更新列表之前，我想指出，我在标题中提到了 VS 代码扩展版本，因为 VS 代码扩展涵盖了更广泛的新特性。不过，如果您使用的是另一个 IDE，也不用担心，大多数特性在支持 Camel 语言服务器的所有其他 IDE(Eclipse Desktop、Eclipse Che 等等)中都是可用的。

## 骆驼 3 号在里面

Camel 3 是几周前[发布的](https://camel.apache.org/blog/release-3-0-0.html)，Camel 语言服务器内部已经在依赖它了。这对最终用户意味着什么？这意味着默认目录现在使用的是 Camel 3。

如果你还是基于 Camel 2.x，没问题；看看下面这个很棒的功能。

## Camel 目录版本选项

现在可以使用一个参数来选择您想要的 Camel 目录版本。这可以在*文件- >首选项- >设置- > Apache Camel 工具- > Camel 目录版本中指定。*

![Camel Catalog Version Preference in VS Code](img/fda199f9752cfd2d8cc5808e3f242923.png)

## 快速修复和更精确范围的诊断

诊断对于查明代码中的问题非常有用。在以前的版本中，全骆驼 URI 提供了诊断范围误差。现在，对于无效的组件参数键和无效的组件参数枚举值，范围更加精确，并且指向确切的相关键或值。

![](img/bc5afeb11b0f1c81667d24a050fd69b9.png)

对于未知的组件参数键，如果存在相对相似的组件参数键，也可以提供快速修复。这是非常有用的情况下，小错别字。

![](img/b928773a5a37abc3babb6d08d771ce88.png)

## 附加 Camel 组件

如果您使用的 Camel 组件不是 Camel 核心目录的一部分，那么现在可以提供 Camel 组件定义，让工具完全支持它。Camel 组件定义被定义为 JSON。JSON 文件可以在 Camel 组件的 jar 中找到。可以通过 settings.json 中的*文件- >首选项- >设置- >阿帕奇骆驼工具- >额外组件- >编辑来指定首选项*

[https://www.youtube.com/embed/U015RzlgFNM?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/U015RzlgFNM?autoplay=0&start=0&rel=0)

## 属性文件支持

Camel 允许你使用一个属性文件来配置一般的组件属性(见[这里](https://github.com/apache/camel/blob/master/examples/camel-example-main/src/main/resources/application.properties#L42)的例子)。组件 id 和组件属性键可以完成。

![](img/77b6f4390c131b7cfe6a502d57c05a75.png)

## 下一步是什么？

这是非常开放的未来。肯定会有针对阿帕奇骆驼 K 支持的改进。Fuse 工具团队正在等待您对 [JIRA](https://issues.redhat.com/browse/FUSETOOLS2) 的反馈，在 [Camel 工具 GitHub 库](https://github.com/camel-tooling)之一。或者任何你喜欢的频道。这将有助于推动未来的路线图。

*Last updated: July 1, 2020*