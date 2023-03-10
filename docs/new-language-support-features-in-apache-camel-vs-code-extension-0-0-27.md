# Apache Camel VS 代码扩展 0.0.27 中的新语言支持特性

> 原文：<https://developers.redhat.com/blog/2020/09/18/new-language-support-features-in-apache-camel-vs-code-extension-0-0-27>

在这篇文章中，我分享了最近发布的对 Apache Camel VS 代码扩展 0.0.27 的[语言支持中的几个新的](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-apache-camel)[语言支持](https://microsoft.github.io/language-server-protocol/)特性。在我讨论这些改进之前，请注意 VS 代码扩展的更新在其他支持 [Camel 语言服务器](https://github.com/camel-tooling/camel-language-server/)的 IDE 中可用，包括 [Eclipse IDE](https://www.eclipse.org/ide/) 、 [Eclipse Che](https://www.eclipse.org/che/) 和 [more](https://github.com/camel-tooling/camel-language-server/#clients) 。将我的演示集中在一个 IDE 上会更容易，所以我选择了[而不是代码](https://developers.redhat.com/blog/category/vs-code/)。

**注** : [Apache Camel](https://camel.apache.org/manual/latest/) 是一个基于已知[企业集成模式](https://camel.apache.org/components/latest/eips/enterprise-integration-patterns.html)的通用开源集成框架。

## 属性的虚线符号

默认情况下，Camel 语言服务器使用 CamelCase 作为属性名。在 0.0.27 版本中，工具现在也支持使用 dash-case。您可以选择您认为可读性最好的符号，并根据给定的上下文(SpringBoot、Quarkus、Kafka、Spring 和其他使用属性文件的技术)进行调整。例如， [Apache Camel Kafka 连接器](https://camel.apache.org/camel-kafka-connector/latest/)更喜欢虚线符号。

如图 1 所示，悬停特性适用于驼峰式和破折号。

[![A demonstration of the cursor hovering over properties file names in the VS Code console.](img/020d17c43543f5eaf4027e299bb9d31a.png "Hover Dashed Names In Properties File")](/sites/default/files/blog/2020/09/hoverDashedNamesInPropertiesFile.gif)

Figure 1: Use the hover-over feature for a property definition (demo).

如图 2 所示，如果在同一个文件中检测到另一个虚线名称，补全特性会提供虚线符号。否则，将提供驼峰式符号。

[![A demonstration of the dash-case completion option For component names in properties files.](img/4451aa6206364ff2939fa8c6925ad251.png "Dashed Completion For Component Option Name In Properties Files")](/sites/default/files/blog/2020/09/dashedCompletionForComponentOptionNameInPropertiesFiles.gif)

Figure 2: Use the completion feature to update the case style in your properties files (demo).

**注意**:该特性在属性文件中可用。您也可以使用 [Camel K Modeline](https://developers.redhat.com/blog/2020/08/31/introducing-ide-support-for-apache-camel-k-modeline/) `property`选项进行访问。

## 改进了 ide 对 apache camel k 建模的支持

在我的[上一篇文章](https://developers.redhat.com/blog/2020/08/31/introducing-ide-support-for-apache-camel-k-modeline/)中，我分享了 IDE 支持 Apache Camel K Modeline 的第一次迭代的细节。在这个版本中，我们更新了完成特性，增加了`property-file`和`resource`选项。完成操作提供任何深度的同级文件和同级文件夹中的文件。例如，在以下内容中，蓝色文件在完成时提供:

```
| anotherDir--| fileInAnotherDir.yaml| dirWithCamelKIntegrationFile
--| test.camelk.yaml  <-- opened file in which we are providing the completion
--| sibling-file1.txt
--| sibling-file2.json
--| my-sibling-folder
   --| sub-sibling-file3.yaml
   --| aSubFolder      --| sub-sibling-file4.txt
```

## Camel 目录 3.5 更新

[Apache Camel 3.5 最近发布](https://camel.apache.org/blog/2020/09/Camel35-Whatsnew/)，默认提供新的 3.5 目录版本。您可以使用**文件- >首选项- >设置- > Apache Camel 工具- > Camel 目录版本**设置来指定目录的不同版本。参见 [Camel 目录版本选项](https://developers.redhat.com/blog/2019/12/16/vs-code-language-support-for-apache-camel-0-0-20-release/)了解更多关于选择您想要的目录版本的信息。

## 阿帕奇骆驼 K 的下一步是什么

Camel K 的未来是开放的，我们一定会继续改进语言扩展支持。Fuse Tooling 团队正在等待您对 [JIRA](https://issues.redhat.com/browse/FUSETOOLS2) 、Camel Tooling [GitHub 资源库](https://github.com/camel-tooling)的反馈，或者您喜欢的任何方法，比如本文中的评论或推文给我( [@apupier](https://twitter.com/apupier) )或 [@FuseTooling](https://twitter.com/FuseTooling) )。

您的反馈有助于我们制定未来改进的路线图。

*Last updated: September 21, 2020*