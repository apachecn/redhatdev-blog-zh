# 宣布 Visual Studio 代码的 Red Hat OpenShift 扩展:公共预览

> 原文：<https://developers.redhat.com/blog/2018/11/28/announcing-red-hat-openshift-extension-for-visual-studio-code-public-preview>

我们非常高兴地宣布，Visual Studio 代码的 [Red Hat OpenShift](https://www.openshift.com/) 扩展的预览版现已发布。您可以从市场上下载 [OpenShift 连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)扩展，或者直接从 Visual Studio 代码中的扩展库中安装它。

本文描述了该扩展的特性和优点，并提供了安装细节。它还提供了一个演示，展示了如何使用该扩展来改进在本地 OpenShift 集群上开发和部署 Spring Boot 应用程序的端到端体验。

## 使用扩展的好处

[Red Hat OpenShift](https://www.openshift.com/) 是一个容器应用平台，为企业带来了 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [containers](https://developers.redhat.com/blog/category/containers/) 的强大功能。无论应用程序架构如何，OpenShift 都可以让您轻松快速地在几乎任何基础设施(公共或私有)中进行构建、开发和部署。

因此，无论是在内部、公共云中还是托管，您都有一个屡获殊荣的平台，可以在竞争对手之前将您的下一个伟大想法推向市场。

使用 OpenShift 连接器，您可以使用 OpenShift 集群的本地实例(如 minishift/[Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview/))与 Red Hat OpenShift 进行交互。利用 OpenShift 应用程序浏览器视图，您可以改善开发应用程序的端到端体验。

该扩展使您能够直接使用 Visual Studio 代码执行所有这些操作，并且消除了记忆一些相当复杂的 CLI 命令的复杂性。

## 开发者用例

在开发人员工作站上，当您加载 Spring Boot 项目时，语言支持检测会自动建议加载 Spring Boot 语言支持扩展，并建议下载和安装 OpenShift 连接器。您可以在 Visual Studio 代码中安装推荐的扩展。

因此，一旦安装了 OpenShift 连接器，就可以在 Visual Studio 代码中的资源管理器面板上启用 OpenShift 应用程序视图。然后，您可以访问视图并连接到正在运行的 OpenShift 集群，执行所需的操作。

## 演示

这里是一个使用扩展开发和部署 Spring Boot 应用到本地 OpenShift 集群的端到端体验的工作演示。这个演示旨在简化 Visual Studio 开发人员的 OpenShift 体验。有关详细的安装和使用信息，请参考[自述文件](https://github.com/redhat-developer/vscode-openshift-tools/blob/master/README.md)。

https://www.youtube.com/watch?v=XIHLbUvGuFM

**注意** : *在这个预览版中，我们只支持 Java 和 Node。JS 组件。我们将在未来的版本中支持其他语言。*

## 装置

首先，你需要安装 Visual Studio 代码 1.12.0 或更高版本。

*   若要使用最新版本的 Visual Studio 代码安装扩展，请调出 Visual Studio 代码命令面板(按 F1)。
*   键入`install`和**选择扩展:安装扩展**。
*   在**搜索市场中的扩展**文本框中，输入`OpenShift`。找到红帽发布的`OpenShift Connector`扩展，点击**安装**按钮。
*   随后，您应该重新加载 Visual Studio 代码，并且在资源管理器视图中会出现一个 OpenShift 图标。

[![OpenShift Connector installation demo](img/94d55f73ae4bb64823681b4d165f609a.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/openshift-extension-installation.gif)

## 属国

该扩展使用两个 CLI 工具与 OpenShift 集群进行交互:

*   OpenShift 客户端工具: [oc](https://github.com/openshift/origin/releases)
*   OpenShift Do 工具: [odo](https://github.com/redhat-developer/odo/releases)

如果`oc`和`odo`位于您的路径中包含的目录中，它们将被自动使用。如果它们不在您的路径中，扩展将提示您下载并安装它们。

## 行动中的扩展

### 连接到 OpenShift 实例

1.  使用 minishift/ [Red Hat 容器开发工具包](https://developers.redhat.com/products/cdk/overview/)启动本地 OpenShift 实例。
2.  在 Visual Studio 代码中安装扩展后，会提示您下载所需的依赖项(`oc`、`odo`)。
3.  单击浏览器视图中的 OpenShift 图标后，OpenShift 应用程序浏览器视图将被激活。
4.  然后需要登录到正在运行的 OpenShift 集群(![](img/f54814c702f706ac563d095af35f05c3.png) -登录集群)。
5.  提供集群 URL 以连接到正在运行的 OpenShift 实例。[![logging in to OpenShift](img/6f9ddc5fcd66acd5bc9e446d9f018046.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/Screenshot-2018-11-26-at-1.12.43-AM.png)
6.  现在，您可以使用以下方法登录服务器:
    *   凭据:使用给定的凭据登录给定的服务器(基本身份验证)。
    *   Token:使用给定的凭证(token)登录到给定的服务器。

[![OpenShift login options](img/862585b4fd9713d1ca9c7b2af9c75f2e.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/Screenshot-2018-11-26-at-1.22.06-AM.png)

7.OpenShift 应用程序浏览器将在树形视图中显示 OpenShift 集群。

8.现在，您可以直接从扩展在连接的集群中执行必要的操作，而无需来回查看命令行。

### 使用 OpenShift

一旦扩展连接到 OpenShift 集群，您就可以在 OpenShift 和构建/部署应用程序中执行操作。注意:*这个扩展目前支持使用 [minishift](https://github.com/minishift/minishift/releases) 或[Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/download/)运行的本地 OpenShift 集群。*

#### OpenShift 群集中可用的操作

*   `OpenShift: List catalog components`–列出 OpenShift 图像生成器中所有可用的组件类型。
*   `OpenShift: List catalog services`–列出所有可用的服务，例如 MySQL。
*   `OpenShift: New Project`–在集群内创建一个新项目。
*   `OpenShift: About`–提供关于 OpenShift 工具的信息。
*   `OpenShift: Log out`–退出当前的 OpenShift 集群。

#### 项目可用的操作

*   `Project -> New Application`–在所选项目内创建新的应用程序。
*   `Project -> Delete`–删除现有项目。

#### 项目内应用程序可用的操作

*   `Application -> New Component`–在所选应用程序内创建一个新组件。
    *   git——使用 git 存储库作为组件的源文件。
    *   本地–使用本地目录作为组件的源文件。
*   `Application -> New Service`–执行服务目录操作。
*   `Application -> Describe`–描述终端窗口中给定的应用程序。
*   `Application -> Delete`–删除现有的应用程序。

#### 应用程序中组件可用的操作

*   `Component -> Create URL`–将组件暴露给外界。使用该命令生成的 URL 可用于从集群外部访问已部署的组件。
*   `Component -> Create Storage`–创建存储并安装到组件。
*   `Component -> Show Log`–检索给定组件的日志。
*   `Component -> Follow Log`–遵循给定组件的日志。
*   `Component -> Open in Browser`–在浏览器中打开暴露的 URL。
*   `Component -> Push`–将源代码推送到组件。
*   `Component -> Watch`–观察变化并在变化时更新组件。
*   `Component -> Describe`–描述终端窗口中的给定组件。
*   `Component -> Delete`–删除现有组件。

## 贡献和反馈

这是一个开源项目，我们欢迎贡献和建议。请遵循这些[投稿](https://github.com/redhat-developer/vscode-openshift-tools/blob/master/CONTRIBUTING.md)指南了解更多详情。

我们很高兴您能尝试一下 [OpenShift 连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)!此外，欢迎任何关于在 Visual Studio 代码上使用 OpenShift 进一步改善开发人员体验的反馈。

如果您有任何疑问、遇到任何问题或有功能需求，请联系我们。

*   对我们如何改进扩展有什么想法吗？干脆[开新一期](https://github.com/redhat-developer/vscode-openshift-tools/issues)！
*   有关更多讨论，请在 [Mattermost](https://chat.openshift.io/developers/channels/adapters) 上与我们聊天。

合作愉快，
红帽开发者工具团队

*Last updated: December 5, 2018*