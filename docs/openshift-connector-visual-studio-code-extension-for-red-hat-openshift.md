# OpenShift 连接器:Red Hat OpenShift 的 Visual Studio 代码扩展

> 原文：<https://developers.redhat.com/blog/2019/10/31/openshift-connector-visual-studio-code-extension-for-red-hat-openshift>

Red Hat OpenShift 4.2 的新版本有许多针对开发者的改进。在这种背景下，我们发布了新版本的 [OpenShift Connector 0.1.1](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector) ，这是一个 Visual Studio (VS)代码扩展，具有更多改进的功能，可以提供无缝的开发人员体验。开发人员现在可以专注于更高层次的抽象，如他们的应用程序和组件，并可以更深入地了解直接从 VS 代码组成他们的应用程序的 [OpenShift](https://developers.redhat.com/openshift/) 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 资源。

让我们深入了解一下 OpenShift 连接器的新特性。

[https://www.youtube.com/embed/m0wBKuKDYO0?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/m0wBKuKDYO0?autoplay=0&start=0&rel=0)

## 摘要

新的 OpenShift 连接器 0.1.1 功能提供了三大优势:

*   通过在 VS 代码中支持完全集成的 OpenShift 开发和部署来加速 OpenShift 开发，这允许您连接到任何 OpenShift 集群，并从 IDE 本身创建、调试和部署。
*   简化云基础设施的内循环开发，因为该扩展使用了类似于 [OpenShift Do (odo)](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_developer_cli/understanding-odo.html) 和 [OpenShift CLI (oc)](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html) 的工具，前者是一种快速简单的 CLI 工具，用于在 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview)上创建应用程序，后者帮助您与本地或生产 OpenShift 实例进行交互，并完成内循环体验。
*   通过为使用 OpenShift 实例(3.x 或 4.x)提供简化的开发人员体验并支持公共云实例(如 Azure 和 AWS 上的 Red Hat OpenShift ),增强开发人员工作流程。

## 为什么要打开 Shift 连接器？

VS 代码上的 OpenShift 连接器提供了与 Red Hat OpenShift 集群交互的端到端开发人员体验。通过利用 IDE 特性，开发人员可以创建、部署和调试(即将推出的)应用程序，然后直接部署到正在运行的 OpenShift 集群。

在即将发布的版本中，连接器还将支持 NodeJS 和 Java 应用程序的调试。另外，这里看一下特性。

### 集群管理

说到集群管理，打开 VS 代码上的 Shift 连接器:

*   支持 3.x 和 4.x 集群的本地 OpenShift 集群管理。
*   支持远程 OCP 集群，如 Azure 上的 Red Hat OpenShift 和 AWS 上的 OpenShift。
*   支持使用令牌和用户名/密码登录集群。
*   提供直接从扩展视图在不同的 kubeconfig 条目之间切换上下文的功能。
*   允许您从 Kubernetes Explorer 视图中查看和管理 OpenShift 资源，如构建和部署配置。

### 发展

说到开发，VS 代码上的 OpenShift 连接器:

*   允许用户直接从 VS 代码连接到本地或托管的 OpenShift 集群。
*   提供简化的内循环体验，并快速更新集群上的更改。
*   在连接的群集上创建组件、服务和路由。
*   从扩展本身直接向组件添加存储。

### 部署

说到部署，打开 VS 代码上的 Shift 连接器:

*   提供一键部署，直接从 VS 代码中打开 Shift 集群。
*   允许用户导航到您创建的多条路线以访问已部署的应用程序。
*   直接在集群上部署多个互连的组件和服务。
*   直接推动和观察组件变化。
*   直接在 VS 代码的集成终端视图上流式记录日志。

### 监视

说到监控，打开 VS 代码上的 Shift 连接器:

*   允许您直接从 VS 代码中使用 Kubernetes 资源。
*   启动和恢复生成和部署配置。
*   查看和跟踪部署、单元和容器的日志。

## 安装延伸部分

您可以使用两种方法之一安装 OpenShift 连接器扩展。首先，您可以直接从 Visual Studio 代码市场安装插件。否则，您可以打开计算机的 Visual Studio 代码接口。搜索 OpenShift 连接器扩展，并使用 VS 代码中的**扩展**查看扩展图标进行安装。

## 入门指南

本入门指南提供了使用 OpenShift 时使用扩展和增强开发人员体验的工作流程。

### 在 OpenShift 集群中创建一个组件

用户可以登录任何本地或托管的 OpenShift 集群。为了在本地运行 OpenShift 集群的实例，开发人员可以使用 [CodeReady 容器](https://github.com/code-ready/crc)用于 OpenShift 4.x 集群，或者使用 [Minishift](https://docs.okd.io/latest/minishift/getting-started/index.html) / [Red Hat 容器开发工具包](https://developers.redhat.com/products/cdk/overview)用于 OpenShift 3.x 集群。

接下来，登录到您的服务器，并保存您的登录供以后使用。更具体地说，使用给定的凭证登录到给定的服务器，或者使用无记名令牌登录到 API 服务器进行身份验证。

要部署组件:

1.  从项目中创建一个新组件，并使用一个`git`存储库、二进制文件或本地工作区(目录)作为组件的源。
2.  当扩展提示您选择上下文文件夹并将其添加到工作区时，为每个组件分配一个上下文文件夹。

**注意:**所有组件配置都保存到`/.odo/config.yaml`。您可以将此文件提交到您的存储库中，以便以后使用相同的配置轻松地重新创建组件，或者与其他人共享它。

3.  要将组件部署到集群或应用配置更改，请单击组件中可用的 **Push** 操作。在树状视图中，组件处于三种状态之一:

在本例中，我们将了解如何创建基于 NodeJS 的组件，将其部署在 Red Hat OpenShift 4.2 集群上，并访问集群路由:

1.  右键**集群** - > **新建项目**如图 1:
    [![Create a new project.](img/cca063b512df940a715205400ba6e2b3.png "new-project")](/sites/default/files/blog/2019/10/Screenshot-2019-10-10-at-7.14.14-PM.png)*图 1:新建项目。*>
2.  选择如图 2 所示的应用程序(如果项目中没有应用程序，扩展工作流会提示用户创建一个新的应用程序):
    [![Select the application.](img/94e68a399a7b4a9078129f1ae067c152.png "new-app")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.04.47-PM.png)*图 2:选择应用程序。*>
3.  选择组件的源类型。有三种可用的源类型。对于这个例子，我们将使用基于`git`的工作流来创建组件，所以我们选择 **Git 库**如图 3 所示:
    [![Select the source type.](img/408bf30ec9538602f5d566944656df23.png "source-type")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.05.02-PM.png)*图 3:选择源类型。*>
4.  选择组件配置所在的 context 文件夹，如图 4 所示(下拉列表会显示工作区中所有可用的文件夹，如果用户需要选择任何其他文件夹，单击**添加新的 context 文件夹**，会出现一个对话框让您选择文件夹):
    [![Select context folder.](img/09186b569b5e4bc009d095db24998b89.png "context-folder")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.05.10-PM.png)*图 4:选择* context *文件夹。*>
5.  提供`git`仓库的 URL，如图 5 所示(本例使用`git`仓库[https://github.com/mohitsuman/nodejs-ex](https://github.com/mohitsuman/nodejs-ex)):
    [![](img/c7d6486793a6ec021f2275216ad29b00.png "git-url")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.05.34-PM-2.png)*图 5:添加`git`仓库。*>
6.  选择任何特定的分支或标签来获取组件的配置，如图 6 所示(该特性提供了直接在 OpenShift 实例上尝试和测试任何工作特性的灵活性):
    [![Select your git branch or tag.](img/0c594196d95552e7e84aed9db0c73d0d.png "git-branch-selection")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.05.45-PM.png)*图 6:选择您的`git`分支或标签。*>
7.  提供组件名称，如图 7:![Name the component.](img/726734d61bcb00f08cebd6b99ebf6254.png) *图 7:命名组件。*
8.  提供组件的类型，如图 8 所示(本例中我们选择**nodejs**):
    [![Select the component's type.](img/f915f1a1b805bdba22571f8e41919f07.png "component-type")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.40.34-PM.png)*图 8:选择组件的类型。*>
9.  提供组件的版本，如图 9 所示(本例中我们选择 NodeJS 的最新版本):
    [![Select the component type's version.](img/5dc9f19ce37a91ea1dcbb9b4ddd79b12.png "component-version")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.06.17-PM.png)*图 9:选择组件类型的版本。*>

一旦操作成功，组件就被创建并添加到本地配置中。组件的状态为“未推送”，并在集群的树视图中更新。在这个阶段，集群上没有推送任何东西，如图 10 所示:

[![This component has not yet been pushed.](img/b4d7385e4241eabd8686e16a2039a79a.png "not-pushed-state")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-12.06.46-PM.png)*Figure 10: This component has not yet been pushed.*">

10.  通过右键单击项目并单击上下文菜单中的 **Push** 将这些更改推送到集群，如图 11 所示:
    [![Push the changes to the cluster.](img/0ef5d088fcef1b1c8cfbcf4f975bdd1e.png "push-component")](/sites/default/files/blog/2019/10/Screenshot-2019-10-01-at-6.17.37-PM.png)*图 11:将更改推送到集群。*>

在集成终端视图中直接观察 push 命令的进度，如图 12 所示:

[![The push command's progress streamed in the integrated terminal view.](img/3a0a5febbc109cf8b8aea8c05f8a7a79.png "push-terminal")](/sites/default/files/blog/2019/10/Screenshot-2019-10-09-at-3.36.06-PM-e1571836436799.png)*Figure 12: The push command's progress streamed in the integrated terminal view.*">

一旦组件被成功地推送到集群，组件的状态就变为“pushed ”,如图 13 所示:

[![The component has now been pushed.](img/1ac19f91dcc7db7c76cff9aed0435bd1.png "pushed-component")](/sites/default/files/blog/2019/10/Screenshot-2019-10-09-at-3.44.06-PM.png)*Figure 13: The component has now been pushed.*">

### 创建并访问到集群的路由

让我们为组件创建一个路由，并通过选择**组件**->-**新 URL** 来公开与之关联的端口。如果组件支持多个端口，扩展会提示用户选择附加到所创建路由的端口。使用该命令生成的 URL 可用于从集群外部访问已部署的组件。

一旦创建了新的 URL，就必须将其推送到集群，以便将新的路由分配给集群。然后，您可以通过右键单击项目并选择**在浏览器**中打开来访问该路线，如图 14 所示:

[![Access your new route to the cluster.](img/7baa1cd7a889df6bb03eabea0a034c65.png "open-in-browser")](/sites/default/files/blog/2019/10/Screenshot-2019-10-09-at-4.17.09-PM.png)*Figure 14: Access your new route to the cluster.*">

如果有多条路线与该组件相关联，则会显示一条提示，让您选择所需的路线并继续操作。

### 向组件添加存储

执行以下操作，让用户创建存储并将其装载到组件:

1.  通过选择**组件** - > **新存储**，选择要附加存储的组件，如图 15:
    [![](img/4c5e2c452013911e9fc19d9484645ecb.png "storage-openshift-connector")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.17.33-PM-1.png)*图 15:向组件添加新存储。*>
2.  为这个存储提供名称，如图 16:
    [![Provide the name for this storage.](img/1f578e249765ef374997819e41e6ec0f.png "storage-name")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.17.51-PM.png)*图 16:为这个存储提供名称。*>
3.  指定存储挂载路径，如图 17:
    [![Provide the storage mount path.](img/4b2903ad8fd11cc993224a069337f11e.png "storage-path")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.18.07-PM.png)*图 17:提供存储挂载路径。*>
4.  从可用选项中选择存储大小，如图 18 所示。
    [![Select storage size.](img/eabdf378b84241b9250684c2e773100b.png "storage-size")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.18.15-PM.png)*图 18:选择存储大小。*>
5.  通过运行 Push 命令将此更改推送到集群，结果如图 19 所示。成功创建存储后，树视图会随存储节点一起更新。
    [![Pushing the storage addition to the cluster.](img/a8524303582de4163b40edb3b25b20c9.png "storage-tree-view")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.42.33-PM.png)*图 19:将存储添加到集群中。*>

### 查看日志

这个扩展添加了两个关于日志的上下文菜单选项。第一个是 **Follow Logs** ，它允许您跟踪来自部署的集成终端输出中的运行 pod 的日志流。第二个是**显示日志**，它允许用户查看特定资源的日志，如图 20 所示:

[![The Show Logs context menu option.](img/28da678c6cf5b788fc6e390fea5eb4ca.png "logs-display")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.02.39-PM.png)*Figure 20: The Show Logs context menu option.*">

### 取消部署组件

要从集群中取消部署一个组件并将其保留在本地配置中，请从组件的上下文菜单中运行 **Undeploy** 命令，如图 21 所示:

[![Undeploy a component.](img/d2aefb1c1579245228b42cbc6177cd83.png "openshift-connector-undeploy")](/sites/default/files/blog/2019/10/Screenshot-2019-10-09-at-4.31.06-PM-1.png)*Figure 21: Undeploy a component.*">

### 删除组件

要从集群和本地工作区删除组件，选择**组件**->-**删除**。该组件将从本地文件夹配置中删除，并且`.odo`文件将从上下文文件夹中删除，如图 22 所示:

[![Delete a component.](img/a378e472bd7d4e515ddbe387dfb35456.png "delete-component")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.10.29-PM.png)*Figure 22: Delete a component.*">

### 描述资源

用户可以描述一个特定的资源，如图 23 所示，包括名称、类型、源、环境变量和暴露的调试端口等信息:

[![Describe your resource.](img/e1e351987565d9ac88f51dad7b734ae6.png "describe")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.15.56-PM.png)*Figure 23: Describe your resource.*">

### 链接组件和服务

现实世界中的应用程序包含需要协同工作的多个组件和服务。使用此扩展，用户可以执行以下操作:

*   使用**组件**->-**链接组件**将多个组件链接在一起。例如，考虑将节点 JS 上的前端组件和 Java 上的后端组件链接在一起。
*   要将一个组件链接到一个服务，选择**组件**->-**链接服务**，如图 24 所示。例如，考虑将节点 JS 上的前端组件链接到 MongoDB 服务。

[![Link a service to a component.](img/97de2cf920ed3e3902e2639643430307.png "link-component")](/sites/default/files/blog/2019/10/Screenshot-2019-10-11-at-5.47.19-PM.png)*Figure 24: Link* a service *to a component.*">

### 查看 Kubernetes 资源

为了更容易地管理 Kubernetes 资源，用户可以使用 Kubernetes explorer(作为一个依赖项添加)，它可以从 VS 代码活动栏访问。这个浏览器允许您访问关于 Kubernetes 集群和资源的信息。您可以设置一个活动的 OpenShift 项目，流式传输和查看其日志，开始和停止构建，等等。以下是可用于与 OpenShift 资源交互的部分操作:

*   **切换 OpenShift 项目**:将指定项目设置为活动项目，并使用其特定资源。
*   **在控制台中打开 OpenShift 项目**:用户可以在正在运行的 OpenShift 控制台仪表盘中打开具体项目，如图 25:
    [![Open your OpenShift project in the console.](img/cf99dc7179743c79988682239021378d.png "use-project")](/sites/default/files/blog/2019/10/Screenshot-2019-10-20-at-7.21.02-PM.png)*图 25:在控制台中打开您的 OpenShift 项目。*>
*   **执行部署配置**中的动作:用户可以执行针对某个部署配置的各种动作，如**显示日志**、**部署**配置，以及**直接从资源管理器的控制台**中打开，如图 26:
    [![Use actions in the deployment config.](img/24c20b0a4720b7d860ba59bac71b5bb3.png "deploy-dc")](/sites/default/files/blog/2019/10/Screenshot-2019-10-20-at-7.30.27-PM.png)*图 26:使用部署配置中的动作。*>
*   **在构建配置**中执行操作:用户可以直接从资源管理器的构建配置中启动一个特定的构建(如图 27 所示)，并在控制台仪表板中查看它:
    [![Start a build in the explorer.](img/269500396d560a5c26207666cc9d9433.png "build-actions")](/sites/default/files/blog/2019/10/Screenshot-2019-10-20-at-7.50.39-PM.png)*图 27:在资源管理器中启动一个构建。*>

## 获得支持

如果您遇到任何错误、令人困惑的命令或不清楚的文档，或者如果您想要提出功能请求，您可以通过以下方式提交反馈:

*   从命令面板选项或扩展标题视图中选择 **OpenShift:报告扩展问题**。
*   将您的问题直接提交到 [GitHub](https://github.com/redhat-developer/vscode-openshift-tools/issues) 上。
*   在 [Gitter](https://gitter.im/redhat-developer/openshift-connector) 上与团队和社区讨论您的问题。

## 结论

尝试来自 VS 代码市场的最新版本的 [OpenShift 连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)。有几个可用的基础设施选项，包括笔记本电脑，它允许您使用 Red Hat CodeReady Containers 在本地[安装一个 OpenShift 集群。](http://developers.redhat.com/products/codeready-containers)

*Last updated: July 1, 2020*