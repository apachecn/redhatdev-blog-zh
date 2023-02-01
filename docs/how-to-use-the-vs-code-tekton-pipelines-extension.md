# 如何使用 VS 代码 Tekton 管道扩展

> 原文：<https://developers.redhat.com/blog/2020/01/14/how-to-use-the-vs-code-tekton-pipelines-extension>

从 [Knative 项目](https://knative.dev/)分支后，Tekton 项目于 3 月宣布，作为 [Kubernetes-](https://developers.redhat.com/topics/kubernetes/) 本地 CI/CD 管道工具，它正在引起人们的兴奋。

Tekton 提供了 Kubernetes 所享有的灵活性和不可知论，并被定位为第一个用于执行流水线的开放标准化引擎。尽管该项目仍处于开发的早期阶段，但我们已经迫不及待地开始让开发人员更容易地跳上 Tekton 列车。在本文中，我们将快速了解 Tekton 管道扩展以及如何使用它。

Red Hat 最初发布的 [Visual Studio (VS)代码 Tekton Pipelines 扩展](https://github.com/redhat-developer/vscode-tekton)是由[林赛·图洛赫](https://github.com/onyiny-ang)在暑期实习期间开发的，并得到了 [OpenShift 连接器扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)、[丹尼斯·戈洛文](https://github.com/dgolovin)和[莫希特·苏曼](https://github.com/mohitsuman)以及 Red Hat Pipelines 团队的支持，特别是[文森特·德梅斯特](https://github.com/vdemeester)和[苏尼尔·塔哈。](https://github.com/sthaha)

Tekton Pipelines 扩展提供了与 Tekton CLI 工具相同的所有功能以及管道视图。这不仅允许开发人员可视化他们正在开发的管道部署，还允许与管道资源进行直观的交互。

今天，Tekton Pipelines 扩展可以在 [VSCode Marketplace](https://marketplace.visualstudio.com/vscode) 上获得，并且可以与最新版本的 VSCode 一起使用。

**支持的功能:**

*   管道资源的片段
*   管道视图
*   通过单击所需的管道资源实施 CLI 功能

## 使用 Tekton 管道扩展

安装 Tekton Pipelines 扩展将触发安装 Kubernetes 扩展和最新版本的 Tekton CLI 工具`tkn`。一旦安装了这些，您将在 VSCode explorer 中看到一个“Tekton Pipelines”视图，其中的`Pipelines`、`Tasks`、`ClusterTasks`和`PipelineResource`作为顶级树节点。单击这些节点中的任何一个都会展开视图，显示嵌套的资源。

*   `Pipelines`显示:`Pipelines > PipelineRuns > TaskRuns`。
*   `Tasks`显示:`Tasks > TaskRuns`。
*   `ClusterTasks`显示:`ClusterTasks`。
*   `PipelineResources`显示:`PipelineResources`。

单击这些资源中的任何一个都会显示一系列操作，与`tkn` CLI 工具的功能相匹配。

#### Tekton 管道/任务/集群任务可用的操作

*   `Pipeline -> Start` —使用用户选择的资源、参数和服务帐户启动管道。
*   `Pipeline -> Restart` —重新开始最后一次管线运行。
*   `Pipeline/Task/ClusterTask -> List` —列出群集中的所有管线。
*   `Pipeline -> Describe` —打印所选管道的 JSON。
*   `Pipeline/Task/ClusterTask -> Delete`-删除所选管线。

#### Tekton PipelineRun 可用的操作

*   `PipelineRun/TaskRun -> List` —列出管道/任务中的所有管道运行/任务运行。
*   `PipelineRun/TaskRun -> Describe` —描述所选的管道运行/任务运行。
*   `PipelineRun/TaskRun -> Logs` —打印所选管道运行/任务运行的日志。
*   `PipelineRun/TaskRun -> Delete` —删除选定的管道运行/任务运行。
*   `PipelineRun -> Cancel` —取消所选的管线运行。

Tekton 项目仍处于开发的早期阶段，因此在扩建部分有足够的扩展和改进空间。当然，GitHub repo 随时欢迎您的建议和请求。

*Last updated: May 26, 2022*