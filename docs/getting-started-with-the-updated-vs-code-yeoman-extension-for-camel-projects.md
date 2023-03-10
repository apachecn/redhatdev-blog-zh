# Camel 项目的最新 VS 代码 Yeoman 扩展入门

> 原文：<https://developers.redhat.com/blog/2019/03/21/getting-started-with-the-updated-vs-code-yeoman-extension-for-camel-projects>

[Visual Studio (VS)代码 IDE](https://code.visualstudio.com/) 是 JavaScript、C#和 Python 开发人员最常用的平台之一，并迅速成为 Red Hat 的[三大工具环境之一。VS 代码是高度可定制的，并为所有类型和技术的扩展提供了一个健康和不断增长的市场，包括对](https://developers.redhat.com/blog/category/vs-code/)[约曼](https://yeoman.io/)的扩展。在本文中，我将解释如何开始使用与最新版本的 VS 代码一起工作的更新的扩展。

你可能还记得我以前的文章([用新的基于 Yeoman 的项目生成器](https://developers.redhat.com/blog/2019/01/07/using-the-yeoman-camel-project-generator-to-jump-start-a-project/)启动 Camel 项目)，Yeoman 将自己描述为“…一个通用的脚手架系统，允许创建任何类型的应用程序。它允许快速启动新项目，并简化现有项目的维护。”以一种不可知论的方式，约曼让用户拼凑整个项目或只是部分。

我们使用 Yeoman 不仅是为了创建我们自己的 [Camel 项目生成器](https://github.com/camel-tooling/generator-camel-project)，也是为了搭建新的 VS 代码扩展。Red Hat 和 Apache Camel 开发人员最近创建了不少扩展，包括 Red Hat 对 Java 的[语言支持，Red Hat](https://marketplace.visualstudio.com/items?itemName=redhat.java) 的[项目初始化器，以及 Apache Camel](https://marketplace.visualstudio.com/items?itemName=redhat.project-initializer) 的[语言支持。](https://marketplace.visualstudio.com/items?itemName=camel-tooling.vscode-apache-camel)

不幸的是，现有的用于 Yeoman 的 [VS 代码扩展自 2017 年年中以来一直没有更新，并在 2018 年的某个时候停止了与最新 IDE 的正常工作。为了补救这种情况，我们决定创建一个 fork 并更新它。](https://marketplace.visualstudio.com/items?itemName=samverschueren.yo)[新的 VS Code Yeoman 项目](https://github.com/camel-tooling/vscode-yeoman)才刚刚开始，但是它已经包含了补丁，所以现在可以在最新版本的 VS Code 中运行。

## 要求

使用 VS 代码 Yeoman 扩展需要一些项目已经存在并安装在系统上，包括 Node.js、NPM 和 Yeoman。VS 代码文档为确保在 VS 代码实例中设置这些组件提供了很好的指导(参见设置指南中的"[附加组件](https://code.visualstudio.com/docs/setup/additional-components))。

约曼有自己的[发电机市场](https://yeoman.io/generators/)可供选择，包括我们自己的[骆驼项目发电机](https://github.com/camel-tooling/generator-camel-project)。有关安装和运行 Camel 项目生成器的详细信息，请参阅“[使用新的基于约曼的项目生成器](https://developers.redhat.com/blog/2019/01/07/using-the-yeoman-camel-project-generator-to-jump-start-a-project/)快速启动 Camel 项目”一文。

## 安装 VS 代码约曼扩展

我们已经在 [VS Code Marketplace](https://marketplace.visualstudio.com/VSCode) 发布了一个更新的 VS Code Yeoman 扩展的预览版本，你可以在这里找到它的列表[。](https://marketplace.visualstudio.com/items?itemName=camel-tooling.yo)

要在 VS 代码实例中安装扩展，通过单击 VS 代码旁边的活动栏中的扩展图标或 View: Extensions 命令(Ctrl+Shift+X)来打开扩展视图。在 marketplace 文本框中，键入`yo`，然后查找 Camel Tooling 发布的`yo`扩展(目前版本为 0.9.5 ),并单击它的 Install 按钮。安装完成后，重新加载您的工作台，工具就可以使用了！

## 使用约曼扩展

在 VS 代码中，按 Ctrl+Shift+P 调出命令面板并键入`yeoman`。当您点击 Enter 键时，您将看到一个已经安装并可供使用的约曼发电机列表。选择生成器后，根据提示回答每个问题以完成该过程。

例如，要使用 camel 项目生成器:

1.  为您的新 Camel 项目创建一个目录。
2.  打开 VS 代码，打开目录(文件->打开文件夹)。
3.  按 Ctrl+Shift+P 打开命令面板，键入`yeoman`。
4.  选择 camel-project 生成器(可能需要单击鼠标)。
5.  按照提示操作。输入文本以覆盖默认值，然后按 Enter 键。
    *   项目名称(默认为目录的名称)
    *   Camel 版本(默认为 2.22.2)
    *   Camel DSL(类型`spring-boot`)
    *   包名(默认为“com”+目录名，但是您可能需要将其更改为有效的 Java 风格的包名)

当您完成时，您的目录将被所有必要的项目文件填充，这些文件是启动一个简单计时器的 Mavenized Camel Spring Boot 项目所必需的。

您可以打开一个终端窗口并键入`mvn install spring-boot:run`来本地运行项目。或者您可以安装[“maven for Java”扩展](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-maven)来通过 Maven 项目侧栏执行 Maven 命令:

*   安装并填充了 Maven for Java 扩展(一旦 Camel 生成器完成创建工件，就应该构建项目)，右键单击根项目 POM 并选择 Custom Goals。
*   键入`spring-boot:run`并按回车键。

您的新 Camel 项目应该在一个新的终端窗口中运行，并最终显示日志消息，同时定时运行。

下面是整个过程的一个例子:

![](img/5a87b422a3a89cfb7cc5b5764b7f5cf3.png)

## 将来的

这是一个有趣的项目，我希望能继续深入下去。展望未来，我们希望在几个地方改进 Yeoman 扩展，包括为 Camel 项目生成器(和其他生成器)提供更好的下拉列表提示，改进命令面板输入框中的验证，以及可能直接安装新的 Yeoman 生成器。然而，获得一个工作扩展是第一步！

有关 VS Code Yeoman 扩展本身的更多信息，请查看 GitHub 项目页面。

有关 Camel 项目生成器的更多信息，请查看 [NPMJS 页面](https://www.npmjs.com/package/generator-camel-project)。

另请参见[红帽保险丝](https://developers.redhat.com/products/fuse/overview/)页面。

*Last updated: March 22, 2019*