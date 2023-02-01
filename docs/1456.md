# 升级到 Vaadin Framework 8(第 1 部分，共 2 部分)

> 原文：<https://developers.redhat.com/blog/2017/07/06/upgrading-to-vaadin-framework-8-part-1-of-2>

对于一个主要的版本，你通常会期望在框架的核心部分有重大的修改。但这一次，迁移并不太复杂。不仅仅是因为提供了迁移工具来实现从 Framework 7 到 Framework 8 的平稳过渡，还因为许多组件的 API 有相似之处。

尽管如此，还是需要一个好的升级策略，我在下面的标题下总结了它们:

1.  [升级 POM 文件中的依赖关系](#pom)
2.  [运行 Maven goal vaadin:upgrade8](#upgrade)
3.  [升级附加组件](#addon)
4.  [升级非数据组件](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-2-of-2-#nondata)
5.  [升级数据组件](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-2-of-2-#data)
6.  [回到未来](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-2-of-2-#future)

我决定在升级[一个企业应用](https://github.com/jbossdemocentral/brms-coolstore-demo)时，在摄像机前现场尝试这些步骤，以展示我将面临的挑战，并展示如何解决它们。

准备好您的爆米花，召集您所有的 Vaadin 专家，让我们实时观看将 Vaadin Framework 7 应用升级到 Vaadin Framework 8 应用的第一部分！

在这一部分，我将介绍第一个也是最重要的三个步骤。通过这些步骤，您可以获得一个可编译的工作应用程序，该应用程序已准备好投入生产，并使用了最新版本的 Vaadin Framework 8。

[https://www.youtube.com/embed/_sXbwjZ67bM?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/_sXbwjZ67bM?autoplay=0&start=0&rel=0)

完成视频后，您的应用实际上使用的是 Framework 8，您可以在构建项目的新部分时开始使用新的组件和 API。

总结本视频中的主要步骤和失败之处；首先，熟悉[迁移指南文档](https://vaadin.com/docs/-/part/framework/migration/migrating-to-vaadin8.html)并遵循以下步骤非常重要:

## 1.升级 POM 文件中的依赖关系[ [08:48](https://www.youtube.com/watch?v=_sXbwjZ67bM&t=8m48s)

很明显，我们需要将 7.x 版的依赖关系更改为 8.x 版，但是我们还需要检查依赖关系中的其他更改。我们需要使用兼容性包来保持从旧组件到新组件的平稳而缓慢的过渡，而不破坏大量代码。我们可能还会添加一些新的参数，比如 widgetset.mode(这是在 Vaadin 7.7 releaser 中技术上添加的)。

我是通过与为 Framework 8 创建的示例项目的 POM 文件进行比较来做到这一点的，以查看与我自己项目的 POM 文件的差异。

## 2.运行 Maven goal vaa din:upgrade 8[[19:31](https://www.youtube.com/watch?v=_sXbwjZ67bM&t=19m31s)

最终，由于新引入的库和包，第一步会产生很多错误。幸运的是，Framework 8 提供了一个简单的工具，可以帮助您修复大多数损坏的包。要使用该工具，只需运行:

```
mvn vaadin:upgrade8
```

这将检查您的源代码，并更改所有具有不兼容更改的 Vaadin 导入，以使用兼容性包中的版本。要修复剩余的编译错误，可能需要手动进行一些小的更改，比如在标签组件中使用新的 ContentMode 枚举。

## 3.升级附加组件[ [21:49](https://www.youtube.com/watch?v=_sXbwjZ67bM&t=21m49s)

除非你使用的是 vanilla Vaadin 框架，没有任何外部依赖，否则你最终会遇到这个问题。我不得不说，这是迁移中最棘手、最复杂的部分。让我们将附加组件分为 5 类:

#### 3.1 专业工具:

它们在 Framework 8 中得到完全迁移和完全支持。跳转到迁移指南的本节，您将找到关于如何执行迁移的重要详细信息。

#### Vaadin 官方附加组件:

它们在 Framework 8 中也得到完全迁移和完全支持。你所要做的就是从目录中检查最新版本，并在你的 POM 文件依赖关系中更新相应的版本。BOM 里面还提供了其他一些官方的附加版本，你只需要从 dependency 部分去掉版本号就可以了，比如 Vaadin CDI，和 Vaadin Spring。

#### 3.3 集成的 Vaadin 附件:

Vaadin Icons Add-on 现在是核心框架 8 的一部分。如果您正在使用它，那么从您的 POM 文件中删除依赖项，从您的源代码中删除旧的导入，并使它们指向来自核心框架的新包:

```
org.vaadin.teemu.VaadinIcons
```

收件人:

```
com.vaadin.icons.VaadinIcons
```

#### 3.4 迁移的社区附加组件:

许多插件开发者已经将他们的插件迁移到了 Framework 8，这太棒了！检查目录以查看 Framework 8 兼容版本的版本号，并可能查看开发人员提供的关于如何执行升级的其他文档。很多情况下，你要做的就是升级版本。在某些情况下，尤其是在与数据相关的附加组件中，当您需要提供一个兼容层来与旧的 Framework 7 数据绑定 API 一起工作时，您可能需要进行手动修改才能让它们工作。

一个例子是在这个项目中使用的 [Viritin](https://vaadin.com/directory/-/directory/addon/#!addon/viritin) 插件:[Matti](https://vaadin.com/web/matti)——插件维护者——提供了一个类似于核心框架中的兼容层，使得使用最新版本的 Framework 8 成为可能，同时保留旧的 Framework 7 数据绑定代码。您只需将包导入从:

```
org.vaadin.viritin.fields.*
```

收件人:

```
org.vaadin.viritin.v7.fields.*
```

迁移工具不会自动处理这些问题，您需要手动修复它们。

#### 3.5 非迁移社区附加组件:

最不幸的情况是，您依赖于尚未升级到 Vaadin 8 的附加组件。你的希望还没有破灭。你有几个选择可以做:

*   Ping 开发者，例如通过 GitHub，让他/她知道需要 Vaadin 8 兼容版本。但是要有礼貌，要明白他/她这么做很可能不是为了钱。
*   帮助维护人员进行升级。您可以派生项目，进行升级，并在应用程序中使用本地版本。最后，向最初的开发人员创建一个 pull 请求，这样社区的其他人就可以获得一个新版本。
*   你也可以在论坛、评论区或 GitHub 上敦促其他社区成员帮助升级。或者，如果您有多余的预算，请 Vaadin 的专家为您进行升级。
*   您也可以尝试从目录中搜索一个替代的附加组件，或者看看您现在是否可以使用 Vaadin Framework 8 而不使用它。

此时，您将拥有一个基于 Framework 8 的工作项目。您已经完成了迁移中最大和最复杂的部分。在本博客的下一部分，我将解释如何迁移到更新的组件，以及如何处理新的数据绑定 API。

有没有你希望迁移到 Framework 8 的附加组件？

您是否执行了这里提到的前三个步骤，但仍然出现错误，或者项目仍然无法编译？

**还有什么与迁移无关的吗？**

### 请在下面的评论中告诉我们！

查看第 2 部分！

*Last updated: July 12, 2017*