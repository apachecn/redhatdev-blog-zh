# 开放数据中心的发展路线图

> 原文：<https://developers.redhat.com/blog/2020/06/22/a-development-roadmap-for-open-data-hub>

开放数据中心(ODH)是在 Red Hat 的基于 Kubernetes 的 OpenShift 4.x 平台上构建人工智能即服务(AIaaS)平台的蓝图。开放数据中心团队最近发布了[开放数据中心 0.6.0](https://developers.redhat.com/blog/2020/05/07/open-data-hub-0-6-brings-component-updates-and-kubeflow-architecture/) ，随后是[开放数据中心 0.6.1](https://developers.redhat.com/blog/2020/06/02/open-data-hub-0-6-1-bug-fix-release-to-smooth-out-redesign-regressions/) 的更小更新。

我们最近聚在一起讨论了下两个版本的计划和时间表。我们的计划基于我们在 4 月 6 日的[开放数据中心社区会议上整理并展示的](https://gitlab.com/opendatahub/opendatahub-community/-/wikis/Open-Data-Hub-Community-Meeting-Agenda#monday-april-6th-2020)[路线图幻灯片](https://gitlab.com/opendatahub/opendatahub-community/-/wikis/uploads/39b75e0eee79a059fffabbf0f38d77cf/ODH_Roadmap__draft_.pdf)。

在本文中，我们展示了未来几个开放数据中心版本的路线图。我们要强调的是，目标日期是乐观的，描述了我们希望实现的目标。随着世界的现状和假期时间的到来，这些日期可能会改变。

## 开放式数据中心 0.7:2020 年 6 月底

我们最接近的版本主要是关于 kube flow 1.0，并在 T2 红帽开放平台 T3 上发布。

### OpenShift 上的 Kubeflow 1.0

这项计划的主要目标是验证 Kubeflow 1.0 在 Red Hat OpenShift 上的工作情况，并修复我们发现的问题。另一个目标是记录并理想地自动化一些验证过程，以开始在 OpenShift 上为 Kubeflow 启用[持续集成](https://developers.redhat.com/topics/ci-cd/) (CI)。

**注**:由于项目比较新，变化比较频繁，我们在 OpenShift 上启用 [Kubeflow 0.7 的同时，禁用了](https://developers.redhat.com/blog/2020/02/10/installing-kubeflow-v0-7-on-openshift-4-2/) [KFServing](https://www.kubeflow.org/docs/components/serving/kfserving/) 。作为这方面工作的一部分，我们将继续调查 KFServing 的状态和任何潜在的问题。

### 改进开放式数据中心 CI

基于 OpenShift CI 的持续集成(CI)正在为[odh-manifest 库](https://github.com/opendatahub-io/odh-manifests/)运行，但是测试集很少。这个计划的目标是扩展和改进所有组件的测试。我们希望能够验证容器不仅已经启动，而且组件确实在工作——这意味着我们计划增加一些功能测试。

这个项目的另一个方面是为其他开放数据中心库启用 CI 组件，主要是 [opendatahub-operator](https://github.com/opendatahub-io/opendatahub-operator) 。

### 开始混合开放数据中心和 Kubeflow 组件

自从基于 Kubeflow 的开放数据中心项目开始以来，我们就计划能够混合开放数据中心和 Kubeflow 的组件。目前，这是可能的，但不能保证这些组件能很好地一起运行和工作。

我们已经发现了在自定义名称空间中运行 [Kubeflow 的问题。Kubeflow 也严重依赖于](https://github.com/kubeflow/manifests/issues/1022) [Istio](https://developers.redhat.com/topics/service-mesh/) ，这还不是开放数据中心的情况。对于这个项目，我们将选择一个 Kubeflow 组件，并在它作为 Open Data Hub 的一部分运行时开始测试它。(第一候选人是培训作业操作员之一: [TF 作业](https://www.kubeflow.org/docs/components/training/tftraining/)或 [Pytorch 作业](https://www.kubeflow.org/docs/components/training/pytorch/))。

### 添加对象存储组件

开放式数据中心依赖于亚马逊简单存储服务(亚马逊 S3)兼容的对象存储。目前，我们还没有一个简单的方法来为我们的用户提供这一点，所以我们计划添加一个组件来实现 S3 兼容的对象存储。这些组件可以是[露天集装箱仓库](https://www.redhat.com/en/technologies/cloud-computing/openshift-container-storage) (OCS)或[露天集装箱仓库](https://rook.io/)。

因为 OCS 的默认安装非常耗费资源，所以我们将与团队合作，帮助我们进行一个较小的(非生产)安装，它可以用于开发、测试、研讨会和其他用例。

### 将数据目录转换为自定义

[数据目录](https://opendatahub.io/news/2019-12-15/data-catalog-in-odh.html)是最新开放数据中心版本中缺少的最后一个组件。我们想转换它，并作为下一个版本的一部分提供它。

## 开放式数据中心 0.8:2020 年 8 月底

对于夏季版本，我们专注于自动化，并确保开放数据中心和 Kubeflow 能够很好地协同工作。

### 持续部署

我们将继续改进开放数据中心的 CI，但故事的第二部分是持续部署(CD)。这项计划有两个目标:在 Red Hat 内部运行的内部数据中心和部署在 [Mass Open Cloud](http://massopen.cloud/) 中的开放数据中心实例。

想法是在这两个目标上自动部署新版本的 Open Data Hub 到 staging。因为 Open Data Hub 至少由三个主要部分组成——操作者、Open Data Hub 清单和 Kubeflow 清单——我们需要研究在 Open Dat Hub 上持续部署的真正含义。我们的目标是提出一个计划，说明自动化的内容以及在部署方面如何实现。

### 开放式数据中心的通用基础映像(UBI)

Red Hat 在开发[Red Hat Universal Base Image](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)(UBI)方面做得非常好，我们希望尽可能地利用这一点。该计划的目标是验证我们的所有组件都运行在 UBI 上，并与上游社区合作，如果不是这样的话。

### 继续混合开放数据中心和 Kubeflow 组件

假设我们在 Open Data Hub 0.7 中混合组件的实验进展顺利，我们将继续测试、修复和验证更多这些组件。我们将在开始下一组 Open Data Hub 0.8 之前完成优先组件列表的工作

## 开放式数据中心 0.9:2020 年秋季

这是目前我们粗略估计的最远版本。在此版本中，我们将重点关注开放式数据中心的企业需求。

### 断开部署

开放式数据中心和 Kubeflow 的离线部署是一个大话题——特别是在金融领域和边缘的人工智能用例中。Kubeflow 社区在过去已经尝试解决这个问题。我们将需要看看我们是否可以在这项工作的基础上再接再厉，或者我们是否应该从头开始。

### 基于 UBI 的 Kubeflow

随着我们对基于 UBI 的开放数据中心的推动，我们想看看 Kubeflow 组件和映像，并找到一种在 UBI 上移植和维护它们的方法。再现性和自动化是这个项目长期成功的关键。

## 遵循路线图！

作为创建新路线图的一部分，我们还在 opendatahub.io 上重新设计了[开放数据中心路线图。我们将不断更新本文档，以便您可以随时了解我们在开放数据中心的当前和未来计划中所处的位置。](http://opendatahub.io/docs/roadmap/future.html)

*Last updated: January 18, 2022*