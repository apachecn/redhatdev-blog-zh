# Kubeflow 1.0 监控和增强的 JupyterHub 内置于开放式数据中心 0.8 中

> 原文：<https://developers.redhat.com/blog/2020/09/18/kubeflow-1-0-monitoring-and-enhanced-jupyterhub-builds-in-open-data-hub-0-8>

新的 [Open Data Hub 版本 0.8](https://github.com/opendatahub-io) (ODH)包括许多新功能、持续集成(CI)新增功能和文档更新。对于这个版本，我们专注于增强 JupyterHub 映像构建，实现开放数据中心和 Kubeflow 组件的更多[混合](https://opendatahub.io/docs/kubeflow/mixing.html)，并设计我们全面的端到端持续集成和持续部署和交付(CI/CD)流程。在本文中，我们将介绍这个最新版本的亮点。

**注** : Open Data Hub 是一个开源项目，是一个社区[运营商](https://developers.redhat.com/topics/kubernetes/operators/)，用于在[红帽 OpenShift](https://developers.redhat.com/products/openshift/overview) 上搭建 AI 即服务(AIaaS)平台。

## JupyterHub

为了简化对 JupyterHub 的代码更改，我们在 [opendatahub-io](https://opendatahub.io/) GitHub 项目下分叉了两个 pivotal 存储库: [jupyterhub-quickstart](https://github.com/opendatahub-io/jupyterhub-quickstart) 和 [jupyterhub-odh](https://github.com/opendatahub-io/jupyterhub-odh) 。今后，JupyterHub 的所有代码更改和特性增强都将归入这两个新的存储库。

我们还简化了 JupyterHub 使用的所有图像，将它们从不同的私有和公共存储库中拖到 Quay.io 上的两个存储库中: [odh-jupyterhub](https://quay.io/organization/odh-jupyterhub) 和 [thoth-station](https://quay.io/organization/thoth-station) 。

笔记本映像现在由 [Thoth Station](https://github.com/thoth-station) 构建和维护，这是一个人工智能(AI)工具，为人工智能应用程序分析和推荐软件栈。关于透特以及如何使用它的更全面的信息，请访问[透特站](https://thoth-station.ninja/)。

我们还用 Elyra 更新了 Open Data Hub 0.8，这是一个人工智能工具包，可以让你启动 JupyterLab 图像。

## 从笔记本到 Elyra 管道

为了让数据科学家能够将他们的笔记本变成 Argo 工作流或 Kubeflow 管道，我们添加了一个令人兴奋的新工具，名为 [Elyra](https://github.com/elyra-ai) 以打开 Data Hub 0.8。将数据科学家在笔记本电脑上创建的所有工作转换为生产级流水线的过程非常繁琐，通常需要手动操作。Elyra 允许您从 JupyterLab 门户执行这个过程，只需点击几下鼠标。如图 1 所示，Elyra 现在包含在一个 JupyterHub 笔记本映像中。

[![A screenshot of the JupyterLab image dialog.](img/ddc5e0f0b8eaef165b55bf98eb01162d.png "Screen Shot 2020-09-01 at 8.41.15 AM")](/sites/default/files/blog/2020/09/Screen-Shot-2020-09-01-at-8.41.15-AM.png)Figure 1: Elyra is provided in a JupyterHub image.">

该映像使用 Elyra 工具运行一个 JupyterLab 环境，如图 2 所示。

[![A screenshot of the JupyterLab environment setup page with the Elyra tools option.](img/8551ece70f3b0532617bbfe28f1fded6.png "Screen Shot 2020-09-01 at 8.43.04 AM")](/sites/default/files/blog/2020/09/Screen-Shot-2020-09-01-at-8.43.04-AM.png)Figure 2: A JupyterLab environment with the Elyra tools option.">

寻找关于如何优化使用 Elyra 的文章和文档。

## 监控库伯流

作为我们努力使 Kubeflow 和开放数据中心组件可互换的一部分，我们已经为 Kubeflow 添加了监控功能。使用 ODH 0.8，用户可以添加 Prometheus 和 Grafana 进行 Kubeflow 组件监控。目前，并非所有 Kubeflow 组件都支持 Prometheus 端点。我们确实打开了 Argo 中的 Prometheus 端点，并提供了如图 3 所示的示例仪表板，让用户可以监控他们的管道。

[![A screenshot of the Grafana dashboard monitoring an Argo workflow.](img/4fbf062448ab5def1e9aeb32d8fc9c22.png "Screen Shot 2020-09-01 at 1.50.12 PM")](/sites/default/files/blog/2020/09/Screen-Shot-2020-09-01-at-1.50.12-PM.png)Figure 3: Monitoring the Argo workflow in a Grafana dashboard.">

两个可用于监控的 Kubeflow 组件是`tfjobs`和`pytorchjobs`。要安装带监控的 Kubeflow，请使用[kube flow-监控示例 Kfdef](https://github.com/opendatahub-io/odh-manifests/blob/master/kfdef/kfctl_openshift_kubeflow_monitoring.yaml) 。

## 分布式机器学习

作为我们之前为 ODH 提供更多分布式学习工具的努力的延续， [PyTorch 操作符](https://github.com/kubeflow/pytorch-operator)现在使用 ODH 组件。要使用 ODH 安装 PyTorch 操作器，请使用[分布式培训示例 Kfdef](https://github.com/opendatahub-io/odh-manifests/blob/master/kfdef/kfctl_openshift_distributed_training.yaml) 。该示例还包括了在 ODH 0.7 中移植的`tfjobs` Kubeflow 组件。有关混合组件的更多信息，请参见 ODH 文档[。](http://opendatahub.io/docs/kubeflow/mixing.html)

## CI/CD

我们添加了更多测试来开放数据中心组件，包括 [Apache Kafka](https://developers.redhat.com/topics/kafka-kubernetes) 和 Superset，并且我们通过添加用于 web 门户测试的 Selenium 来增强 JupyterHub 测试。我们还增加了在 CI 管道中分叉的 [Kubeflow 清单库](https://github.com/opendatahub-io/manifests/)中运行这些测试的能力。然而，目前测试只验证清单。我们需要扩展它们以包含更全面的测试，比如我们为 [odh-manifest](https://github.com/opendatahub-io/odh-manifests) 库开发的测试。该团队还在继续研究一个完整的动态 CI/CD 管道的持续部署和交付系统。

## 结论

访问 [opendatahub.io](https://opendatahub.io/) 获取完整的 Open Data Hub 0.8 文档。您将在每个组件的 GitHub 部分找到关于特定组件的详细信息。我们还添加了[新的指导方针，通过容器针对现有集群在本地开发和运行测试套件](https://github.com/opendatahub-io/odh-manifests/tree/master/tests)，这使得开发更加容易。这个方法反映了我们的 CI 系统中实际运行的内容，因此它是一个更有用的调试方法。

*Last updated: September 23, 2020*