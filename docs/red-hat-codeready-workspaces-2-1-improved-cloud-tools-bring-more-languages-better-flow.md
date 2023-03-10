# Red Hat CodeReady Workspaces 2.1:改进的云工具带来了更多的语言，更好的流程

> 原文：<https://developers.redhat.com/blog/2020/04/20/red-hat-codeready-workspaces-2-1-improved-cloud-tools-bring-more-languages-better-flow>

我们很高兴地宣布[Red Hat code ready work spaces 2.1](https://developers.redhat.com/products/codeready-workspaces/overview)的发布。基于 [Eclipse Che](https://www.eclipse.org/che/getting-started/cloud/?sc_cid=701f2000000RtqCAAS) ，其上游项目 CodeReady Workspaces 是一个[Red Hat open shift](https://developers.redhat.com/openshift/)-原生开发者环境，支持开发者团队进行云原生开发。

CodeReady Workspaces 2.1 现已在 OpenShift 3.11 和 OpenShift 4.3+上推出。

这个新版本引入了:

*   **仪表板:**新的入职流程。
*   **夸库:**一个新的工作空间让你从[夸库](https://developers.redhat.com/topics/quarkus/)开始。
*   **语言:**的添加。NET Core 3.1，Java 11，还有 Camel DSL (Apache Camel K)。
*   **其他:**编辑器和 AirGap 改进。

最后但同样重要的是，CodeReady Workspaces 2.1 现在总是使用与 HTTPS 的安全通信。

## **新入职流程**

CodeReady Workspaces 2.1 在其仪表板中提供了一个新的入职流程，如图 1 所示。当没有工作区时，默认情况下会显示此页面。

[![CodeReady Workspaces Getting Started Dashboard page](img/beb38059defb57f09010e2ba804cdd1e.png "img_5e8c1278e54cf")](/sites/default/files/blog/2020/04/img_5e8c1278e54cf.png)

Figure 1: Create new workspaces faster with the new onboarding flow.

getting started 页面允许您快速创建一个新的工作区，它提供了一个创建工作区的按钮，无需进行任何配置。例如，没有 devfiles 或选项，这使得这对于用户来说是最平滑和最容易的入门过程。新的图标、标题和描述也有助于您快速识别技术。

## **整体工作空间创建**

仪表板现在提出启用/禁用临时存储的选项，并显示创建工作区时使用的 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 名称空间，如图 2 所示。

[![The CodeReady WorkSpaces 2.1 New Workspace -&gt; Ready-To-Go Stack dialog box.](img/ef2afbd62e413f69caaf206ada72397e.png "img_5e8c1293b79ed")](/sites/default/files/blog/2020/04/img_5e8c1293b79ed.png)

Figure 2: Set up temporary storage and view this workspace's Kubernetes namespace.

## **Quarkus 工作空间创建**

[![](img/cef4369f85f82d1cd77fc86f7d864b07.png)](https://developers.redhat.com/blog/wp-content/uploads/2020/04/img_5e8c12b49d611.png)

现在，您可以从入门页面创建一个 Quarkus 工作区。默认情况下，该步骤提供创建、读取、更新和删除(CRUD)服务。此工作区包含启动开发人员模式或调试服务的现成命令。这样，在进行代码更改时，就很容易看到实时更新。

这个工作区还包含 [Red Hat VS Code Quarkus 插件](https://github.com/redhat-developer/vscode-quarkus),用于生成新项目、提供代码片段等等，如图 3 和图 4 所示。

[![CodeReady Workspaces 2.1 Red Hat VS Code Quarkus plug-in with &quot;qu&quot; typed at the prompt to show the options that start with &quot;Qu.&quot;](img/18bed56ce00d2352d8b10d7b9d592b97.png "img_5e8c12c280dac")](/sites/default/files/blog/2020/04/img_5e8c12c280dac.png)

Figure 3: Generate your new Quarkus project quickly by selecting the features you need.

[![CodeReady Workspaces 2.1 Quarkus workspace with the &quot;Getting started with Quarkus&quot; file displayed](img/accedfce1cfa21f7a024161ae244307f.png "img_5e8c12d1378cd")](/sites/default/files/blog/2020/04/img_5e8c12d1378cd.png)

Figure 4: Browse the plug-in to access common VS Code commands.

## 语言更新

CodeReady Workspaces 2.1 提供了以下语言更新。

### **。NET 3.1**

这是一个重大更新。CodeReady 工作区的前一版本提供了 2.1 版。

### **骆驼 DSL(阿帕奇骆驼 K)**

[![](img/0e8d249a4a3e99d7dfa7744f64c2fd63.png)](https://developers.redhat.com/blog/wp-content/uploads/2020/04/img_5e8c14501ac54.png)

Apache Camel K 是一个轻量级集成框架，由 Apache Camel 构建，在 Kubernetes 上本地运行，专门为无服务器和微服务架构而设计。Camel K 的用户现在可以立即在他们喜欢的云平台上运行用 Camel DSL 编写的集成代码，比如 OpenShift。

### Java: Java 11 和 Gradle

Gradle 支持现已作为构建工具提供。Java 工作区在处理 Java 项目时可以使用 Maven 或 Gradle。此外，除了现有的 Java 8 插件之外，还有一个新的 Java 11 插件。这两个 java 版本都支持 LTS。

可以用专用的栈试试 Java 11，包括基于 Java 11 的用户镜像。使用 [Red Hat VS Code Java](https://github.com/redhat-developer/vscode-java) 插件，`java` / `javac`工具与一个 IntelliSense 容器一起安装。

## **气隙改进**

通过使用 OpenShift 4.3，现在可以按照[受限网络设置](https://docs.openshift.com/container-platform/4.3/operators/olm-restricted-networks.html)来配置 CodeReady Workspaces 操作符。CRW 也兼容 [OpenShift 4.2 受限网络设置](https://docs.openshift.com/container-platform/4.2/operators/olm-restricted-networks.html)，虽然这个选项涉及更多的手动步骤。

当[Red Hat open shift Container Platform](https://developers.redhat.com/products/openshift)安装在受限网络(也称为断开集群)上时，运营商生命周期管理器(OLM)不能再使用默认的 OperatorHub 源，因为它们需要完全的互联网连接。集群管理员现在可以禁用这些默认源并创建本地镜像，以便 OLM 可以从本地源安装和管理操作员。

## **编辑改进**

我们在 VS 代码 API 兼容性方面有很多改进。更多的插件现在与我们基于 Eclipse 忒伊亚的内部编辑器兼容。此外，还有许多小的增强；例如，文件树，其中的空白空间减少了(图 5 和图 6)。

[![Screenshot of the previous file tree's spacing](img/c114bfb9b3c1b9f7c7f588f2fc4ccc36.png "img_5e8c12f3c8860")](/sites/default/files/blog/2020/04/img_5e8c12f3c8860.png)

Figure 5: The previous version.

[![Screenshot of the new file tree spacing](img/d79e25235af6a6f296048cad70ef8840.png "img_5e8c13058fb1f")](/sites/default/files/blog/2020/04/img_5e8c13058fb1f.png)

Figure 6: The new version.

此外，terminal 选项卡现在显示容器名称，如图 7 所示。

[![](img/f73f35d7275c02f8a046ede0be253fb1.png)](https://developers.redhat.com/blog/wp-content/uploads/2020/04/img_5e8c122bc9648.png)

当您克隆 Git 项目时，不会再提示您将项目存储在何处以进行克隆。新的默认值在`/projects`文件夹中。

## **立即试用 CodeReady work spaces 2.1**

CodeReady Workspaces 2.1 现已在 OpenShift 3.11 和 OpenShift 4.x 上推出。要试用它:

*   如果你使用的是 OpenShift 3.11，你可以[在这里](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.1/html/installation_guide/installing-codeready-workspaces-on-openshift-3-using-the-operator_crw)找到安装说明。
*   如果使用的是 OpenShift 4.x，可以直接从 OperatorHub 安装。[遵循此处的文档](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.1/html/installation_guide/installing-codeready-workspaces-on-ocp-4#installing-codeready-workspaces-on-openshift-4-from-operatorhub_installing-codeready-workspaces-on-openshift-container-platform-4)。
*   [下载 Red Hat CodeReady work spaces CLI](https://developers.redhat.com/products/codeready-workspaces/download)。
*   转到 [Red Hat CodeReady Workspaces 产品页面](https://developers.redhat.com/products/codeready-workspaces)。

*Last updated: June 29, 2020*