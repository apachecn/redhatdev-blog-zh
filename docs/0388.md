# 开放式数据中心和 Kubeflow 安装定制

> 原文：<https://developers.redhat.com/blog/2020/07/23/open-data-hub-and-kubeflow-installation-customization>

Kubernetes 的主要目标是达到期望的状态:部署我们的豆荚，建立网络，并提供存储。这个范例扩展到了[操作符](https://www.openshift.com/learn/topics/operators)，它们使用[定制资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)来定义状态。当操作者选择定制资源时，它将总是试图到达它所定义的状态。这意味着，如果我们修改由操作员管理的资源，它将很快替换它以匹配所需的状态。

当我们试图定制开放数据中心(ODH)或 Kubeflow 安装时，这可能会变得混乱，因为 [KFDef 定制资源](https://github.com/opendatahub-io/odh-manifests/blob/master/kfdef/kfctl_openshift.yaml)没有公开所有潜在的配置选项。

在本指南中，我将通过三个选项来修改部署:直接编辑 fork 中的清单、使用覆盖创建存储库以及添加覆盖。我将使用一个通过提供的 [ConfigMap](https://github.com/opendatahub-io/odh-manifests/blob/master/jupyterhub/jupyterhub/base/jupyterhub-configmap.yaml) 定制 JupyterHub 组件的例子。

## 如何自定义安装

您可以通过三种不同的方式自定义部署。所有这些都涉及到使用 Git 和 odh-manifest 库:

*   方法 1:Fork odh——在那里显示和维护更改。
*   方法 2:创建自己的清单存储库来覆盖组件资源。
*   方法 3:派生 odh-manifest 存储库，并用组件覆盖来扩展它。

以上每一个都需要你有一个 Git 库，里面有 [Kustomize](https://kustomize.io/) 清单和 [Red Hat OpenShift](https://developers.redhat.com/openshift) 资源。(您将从 KFDef 资源中引用这些内容。)然而，每种方法都需要不同程度的努力来创建和维护。

我推荐定制安装的第三种方法。然而，在本文中，我将详细介绍每个选项。

### 方法 1:派生并修改 odh 清单存储库

派生和修改 odh-manifest 存储库并维护其中的更改:

1.  转到 [odh 清单库](https://github.com/opendatahub-io/odh-manifests)并点击 fork 按钮。
2.  克隆您的 fork 并对其进行更改。
3.  将更改推送到分叉的存储库，更新 [KFDef 资源](https://github.com/opendatahub-io/odh-manifests/blob/v0.6.1/kfdef/kfctl_openshift.yaml#L118)以指向分叉，然后部署。

从这种方法开始非常简单。但是，如果您需要与原始 odh-manifests 存储库同步，可能很难维护。做出任何改变都可能导致 Git 级别的冲突。这将需要您在重新调整基数时进行手动干预。即使对于添加环境变量或更改其值这样简单的事情也是如此。

另一方面，很容易看到上游存储库所做的更改，因为您有 Git 历史和跟踪。

### 方法 2:通过覆盖进行定制

要通过覆盖进行自定义，请从空存储库开始，然后:

1.  创建与 odh 清单库中的目录结构相匹配的目录结构。
2.  添加您想要自定义的文件。
3.  将您的存储库添加到 KFDef 资源中，作为额外的存储库引用。然后复制您正在定制的组件。使用您的覆盖将存储库引用(repoRef)更改为存储库是很重要的。

操作员按照定义的顺序使用 KFDef 中的存储库和组件。这意味着您的文件将替换来自 odh 清单的文件。

这种方法很简单，您不必担心 Git 冲突。另一方面，在上游 odh-manifests 存储库中跟踪您的更改更加困难。例如，操作员可以重命名您在 odh 清单中覆盖的文件。那么您将会以不一致的状态结束，在这种状态下，您不会覆盖资源，而只是将它添加到已部署的清单集中。在这种方法中很难发现这样的变化，因为您没有连接两个存储库的 Git 历史。

例如，让我们定制 JupyterHub 组件。为此，我们将使用[odh-manifest-overrides 示例存储库](https://github.com/vpavlin/odh-manifests-overrides/tree/master)，它为作为 JupyterHub 组件的一部分部署的 ConfigMap 提供定制:

1.  部署 Open Data Hub Operator 并将 KFDef 自定义资源上传到集群(参见[快速安装指南](https://opendatahub.io/docs/getting-started/quick-installation.html)):

```
$ oc apply -f https://raw.githubusercontent.com/vpavlin/odh-manifests-overrides/master/kfdef/kfctl_openshift_custom.yaml
```

2.  等待 Open Data Hub 实例启动。
3.  运行以下命令以获取自定义的配置图:

```
$ oc describe cm jupyterhub-cfg
```

内容应该与[odh-manifest-overrides 存储库](https://github.com/vpavlin/odh-manifests-overrides/blob/master/jupyterhub/jupyterhub/base/jupyterhub-configmap.yaml)中可用的配置图相匹配。

### 方法 3:用覆盖图定制

最后一种方法结合了前面的方法:

1.  派生存储库，但是不要接触存储库中已经存在的任何清单。
2.  添加新的覆盖。
3.  创建新资源或删除并修改现有资源。

使用覆盖比直接编辑清单更复杂。然而，这个过程比创建覆盖更方便和一致。从长远来看，覆盖方法提供了更多的灵活性。有了它，您还可以在相同的代码库中为不同的用例或环境创建不同的覆盖。

这种方法防止了 rebases 上的冲突，因为它不需要您更改任何现有的清单。覆盖应该在 rebases 和 ODH 版本之间保持工作。有几个例外。一个是如果你正在做一些复杂的事情，比如通过补丁重命名现有的资源。另一个例外是如果上游组件有重大更改，比如需要完全更改资源类型的主要组件版本更改。

为了定制 JupyterHub 组件，让我们看另一个例子。这次我们将使用[odh-manifest-overlays 存储库](https://github.com/vpavlin/odh-manifests-overlays)，它提供了一个 overlays 方法的示例。您可以立即看到这是 odh-manifest 存储库的副本。它还分享了它的 Git 历史。

这个版本另外有一个提交，它向 JupyterHub 组件添加了一个[覆盖。此覆盖修改了 JupyterHub 配置图。它还添加了一个额外的 JupyterHub 单用户配置文件配置映射:](https://github.com/vpavlin/odh-manifests-overlays/commit/HEAD)

让我们来测试一下:

1.  部署开放式数据中心操作器(参见[快速安装指南](https://opendatahub.io/docs/getting-started/quick-installation.html))。
2.  将[定制的 KFDef 资源](https://github.com/vpavlin/odh-manifests-overlays/blob/master/kfdef/kfctl_openshift_custom.yaml)上传到集群:

```
$ oc apply -f https://raw.githubusercontent.com/vpavlin/odh-manifests-overlays/master/kfdef/kfctl_openshift_custom.yaml
```

自定义配置图应出现在集群中:

```
$ oc describe cm jupyterhub-cfg
```

3.  您还可以通过运行以下命令来查看添加的 ConfigMap:

```
$ oc describe cm jupyterhub-additional-singleuser-profiles
```

## Kubeflow

在本文的开始，我承诺您也将学习如何定制 Kubeflow 部署。然而，我甚至没有提到库伯弗洛。这是因为由于开放式数据中心采用了 Kubeflow 部署工具，您可以将任何关于开放式数据中心的内容应用于 [Kubeflow 清单](https://github.com/kubeflow/manifests/)。两个存储库遵循相同的结构，并利用相同的模式进行组件定义和定制。(详见[“Open Data Hub 0.6 带来组件更新和 Kubeflow 架构”](https://developers.redhat.com/blog/2020/05/07/open-data-hub-0-6-brings-component-updates-and-kubeflow-architecture/)。)

## 结论

在本文中，我介绍了三种不同的方法来定制和维护对单个开放式数据中心部署的更改。这些是直接编辑 fork 中的清单，创建带有覆盖的存储库，以及添加覆盖。

开放数据中心文档只详细讨论了其中的一个:覆盖层。我们认为它是三者中最灵活、最易维护、最方便的。这也是我们对 Red Hat 开放数据中心内部部署的建议。内部数据中心团队维护 odh 清单存储库的一个分支，并使用覆盖来定制默认配置，以满足该生产部署的需求。

*Last updated: July 21, 2020*