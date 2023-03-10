# Visual Studio 代码的新 Tekton 管道扩展

> 原文：<https://developers.redhat.com/blog/2020/01/08/the-new-tekton-pipelines-extension-for-visual-studio-code>

从 [Knative 项目](https://knative.dev/)分支后，Tekton 项目于 3 月宣布，作为 [Kubernetes-](https://developers.redhat.com/topics/kubernetes/) 本地 CI/CD 管道工具，它正在引起人们的兴奋。

它提供了 Kubernetes 所享有的灵活性和不可知论，并被定位为第一个用于执行流水线的开放标准化引擎。尽管该项目仍处于开发的早期阶段，但我们已经迫不及待地开始让开发人员更容易地跳上 Tekton 列车。因此，在本文中，我们将快速了解 Tekton 管道扩展以及如何使用它。

Red Hat 最初发布的 [Visual Studio (VS)代码 Tekton Pipelines 扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-tekton-pipelines)是由[林赛·图洛赫](https://github.com/onyiny-ang)在暑期实习期间开发的，并得到了 [OpenShift 连接器扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)、[丹尼斯·戈洛文](https://github.com/dgolovin)和[莫希特·苏曼](https://github.com/mohitsuman)以及 Red Hat Pipelines 团队的支持，特别是[文森特·德梅斯特](https://github.com/vdemeester)和[苏尼尔·塔哈。](https://github.com/sthaha)

该扩展提供了与 Tekton CLI 工具相同的所有功能以及管道视图。这不仅允许开发人员可视化他们正在开发的管道部署，还允许与管道资源进行直观的交互。

如今，该扩展可在 [VSCode Marketplace](https://marketplace.visualstudio.com/vscode) 上获得，并可与最新版本的 VSCode 配合使用。

**支持的功能:**

*   管道资源的片段
*   管道视图
*   通过单击所需的管道资源实施 CLI 功能
*   与微软的 Kubernetes 扩展集成

## 使用 Tekton 管道扩展

安装 Tekton Pipelines 扩展将触发安装 [Kubernetes 扩展](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)和最新版本的 Tekton CLI 工具`tkn`。一旦安装了这些，您将在 VSCode explorer 中看到一个“Tekton Pipelines”视图，其中的`Pipelines`、`Tasks`、`ClusterTasks`和`PipelineResource`作为顶级树节点。

![](img/6052a42ad5a08c09cea56a10921e662f.png)

单击这些节点中的任何一个都会展开视图，显示嵌套的资源。

*   `Pipelines`显示:`Pipelines > PipelineRuns > TaskRuns`。
*   `Tasks`显示:`Tasks > TaskRuns`。
*   `ClusterTasks`显示:`ClusterTasks`。
*   `PipelineResources`显示:`PipelineResources`。

这些资源的上下文菜单显示一系列操作，与`tkn` CLI 工具的功能相匹配。

#### 可用操作:

##### 管道

*   `Start Last Run` —重新开始管道的最后一次管道运行。
*   `Start` —使用用户选择的资源和参数启动管道。
*   `Describe` —描述管道。
*   `List PipelineRuns` —列出管道的所有管线走向。
*   `Delete`-删除一条管线。

##### 工作

*   `Start` —使用用户选择的资源和参数启动任务。
*   `List TaskRuns` —在终端中列出一个任务的所有任务运行。
*   `Delete` —删除任务。

##### clustartask

*   `Start` —使用用户选择的资源和参数启动集群任务。
*   `List TaskRuns` —在终端中列出一个集群任务的所有任务运行。
*   `Delete` —删除集群任务。

##### 管道运行

*   `Describe` —描述一条管线
*   `Show Logs` —显示管道的日志。
*   `List TaskRuns` —列出终端中当前名称空间的所有任务运行。
*   `Cancel` —取消管道运行。
*   `Delete` —删除一个管道管路。

##### 任务运行

*   `Show Logs` —显示任务运行的日志。
*   `Delete` —删除任务运行。

##### 资源

*   `Describe` —描述资源。
*   `Delete` —删除资源。

该扩展定义了代码片段，使得在 yaml 文本编辑器中创建 Tekton 资源更加容易。

![](img/4e890476447908638c908783789e0987.png)

#### 可用片段:

*   clustartask
*   管道或任务参数
*   管道资源
*   管道任务参考
*   管道任务引用输入、参数和输出
*   管道资源资源
*   管道资源类型
*   管道运行资源
*   Kubernetes 资源限制和请求
*   Tekton 任务资源
*   Tekton 任务输入、参数和输出
*   流水线任务参数
*   Tekton 任务运行资源
*   Tekton 任务步骤

#### 与 Kubernetes 集群视图集成

该扩展将`Tekton Piplines`节点添加到 Kubernetes 集群视图中。它为您提供了一种在活动的 Kubernetes 名称空间中访问原始 Tekton 资源的方法。

![](img/e586a5c660977d3f74d06b8b9730b97b.png)

Tekton 项目仍处于开发的早期阶段，因此在扩建部分有足够的扩展和改进空间。当然，在 GitHub 知识库中，问题和请求形式的建议总是受欢迎的，可以在这里找到。

*Last updated: April 7, 2022*