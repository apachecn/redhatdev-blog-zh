# 在 Eclipse Che 中添加对 Apache Camel K 的 Java 语言支持

> 原文：<https://developers.redhat.com/blog/2020/09/02/add-java-language-support-for-apache-camel-k-inside-eclipse-che>

[阿帕奇骆驼 K](https://camel.apache.org/projects/camel-k/) 要尽量轻量化。因此，Camel K 项目提供了描述 Camel 集成的独立 Java 文件。这种做法的缺点是现有的 ide 不能提供开箱即用的完整支持。几个月前，我提到了在 *[红帽 Visual Studio 代码(VS 代码)扩展](https://developers.redhat.com/blog/2019/09/30/sending-a-telegram-with-apache-camel-k-and-visual-studio-code)、*中讨论的[对 Apache Camel K](https://developers.redhat.com/blog/2020/02/03/camel-k-standalone-java-file-now-with-java-language-support) 的 Java 语言支持，以及它如何为 Apache Camel K 提供 [Java 语言支持。在本文和演示中，我将向您展示如何用](https://developers.redhat.com/blog/2020/02/03/camel-k-standalone-java-file-now-with-java-language-support) [Eclipse Che](https://www.eclipse.org/che/) 和 [che.openshift.io](https://che.openshift.io) 做同样的事情。

## 演示:使用 Java 语言插件

我已经[编写了一个示例应用程序](https://github.com/apupier/camelk-on-che-with-java-support-example)来演示如何使用这个插件在 Eclipse Che 中为 Apache Camel K 添加 [Java](https://developers.redhat.com/topics/enterprise-java) 语言支持。如果你在 [che.openshift.io](https://che.openshift.io/dashboard/) 上有一个账户，这个界面提供了一个按钮，你可以点击它在一个现成的 che 工作区中打开这个例子。包含的视频展示了使用预配置的 devfile 启动工作区是多么容易。

幸运的是，Java 语言支持扩展与 Che 上的其他 [Camel K 特性兼容。(单击链接可以获得对 Eclipse Che 内部 Apache Camel K 开发的详细介绍。)](https://developers.redhat.com/blog/2020/01/24/apache-camel-k-development-inside-eclipse-che-iteration-1/)

[https://www.youtube.com/embed/s54uEFYmSGw?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/s54uEFYmSGw?autoplay=0&start=0&rel=0)

## 为 Apache Camel K 项目配置 Java 支持

关键是引用您需要的特定插件配置。在您的 devfile 中，您需要添加对 Camel K 的`chePlugin`的引用:

```
- type: chePlugin
  reference: >-
     https://raw.githubusercontent.com/apupier/camelk-on-che-with-java-support-example/master/.che/camelk-plugin-meta.yaml
alias: vscode-camelk

```

除此之外，Che devfile 配置是典型的。参见 [Eclipse Che 文档](https://www.eclipse.org/che/docs/stable/overview/introduction-to-eclipse-che/)中关于利用 Che devfile 配置可能性的各种方法的讨论。

## 技术见解

所提供的插件配置将用于 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 、Camel K 和 Java 的 VS 代码扩展组合在一起。我们使用一个定制的 docker 文件作为 sidecar 来收集 Che 插件定义中使用的所有扩展的需求。

这些扩展被分组，因为它们都依赖于通过文件系统传递文件(VS 代码全局存储)、共享`kubeconfig`，以及重用命令行工具(`kubectl, mvn`)。将来，我们预计我们将不需要特定的插件定义；我们还希望配置一个预定义的堆栈。

## 结论

我们正致力于将 Java 语言支持直接包含在 Apache Camel K 栈中。为了帮助推广这一倡议，请关注、投票并帮助我们解决[这一问题](https://github.com/eclipse/che/issues/16018)。

*Last updated: November 2, 2022*