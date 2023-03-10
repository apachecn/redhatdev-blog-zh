# 使用 OpenShift 4.4 的新管道生成器和 Tekton 管道创建管道

> 原文：<https://developers.redhat.com/blog/2020/04/30/creating-pipelines-with-openshift-4-4s-new-pipeline-builder-and-tekton-pipelines>

Tekton 最初是 Knative 项目的一部分，但最终成为自己的项目。它已经存在一年多了，并且正在成为以 Kubernetes 特有的方式建立连续输送管道的事实上的标准。

OpenShift 管道建立在 Tekton 之上，并且可以从 OperatorHub 为您的 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 集群提供。 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview) 4.4 刚刚通过 Pipeline Builder 将管道创建向前推进了一步。作为 web 控制台开发人员视角的一部分，Pipeline Builder 使软件开发人员更容易创建自己的管道。

想看看这一切是如何运作的吗？太好了，我们开始吧。

## 步骤、任务、管道和资源

Tekton 的目标是创建可重用、可组合和声明性的小构建块，所有这些都在云原生环境中实现。它使用步骤、任务、管道和资源来完成这项工作，如图 1 所示。

[![Diagram showing the building blocks for a Tekton Pipeline](img/fe44497fa6d397cec091e7ce5b4fe61f.png "F1-PipelinesArchitecture")](/sites/default/files/blog/2020/04/F1-PipelinesArchitecture.png)Figure 1: Tekton building blocks.

Figure 1: Tekton building blocks.

### 步伐

*步骤*是对输入执行的操作。如果您正在构建 Node.js 应用程序，它可能会运行一些单元测试，或者验证代码是否遵循您已有的编码标准。Tekton 将在您提供的容器中执行每个步骤。对于 Node.js 应用程序，您可以运行安装了 Node.js 的 Red Hat 通用基础映像。台阶是泰克顿最基本的单位。

### 任务

*任务*是顺序执行的步骤列表。Tekton 在 Kubernetes pod 中运行任务，这允许您在设计一系列相关步骤时拥有一个共享环境。例如，您可以在任务中装入一个卷，该卷将在该特定任务的每个步骤中共享。

### 管道

*管道*是可以并行或顺序执行的任务的集合。Tekton 在如何以及何时执行这些任务方面为开发人员提供了很大的灵活性。您甚至可以指定任务必须满足的条件，以便开始下一个任务。

### 资源

大多数管道需要输入来执行任务，也可以产生输出。在 Tekton，这些被称为*资源*。资源可以有许多不同的类型，比如 Git 存储库、容器映像或 Kubernetes 集群。

## 管道建造商

现在我们已经介绍了 Tekton 的一些基本细节，让我们深入到 Pipeline Builder，它可以在开发人员透视图的 Pipelines 页面上找到。良好的界面对于帮助软件开发人员轻松构建自己的管道至关重要，因此 Pipeline Builder 的界面为开发人员提供了管道的可视化表示，可以轻松修改以满足您的需求。如图 2 所示，任务可以按顺序或并行排列。

[![Animation showing how to define task structure](img/3608f8f2121b5ffccc83c1da5d36f305.png "F2")](/sites/default/files/blog/2020/04/F2.gif)Figure 2: Defining the task structure in the Pipeline Builder.">

侧面板还可以轻松编辑每个任务的设置。从这里，您可以添加参数或者更改关联的资源，一旦您启动管道，这些资源将被映射到资源，如图 3 所示。

[![animation showing how to edit individual task settings](img/40e2b4d7b21838e65a9347b95c0e6d26.png "F3")](/sites/default/files/blog/2020/04/F3.gif)Figure 3: Edit individual task settings.">

## 管道线路

一旦创建了管道，最后一步就是执行它们。流水线的执行被称为 *PipelineRun* 。OpenShift 管道使得触发这些管道变得容易。在管道列表中找到想要启动的管道，并选择**启动**。将出现一个对话框，您可以在其中指定要用于该运行的资源，这将创建管道运行并开始执行。

然后，系统会提示您本次运行要使用的资源。该选择将创建`PipelineRun`并开始执行。然后，您可以从 Pipelines 仪表板跟踪此运行的状态，或者通过深入到一个任务来查看每个步骤的详细信息。

![](img/3609da575b965e8148c191ae5bd64bb6.png)

图 4:管道运行的可视化。

这种可视化表示也使开发人员更容易看到部署失败的地方。在下面的动画中，您可以看到管道(PipelineRun)的执行及其失败的位置。这也阻止了这些代码在生产中的部署。通过查看输出日志，软件开发人员可以很容易地诊断并找到导致管道失败的原因。

![](img/4f722d905f64dfad806e97fec8fb6acd.png)

图 5:从 PipelineRun 可视化中访问故障信息。

所以，我们仅仅触及了 OpenShift 管道所能实现的表面。我们希望这个新的界面和管道构建器将使软件工程师更容易构建和维护他们的 CI/CD 部署管道，让他们专注于他们真正关心的事情。

在未来的版本中，我们将继续增强这方面的功能。请留意添加触发器、工作区支持等功能！

## 准备好开始了吗？

今天就试试 OpenShift。

## 提供您的反馈

[在此分享您的想法](https://forms.gle/6HArjszuqyE1xr3f8)并向我们提供您的反馈或改进管道体验的想法。加入我们的 [OpenShift 开发者体验谷歌小组](https://groups.google.com/forum/#!forum/openshift-dev-users)，参与讨论并了解我们的办公时间会议，在这里您可以与我们合作并提供反馈。

## 了解更多信息

有兴趣[了解更多关于 OpenShift 应用程序开发的](https://developers.redhat.com/blog/2020/04/30/whats-new-in-the-openshift-4-4-web-console-developer-experience/)信息吗？以下是一些可能有帮助的红帽资源: [OpenShift 管道](https://www.openshift.com/learn/topics/pipelines)和[open shift](https://developers.redhat.com/openshift)上的应用开发。

*Last updated: June 29, 2020*