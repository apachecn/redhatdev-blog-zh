# Open Data Hub 0.6 带来了组件更新和 Kubeflow 架构

> 原文：<https://developers.redhat.com/blog/2020/05/07/open-data-hub-0-6-brings-component-updates-and-kubeflow-architecture>

开放数据中心(ODH)是在 Red Hat 的基于 Kubernetes 的 OpenShift 4.x 的基础上构建人工智能即服务平台的蓝图。Open Data Hub 版对整体架构进行了重大更改，并对组件进行了更新和添加。在本文中，我们将探讨这些变化。

## 从可操作符到自定义

如果你密切关注[开放数据中心](https://opendatahub.io)项目，你可能会意识到我们已经为一个重大的设计变更工作了几个星期。由于我们开始与 [Kubeflow 社区](https://www.kubeflow.org/)密切合作，让 [Kubeflow 在 OpenShift](https://www.kubeflow.org/docs/openshift/) 上运行，我们决定利用 Kubeflow 作为开放数据中心的上游，并采用其部署工具——即 KFdef manifests 和[Kustomize](https://kustomize.io/)——进行部署清单定制。

我们仍然相信 Ansible Operator 为构建运营商提供了一个很好的框架，但同时，我们相信我们需要靠近我们的上游(Kubeflow)，这就是为什么在部署和生命周期管理方面与项目保持一致是非常有意义的。为此，我们分析了所有的组件，并开始将它们从可转换的角色改造成 Kustomize 兼容的结构。

你可以在`odh-manifest repository` 中找到[新的清单。它紧密遵循 Kubeflow 清单库](https://github.com/opendatahub-io/odh-manifests)的结构，以确保项目的兼容性。

## 更新的组件

作为此次重写的一部分，一些组件也进行了更新。我们决定从版本 0.6.0 开始，将来依赖 OpenShift 4x。这一决定使我们能够充分利用[运营商生命周期管理器](https://docs.openshift.com/container-platform/4.4/operators/understanding_olm/olm-understanding-olm.html) (OLM)，并通过在 OLM 目录中提供他们的项目来避免重复其他团队已经经历的维护负担。

从技术角度来看，这意味着我们并不持有我们部署和管理的所有组件的部署清单，而是通过 OLM 经由订阅来部署操作员，并且仅通过特定的定制资源来指示操作员。通过 OLM 部署的组件有[斯特里姆兹](https://strimzi.io/)、[普罗米修斯](https://prometheus.io/)和[格拉夫纳](https://grafana.com/)。

让我们看看其他组件。

### 火花

另一个收到更新的组件是[radanalytics . io Spark Operator](https://github.com/radanalyticsio/spark-operator/)。我们现在使用`SparkCluster`定制资源而不是`[ConfigMaps](https://github.com/radanalyticsio/spark-operator#config-map-approach)`，并且我们将我们的 Spark 依赖项更新到 Spark 2.4.5。

### 南船星座

我们原本想通过 Kubeflow 项目直接使用 Argo，但是 Kubeflow 搭载的是 Argo 2.3，对于开放数据中枢用户来说太老了。因为我们必须将组件转换为 Kustomize，所以我们决定也将其更新到最新的稳定版本，所以 ODH 现在提供了 [Argo 2.7](https://blog.argoproj.io/argo-workflows-v2-7-6ace8c210798) 。

### JupyterHub

JupyterHub 的变化将以更新库 [JupyterHub 单用户配置文件](https://github.com/vpavlin/jupyterhub-singleuser-profiles/)的形式出现，我们使用该库来定制 Jupyter 服务器部署以及 JupyterHub 与 Spark 等其他服务之间的集成。这些变化主要涉及到重构一些配置结构，使之更加 Kubernetes 本地化，以及改进我们与 Spark 操作符的集成。

### 超集

上面的更新组件列表不包括超集，但是您不必担心——超集仍然是 Open Data Hub 的一部分。部署根本没变，改成了 Kustomize。

### av 库

最后一个没有提到的组件是 AI 库。你可以在资源库中看到它，但我们没有正式将其包含在 0.6 版本中，因为我们无法将 Seldon 放入，这是该库的一个依赖项。我们将确保 AI 库和 Seldon 都是下一个版本的一部分。

## 开放式数据中心增加了气流

很长一段时间以来，我们一直听到对开放数据中心的[气流](https://airflow.apache.org/)的要求。因为我们已经有一个工作流管理系统(Argo)在 ODH 中可用，我们想确保我们不是简单地复制组件。事实证明，许多开放数据中心用户也是 Airflow 用户，让 ODH 部署 Airflow 会为他们增加很多价值。

我们的团队研究了在开放数据中心的组件之间增加气流的选项，并决定使用[气流操作器](https://github.com/opendatahub-io/airflow-on-k8s-operator)。我们还提供了一个使用芹菜部署气流的[示例气流集群](https://github.com/opendatahub-io/odh-manifests/tree/master/airflow/example-celery/base)定义，用户可以轻松启用和尝试。

## 安装开放式数据中心 0.6

与之前所有版本的 Open Data Hub 一样，我们非常关心用户在安装过程中的体验。这就是我们重组运营商目录条目(图 1)以提供两个渠道的原因——beta 和 legacy(图 2)。

[![Open Data Hub 0.6 OperatorHub entry](img/5e740852d1703462381a78cc5f74a4b3.png "Screenshot from 2020-05-04 13-54-11")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-04-13-54-11.png)

Figure 1: The Open Data Hub 0.6 OperatorHub entry.

[![Open Data Hub 0.6 channels](img/fb2d3332a1ec16e42c0fad7672c89855.png "Screenshot from 2020-05-04 13-54-39")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-04-13-54-39.png)

Figure 2: The channels in Open Data Hub 0.6.

### 贝塔频道

beta 通道提供了从 ODH 0.6.0 开始的新版本的 Open Data Hub。现在基于 KFDef 的定制资源已经随着实现发生了显著的变化，所以不要对新的例子感到惊讶。

### 传统频道

传统渠道仍然提供 ODH 0.5.x 选项。在发布的时候，它是 0.5.1，但是我们将尽最大努力保持错误修复流入 0.5 版本，因为我们知道一些用户需要暂时停留在那个版本，因为 0.6 只在 OCP 4 上工作。不过，我们不会在 ODH 0.5 中添加新的特性或组件。

### OpenShift 上的 Kubeflow

要提到的一个重要特性是，由于我们使用与 Kubeflow 相同的工具，所以您可以使用 Open Data Hub Operator 0.6 在 OpenShift 上部署 Kubeflow。操作员只支持 KFDef v1，它比 Kubeflow 0.7 包含的内容更新，所以我们在[我们的 Kubeflow 清单库](https://github.com/opendatahub-io/manifests)中为您准备了一个更新的自定义资源。

目前还不能同时部署和使用 Kubeflow 和 Open Data Hub，但这一功能将在即将发布的版本中提供。

## 社区

这一部分一点也没变，我们希望得到您的反馈。我们完全理解 ODH 0.6 带来了重大的变化，如果您遇到任何问题或有任何建议，让我们知道这一点对我们来说更加重要。

因为我们采用 Kubeflow 作为我们的上游，而且那个社区住在 GitHub 上，所以我们也要搬到那里去。我们使用 [GitHub 项目](https://github.com/orgs/opendatahub-io/projects)开始了我们对未来 sprints 的规划，我们也将迭代地将大部分源代码和文档转移到 GitHub，所以如果项目在链接和指针方面感觉有点混乱，请耐心等待——我们正在努力。

无论如何，不要犹豫加入我们的[社区会议](https://gitlab.com/opendatahub/opendatahub-community/-/wikis/Open-Data-Hub-Community-Meeting-Agenda)、[邮件列表](https://lists.opendatahub.io/admin/lists/)，或者通过 [GitHub 问题](https://github.com/opendatahub-io/odh-manifests/issues)与我们联系。

*Last updated: June 20, 2022*