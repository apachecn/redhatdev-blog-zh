# JetBrains IntelliJ Red Hat OpenShift 扩展为 open shift 组件提供调试支持

> 原文：<https://developers.redhat.com/blog/2020/04/03/jetbrains-intellij-red-hat-openshift-extension-provides-debug-support-for-openshift-components>

用于 JetBrains IntelliJ 的 0.2.0 发布版本的 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 扩展现已推出。你可以从 JetBrains 插件库中下载 [OpenShift 连接器](https://plugins.jetbrains.com/plugin/12030-openshift-connector-by-red-hat)扩展。这个版本提供了一个新的 **OpenShift: Debug** 动作来简化推送到集群的 OpenShift 组件的调试。它类似于为 [Visual Studio 代码](https://developers.redhat.com/blog/2020/02/28/debugging-components-in-openshift-using-vs-code/)和 [JBoss Tools for Eclipse](https://tools.jboss.org/blog/12.14.0.ga.html) 开发的特性。OpenShift 连接器使用[open shift Do](https://github.com/openshift/odo/)s(`odo`s)debug 命令，仅支持本地 Java 和 Node.js 组件。这种增强让用户无需离开 IntelliJ 就可以编写和调试本地代码。

本文解释了 **OpenShift: Debug** 如何工作，并分享了在 IntelliJ 中调试 Java 和 Node.js 组件的区别。

## OpenShift: Debug 是如何工作的？

调试功能仍处于试验阶段，仅支持 Java 和 NodeJS 组件。当`odo`支持的时候，将会添加更多像 Python 这样的语言。此操作在 OpenShift 视图的组件节点上下文菜单中可用。它允许开发人员像往常一样使用 IntelliJ 来调试应用程序(设置断点、检查堆栈和变量、逐步执行等)。)而应用程序实际上是在 OpenShift 上运行的。

让我们看一下如何一步一步地调试本地组件:

1.  从[市场下载并安装 OpenShift 连接器。](https://plugins.jetbrains.com/plugin/12030-openshift-connector-by-red-hat)
2.  登录到 OpenShift 集群。
3.  如果尚未创建，请在 OpenShift 中创建一个项目。
4.  使用本地模块创建一个组件(或者检查并使用一个[示例](https://github.com/spring-projects/spring-petclinic.git)。)
5.  创建一个 URL 以在浏览器中访问应用程序。
6.  推动组件。
7.  在代码中放置一个断点。
8.  右击组件并选择**调试**。
9.  等待本地调试器连接。
10.  右键单击组件下面的 URL 元素，并选择**在浏览器中打开**。
11.  导航到应用程序，到达代码中设置断点的位置。

回到 IntelliJ:调试器现在是活动的，正在等待操作。

### 调试 Java 组件

在任何版本的 IntelliJ 中都可以调试 Java 组件。只需在 Java 组件的上下文菜单中选择 **Debug** 动作。这样做将自动创建一个新的 Java 远程调试配置，并使用它连接到 OpenShift 上运行的应用程序。

[https://www.youtube.com/embed/rMXFwrBjAP0?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/rMXFwrBjAP0?autoplay=0&start=0&rel=0)

### 调试 Node.js 组件

要调试 Node.js 组件，您需要一个支持 JavaScript 和 Node.js 的 IntelliJ 版本。请参见 JetBrains 网站上的[版本矩阵，了解支持的版本。](https://www.jetbrains.com/idea/features/editions_comparison_matrix.html)

与前面相同的 **Debug** 操作用于调试 Node.js 组件，但是这次它使用 JavaScript 调试器。

[https://www.youtube.com/embed/Bl0gvRLEVTI?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/Bl0gvRLEVTI?autoplay=0&start=0&rel=0)

## 保持联系

如果你想要更多关于这个新特性的信息，可以使用[文档](https://github.com/redhat-developer/intellij-openshift-connector/wiki/How-to-debug-a-component)，你也可以使用[这个 Gitter 频道](https://gitter.im/redhat-developer/openshift-connector)和开发团队聊天。

和往常一样，这个版本的源代码可以在 EPL 许可下在 GitHub 上获得。我们感谢您的反馈和帮助，以改善您的开发者体验，如果您有任何问题或想法，请随时通过 Gitter 联系我们或在 GitHub 上打开问题。

尽情享受吧！

*Last updated: June 29, 2020*