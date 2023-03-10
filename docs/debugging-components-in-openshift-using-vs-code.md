# 使用 VS 代码调试 OpenShift 中的组件

> 原文：<https://developers.redhat.com/blog/2020/02/28/debugging-components-in-openshift-using-vs-code>

最新发布的 [OpenShift 连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)增强了开发者在[红帽 OpenShift](http://developers.redhat.com/openshift/) 上的体验，支持本地代码调试。这种增强让用户无需离开编辑器就可以编写和调试本地代码。

根据与 OpenShift Connector for Visual Studio Code(VS Code)相关的开发人员社区反馈，主要关注点之一是简化开发人员的 OpenShift 工作流，让他们直接从 VS 代码调试部署在 open shift 上的代码。Visual Studio 代码的调试架构允许扩展作者轻松地将现有调试器集成到 VS 代码中，同时为所有调试器提供一个公共的用户界面。

遵循这个原则，我们为 VS 代码的 OpenShift 扩展添加了一个新的调试特性。该特性允许直接从 IDE 调试部署在 OpenShift 实例上的本地节点和 Java 组件。

这个特性从 0.1.3 版本开始对社区开放，扩展可以从 [VS 代码市场](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)安装。

## 它是如何工作的？

这个版本提供了新的`OpenShift: Debug`命令，它提供了一种更简单的方式来开始调试推到集群的 OpenShift 组件。

`OpenShift: Debug`是一个实验性的特性，使用实验性的 OpenShift Do ( `odo` ) `debug`命令下的引擎盖。这种增强允许开发人员直接在源代码中设置断点，监视变量，并在调试时跟踪完整的调用堆栈—所有这些都不需要离开编辑器。

**注意**:只有使用`local workspace`创建并部署在 OpenShift 上的组件才支持调试功能。使用`Git Repository`和`Binary File`创建的组件*不受*支持。

可以通过以下方式调用`debug`命令:

1.  使用**命令面板** [OpenShift: Debug](图 1)

[![openshift-connector-debug-command-palette](img/01975294c9d8b9e129cb795f5551151c.png "Screenshot 2020-02-20 at 4.30.01 AM")](/sites/default/files/blog/2020/02/Screenshot-2020-02-20-at-4.30.01-AM.png)

图 1:通过命令面板访问`OpenShift: Debug`命令。">

2.使用 **OpenShift 应用浏览器**视图。用户需要进入组件节点的上下文菜单并选择`Debug`动作，如图 2 所示。

[![openshift-connector-debug-context-menu](img/1d1fd56486f3a48c0c2ad70822abccf6.png "Screenshot 2020-02-20 at 4.35.40 AM")](/sites/default/files/blog/2020/02/Screenshot-2020-02-20-at-4.35.40-AM.png)

图 2:通过 **OpenShift 应用浏览器**视图访问`OpenShift: Debug`命令。”>

### 在 OpenShift 中调试 NodeJS 组件

默认的 Visual Studio 代码安装包括 JavaScript/TypeScript 语言支持和调试 NodeJS 组件所需的调试器扩展。这意味着新的`OpenShift: Debug`命令可以在不安装任何附加扩展的情况下使用。要开始使用:

1.  从 VS 代码中的[市场](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)或**扩展**图标安装 OpenShift 连接器。
2.  登录正在运行的 OpenShift 集群。
3.  从本地工作区创建一个 NodeJS 组件，并将其部署在 OpenShift 中。
4.  执行如下视频所示的调试操作，该视频详细展示了在 OpenShift 上调试本地 NodeJS 组件的工作流程:

https://developers.redhat.com/media/119201

### 在 OpenShift 中调试 Java 组件

要调试一个 Java 组件，需要 [Java 语言支持](https://marketplace.visualstudio.com/items?itemName=redhat.java)和 [Java 调试器](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug)扩展。因此，在启动 Java 组件的调试器之前，OpenShift 连接器扩展将提示您安装缺少的扩展。要开始使用:

1.  从 VS 代码中的[市场](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)或**扩展**图标安装 OpenShift 连接器。
2.  登录正在运行的 OpenShift 集群。
3.  从本地工作区创建一个 Java 组件，并将其部署在 OpenShift 中。
4.  执行如下视频所示的调试操作，该视频详细展示了在 OpenShift 上调试本地 Java 组件的工作流程:

https://developers.redhat.com/media/119191

### 反馈

我们已经在 [GitHub](https://github.com/redhat-developer/vscode-openshift-tools) 上发布了这个扩展，作为麻省理工学院许可的开源项目，这个扩展可以在 [VS 代码市场](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)上获得。我们非常希望得到您的反馈，并帮助构建更好的调试体验。如果您有任何问题或改进的想法，请随时通过 [Gitter](https://gitter.im/redhat-developer/openshift-connector) 或 [GitHub](https://github.com/redhat-developer/vscode-openshift-tools) 联系我们。

在下一个版本中，期待更多令人敬畏的特性！

*Last updated: May 13, 2021*