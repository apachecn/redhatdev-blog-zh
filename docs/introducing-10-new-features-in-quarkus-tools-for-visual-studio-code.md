# 介绍 Quarkus Tools for Visual Studio 代码中的 10 项新功能

> 原文：<https://developers.redhat.com/blog/2020/02/07/introducing-10-new-features-in-quarkus-tools-for-visual-studio-code>

Quarkus Tools for Visual Studio 代码版本 1.3.0 已经在 VS 代码市场上发布，开始新的一年。随着 Quarkus 不断推出改进和新特性，如`application.yaml`和服务器端模板支持，quar kus Visual Studio 代码工具也在不断发展，以适应这些新特性和改进。

有关所有变更的列表，请参考[变更日志](https://github.com/redhat-developer/vscode-quarkus/blob/master/CHANGELOG.md)。

您还可以[观看本文中所有特性的演示视频](https://youtu.be/6SZPJOaswtA)。

## 新功能

quar kus Tools for Visual Studio Code 1 . 3 . 0 新增的功能包括:

*   rest easy JAX RS GET 方法的新 URL CodeLens。
*   Java 源代码中对`@ConfigProperty`名称的悬停支持。
*   REST 客户端的微配置文件属性支持。
*   Kubernetes、 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 、Docker 和 S2I 物业支持。
*   添加要忽略的未知属性命名空间的快速修复。
*   从`application.properties`转到枚举值的定义支持。
*   YAML 支持(实验性)。
*   一个新的扩展描述切换按钮。
*   打开新 Quarkus 项目的不同方式。
*   Qute 语言的语法高亮显示。

## RESTEasy JAX-RS GET 方法的 URL CodeLens

当当前 Quarkus 应用程序运行在开发模式(`./mvnw compile quarkus:dev`或`./gradlew quarkusDev`)下编辑资源类时，现在有 CodeLenses 为 GET 端点提供 URL。这个特性会考虑到您的`application.properties`文件中的路径名和 HTTP 服务器端口，以便创建 URL。

单击 CodeLens URL 会在默认浏览器中打开这个 URL，如图 1 所示。

[![](img/a22e049b325d35bfdbb5d428a2c7df7e.png "codelensURL-min")](/sites/default/files/blog/2020/01/codelensURL-min.gif)Figure 1: Clicking the CodeLens URL opens it in your default browser.">

一旦 Quarkus 应用程序停止运行，CodeLens URL 将不再出现，如图 2 所示。

[![Animation showing edits and then the missing CodeLens URL.](img/1c454bd495594353985717d42427a73a.png "codelensURL2-min")](/sites/default/files/blog/2020/01/codelensURL2-min.gif)Figure 2: The CodeLens URL disappears once the Quarkus application stops running.">

请记住，VS 代码中的 CodeLenses 只有在某些事件发生时才会更新。如果 URL CodeLens 没有出现，有两种简单的方法来触发 CodeLens 更新:切换标签或开始在任何文件中键入。此外，确保`quarkus.tools.codeLens.urlCodeLensEnabled` VS 代码设置设置为`true`。

## Java 源代码中对`@ConfigProperty`名称的悬停支持

将鼠标悬停在`@ConfigProperty`注释中的 name 值上，现在会显示悬停属性的值，如图 3 所示。目前，该值或者来自`application.properties`文件，或者来自缺省值字段。

[![Animation showing that you can hover over these name values to see the property's value.](img/05cad3906e2efb194df6e371c257e4a4.png "hoverconfigproperty-min")](/sites/default/files/blog/2020/01/hoverconfigproperty-min.gif)Figure 3: Hover over a `@ConfigProperty` annotation's name value to display the hovered property's value.">

## REST 客户端的微配置文件属性支持

现在，REST 客户端已经完成了对微文件属性的完成、悬停、文档化和验证。使用`@RegisterRestClient`注册 REST 客户端后，如下所示:

```
package com.mycompany.remoteServices;

@RegisterRestClient
public interface MyServiceClient {
    @GET
    @Path("/greet")
    String greet();
}
```

语言特性将对相关的 MicroProfile 配置属性变得可用，如图 4 所示。

[![Animation showing the new properties available in the extension.](img/9d3aeb88c2d1261cf48540b6547f193b.png "MPRest")](/sites/default/files/blog/2020/02/MPRest.gif)Figure 4: Language features for the new properties.">

有关使用 MicroProfile REST 客户端的更多信息，请参见 Quarkus 指南[这里的](https://quarkus.io/guides/rest-client)。

## Kubernetes、Openshift、Docker 和 S2I 物业支持

同样，现在对来自 [Kubernetes Quarkus 扩展](https://quarkus.io/guides/kubernetes#enable-kubernetes-support)的`kubernetes.*`、`openshift.*`、`docker.*`和`s2i.*`属性进行完成、悬停、文档化和验证，如图 5 所示。

[![Animation showing the new properties available in the extension.](img/8ec06368b6fe66929ca0e46b76141f0a.png "kubopedocs2i-min")](/sites/default/files/blog/2020/01/kubopedocs2i-min.gif)Figure 5: Languages features for the new properties.">

关于生成 Kubernetes 资源和相关配置属性的深入文档可以在 Quarkus 指南[这里](https://quarkus.io/guides/kubernetes#configuration-options)中找到。

## 添加要忽略的未知属性命名空间的快速修复

现在有一个新的快速修复方法，可以帮助您从未知属性验证中排除大量未知属性，只要它们共享相同的父名称空间。例如，如果您的`application.properties`文件包含四个带有未知属性错误的属性，如下所示:

```
# All four properties cause an 'Unknown property' error
unknown.test1=a
unknown.test2=b
unknown.test3=c
unknown.test4=d
```

忽略未知属性验证的所有四个属性很容易通过快速修复完成，它将`unknown.*`添加到`quarkus.tools.validation.unknown.excluded`工作区配置数组，如图 6 所示。

[![Animation showing how you can ignore certain properties.](img/5cf84418ad86d8636407c286ab279471.png "codeaction")](/sites/default/files/blog/2020/01/codeaction.gif)Figure 6: Ignoring properties from unknown property validation.">

## 从`application.properties`转到枚举值的定义支持

到目前为止,“转到定义”仅支持配置属性键，不支持它们的值。这个版本为枚举值带来了 Go to definition 特性，如图 7 所示。

[![Animation showing how to use the definition features for enum values.](img/3d38a5176b5ed05e963efbe6fd18fb07.png "enumvalue")](/sites/default/files/blog/2020/01/enumvalue.gif)Figure 7: Using the Go to definition feature with enum values.">

## YAML 支持(实验)

[Quarkus 1.1.0.Final](https://quarkus.io/blog/quarkus-1-1-0-final-released/) 的发布，带来了 [YAML 配置支持](https://quarkus.io/guides/config#yaml)，这意味着你现在可以用`application.yaml`文件或`application.properties`文件来配置你的 Quarkus 应用程序(但尽量坚持使用其中一个)。

结果，现在有了对`application.yaml`文件的完成支持，如图 8 所示。与`application.properties`类似，`application.yaml`文件中的完成选项将与当前项目(在`pom.xml`或`build.gradle`中)可用的 Quarkus 扩展同步，因此只给你相关的完成选项。

这个特性依赖于由红帽扩展的 [YAML 语言支持。如果当前没有安装，一个新的提示将建议安装它。](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)

[![Animation showing the completion support in use.](img/76a8eef7e20f9c99357fc30f870efaef.png "yamlcompletion_gif")](/sites/default/files/blog/2020/02/yamlcompletion_gif.gif)Figure 8: New completion support for the `application.yaml` file.">

对`application.yaml`文件的语言特性支持还处于试验阶段。与`application.properties`支持相比，缺少一些功能:

*   转到定义支持。
*   代码操作支持。
*   默认值的自动完成。
*   有限的配置属性和值验证支持。

## 一个新的扩展描述切换按钮

随着 Quarkus 扩展的数量不断增加，扩展选择提示中的新扩展描述将帮助您识别和发现新的扩展，如图 9 所示。从 **Quarkus:生成 Quarkus 项目**和 **Quarkus:向当前项目**添加扩展向导中选择 Quarkus 扩展时，将出现扩展选择提示。

[![Animation showing extension discovery before and after this release.](img/41aec2afc6bf26a6f6646a9daeca41f1.png "extensions_before_after_text")](/sites/default/files/blog/2020/01/extensions_before_after_text.png)Figure 9: Discover new extensions with the new descriptions.">

在选择框的右上方还有一个新按钮，可以切换扩展描述是否应该出现，如图 10 所示。

[![Animation showing how to use the toggle for extension descriptions.](img/ecc05972e3db11256d1787c99369f431.png "toggle_gif")](/sites/default/files/blog/2020/02/toggle_gif.gif)Figure 10: Toggle whether or not extension descriptions should appear.">

## 打开新 Quarkus 项目的不同方式

使用**quar kus:Generate a quar kus project**向导创建新项目后，现在会有一个新的提示，询问新项目应该如何打开。下面的前的*和*后的*图描述了这些变化:*

[!['Before' diagram.](img/1365f4b57fd1ca60b9a288dcc58de662.png "gen_before_updated")](/sites/default/files/blog/2020/02/gen_before_updated.png)Figure 11: *(Before)* Scenarios and options provided when generating a new Quarkus project.">[!['After' diagram.](img/091493267bd5168f83c4eedf31faedb8.png "gen_after_updated")](/sites/default/files/blog/2020/02/gen_after_updated.png)Figure 12: *(After)* Scenarios and options provided when generating a new Quarkus project.">

为了帮助可视化一个可能的场景，图 13 显示了在工作区打开的情况下生成项目时出现的选项。

[![](img/7d9aaca6b705893e2f17491287223cbc.png "newproject")](/sites/default/files/blog/2020/01/newproject.gif)Figure 13: Creating a new project with the **Quarkus: Generate a Quarkus project** wizard.">

## Qute 语言的语法高亮显示

[Qute](https://quarkus.io/guides/qute-reference) 是一个新的服务器端模板引擎，是为 Quarkus 设计的。这个版本在 VS 代码中引入了新的 Qute 语言模式:Qute HTML、Qute JSON、Qute YAML 和 Qute Text。如果您的文件扩展名分别为`.qute.html`、`.qute.json`、`.qute.yaml`或`.qute.txt`，这些新的语言模式将自动应用于您当前的文件。

由于新的语言模式，现在提供了特定于 Qute 的语法高亮和注释，如图 14 所示。

[![Animation showing Qute-specific syntax highlighting and commenting.](img/743fcc1d2c4859a311139c75167d6f12.png "qute_gif")](/sites/default/files/blog/2020/02/qute_gif.gif)Figure 14: Qute-specific syntax highlighting and commenting.">

有关 Qute 模板引擎的更多信息，请参考 [Quarkus 模板引擎指南](https://quarkus.io/guides/qute)。

## 走向

这总结了这个版本中的新的主要特性。如果您有任何建议或反馈，请随时[打开 GitHub 问题](https://github.com/redhat-developer/vscode-quarkus/issues/new)。

对于未来的版本，除了一般的增强，我们的目标是为`application.yaml`和 Qute 语言带来更强大的语言特性支持。敬请期待下一个版本！

## 链接

以下是重要的链接:

*   [VS 代码市场](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-quarkus)
*   [GitHub 库](https://github.com/redhat-developer/vscode-quarkus)
*   [开一期 GitHub](https://github.com/redhat-developer/vscode-quarkus/issues/new)
*   [变更日志](https://github.com/redhat-developer/vscode-quarkus/blob/master/CHANGELOG.md)
*   [1 . 2 . 0 版本发布文章](https://developers.redhat.com/blog/2019/11/21/new-features-in-quarkus-tools-for-visual-studio-code-1-2-0/)
*   [1 . 0 . 0 版本发布文章](https://developers.redhat.com/blog/2019/09/23/how-the-new-quarkus-extension-for-visual-studio-code-improves-the-development-experience/)

*Last updated: June 29, 2020*