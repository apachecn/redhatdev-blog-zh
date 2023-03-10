# 开放数据中心 0.6.1:发布错误修复以消除重新设计的回归

> 原文：<https://developers.redhat.com/blog/2020/06/02/open-data-hub-0-6-1-bug-fix-release-to-smooth-out-redesign-regressions>

自从我们[发布开放数据中心(ODH) 0.6.0](https://developers.redhat.com/blog/2020/05/07/open-data-hub-0-6-brings-component-updates-and-kubeflow-architecture/) 以来，仅仅过了短短几周时间，它为底层架构带来了许多变化和一些新功能。我们发现[在这个新版本中有几个关于 Kubeflow 操作符的问题](https://github.com/orgs/opendatahub-io/projects/3)和几个随着新的 JupyterHub 更新而来的回归。为了确保您使用 ODH 0.6 的体验不会因为我们希望尽早发布而受到影响，我们提供了一个新的(主要是)错误修复版本:Open Data Hub 0.6.1。

## 操作员

可能最重要的 bug 修复进入了[操作符的代码](https://github.com/opendatahub-io/opendatahub-operator/)本身。那里的变化对项目很重要，不仅因为它们修复了什么，还因为它们证明了我们与 Kubeflow 社区的关系，因为大部分代码直接进入了[上游](https://github.com/kubeflow/kfctl)。我们还重新调整了我们的存储库，以确保运营商中的任何新功能都可以加入。让我们来看看已经修复的一些东西。

### 使用实例删除命名空间

[这个问题](https://github.com/kubeflow/kfctl/issues/241)导致了一些你通常不会从应用程序中预料到的事情。当用户删除 KFDef 自定义资源时，运营商也删除了部署开放式数据中心的名称空间。这肯定是我们不希望的，因为可能有其他应用程序在 ODH 不控制的名称空间中运行。我们提出了这个问题，并与社区合作，让运营商行为端正。

### 缓存和冲突

Kubeflow 的`kfctl`工具在本地下载并缓存清单，因此您不需要在每次运行命令时都下载它们。它还基于 KFDef 内容生成 Kustomize 结构和清单。

这个流在本地工作得很好，您可以手动移动东西，但是在操作符中，这意味着所有的 KFDef 自定义资源使用与第一个相同的缓存。之后，所有实例也从相同的生成的 Kustomize 清单中部署。那种行为显然是不对的，并导致了很多问题。

新版操作符更好地处理了这个问题。它将缓存和 Kustomize 清单放在基于名称空间和 KFDef 名称的目录中，并在必要时重新加载缓存以适应任何潜在的清单更改。

### 建立操作员形象

[最后一个变化](https://github.com/kubeflow/kfctl/pull/321)我们需要在运营商是关于如何建立形象。由于`[operator-sdk](https://github.com/operator-framework/operator-sdk)`基于克隆的目录名命名管理器二进制文件，我们需要在 Dockerfile 文件中对其进行参数化，以适应从`kfctl`到`opendatahub-operator`的变化。另一个变化是我们倾向于在通用基础映像(UBI)之上运行，这与 Kubeflow Operator 不同，因此我们将构建过程定制为插入[替代 Dockerfiles](https://github.com/kubeflow/kfctl/blob/master/build/Dockerfile.ubi) 。

## 显示

我们在`odh-manifests`库中做了一些更新，主要是为了容纳缺失的组件和改进文档。

### READMEs

我们向阅读材料中的所有组件添加了基本描述。只需[转到存储库](https://github.com/opendatahub-io/odh-manifests/)并点击组件。您将看到一个自述文件，其中讨论了该组件的用途、依赖项和配置选项，以及如何在 KFDef 资源中启用该组件的示例。

### JupyterHub

正如我们在 ODH 0.6.0 的公告中提到的，JupyterHub 依赖项之一——[JupyterHub 单用户配置文件](https://github.com/vpavlin/jupyterhub-singleuser-profiles/)——经历了一些变化，我们发现[有一些回归](https://github.com/opendatahub-io/odh-manifests/issues/44)。它们都被修复了，包括一个阻止在支持 GPU 的节点上成功部署 Jupyter 笔记本服务器的问题。

### 艾图书馆和很少

我们在之前的版本中省略了 [AI 库](http://opendatahub.io/docs/ai-library.html)，因为我们遗漏了 Seldon，它是 AI 库的一个依赖项。由于[塞尔顿操作员](https://catalog.redhat.com/software/operators/detail/5e9f1a3769aea31642b613f4)最近被成功认证，我们能够通过操作员生命周期管理器添加它，从而再次启用 AI 库。

## 测试和持续集成

开放式数据中心开发和维护的一个主要的长期问题是缺乏对传入的拉请求(PRs)的自动化测试。因为这个问题，我们所有的验证都是手动的，花费了很多时间。自从我们开始计划迁移到 GitHub，我们就对 OpenShift CI 寄予厚望，认为它是我们持续集成基础设施的可行解决方案。

我们很高兴地分享，我们现在[被连接到 OpenShift CI](https://github.com/opendatahub-io/odh-manifests/blob/master/tests/TESTING.md) 中，并且[测试](https://github.com/opendatahub-io/odh-manifests/tree/master/tests)正在`odh-manifests`库中的所有 pr 上运行。我们将致力于添加更多的测试，并关注新的 PRs，以确保它们带有测试，以避免在未来引入回归。

*Last updated: June 25, 2020*