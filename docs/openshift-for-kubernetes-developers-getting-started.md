# 面向 Kubernetes 开发者的 OpenShift:入门

> 原文：<https://developers.redhat.com/blog/2020/08/14/openshift-for-kubernetes-developers-getting-started>

如果你熟悉[容器](https://developers.redhat.com/topics/containers)和 [Kubernetes](https://developers.redhat.com/topics/kubernetes) ，你可能听说过[红帽 OpenShift](https://developers.redhat.com/products/openshift/getting-started) 带给这个平台的企业特性。在本文中，我向熟悉 Kubernetes 的开发人员介绍了 OpenShift 的命令行特性和原生扩展 API 资源，包括构建配置、部署配置和图像流。

## OpenShift 的命令行界面

我们先来看看 [OpenShift 的`oc`命令行实用程序](https://docs.openshift.com/container-platform/4.5/cli_reference/openshift_cli/getting-started-cli.html)如何在几个关键领域构建 Kubernetes `kubectl`实用程序。

### 身份验证和安全性

`kubectl`命令要求您创建一个`kubeconfig`文件来登录您的集群。使用`oc`实用程序，您只需使用您的凭证登录即可:

```
$ oc login -u <user_name>  <cluster_url>

```

**注意**:如果您可以访问 [Red Hat CloudForms 管理引擎](https://access.redhat.com/products/red-hat-cloudforms)，您可以提供一个共享集群用于学习目的。

除了登录到您的集群，您还可以使用您的凭证登录到通过 OpenShift 安装的其他组件，比如 Jenkins 和 Red Hat Registry。

虽然 Kubernetes 不需要基于角色的访问控制(RBAC)，但 RBAC 和基本认证一样，是 OpenShift 不可或缺的一部分。安全性至关重要，尤其是在企业级。

### 项目管理

登录后，您可以从`oc`命令行轻松地在 OpenShift 项目和名称空间之间切换。只需输入以下内容:

```
$ oc project <project-name-to-switch-to>

```

在 Kubernetes 中，您可以使用`kubectl`实用程序在项目和名称空间之间切换，但是它需要额外的工具，比如`kubens`和`kubectx`。

## 应用程序工作流

简化工作流程是 OpenShift 给 Kubernetes 带来的最本质的特性。为了开始在您的项目中开发应用程序，OpenShift 支持模板。一个*模板*是一个蓝图，你可以用它来开发你的应用。修改快速入门模板是创建新应用程序的一种简单快捷的方法。

如本文中[所述，OpenShift 还支持](https://www.openshift.com/blog/from-templates-to-openshift-helm-charts)[舵图](https://developers.redhat.com/blog/2020/07/20/advanced-helm-support-in-the-openshift-4-5-web-console/)。Kubernetes Helm chart 是一个应用程序管理器，它将应用程序及其依赖项打包成一个. zip 文件。您可以使用 Helm charts 在 OpenShift 中部署应用程序或更大应用程序的组件。

## 示例:OpenShift Django 快速入门

现在考虑一个例子。假设我们正在使用 OpenShift 的 Django quickstart 模板在 GitHub 存储库中部署应用程序。我已经为 Django 创建了一个 Hello World 应用程序，我将在示例中使用它。您可以[在这里](https://github.com/mynamo/django-openshift-webhook)找到示例应用。

我们从以下命令开始:

```
$ oc new-app <template> --param=SOURCE_REPOSITORY_URL=<git_url>

```

默认情况下，`oc new-app`命令会自动创建任何应用程序所需的所有 OpenShift 资源。

**注意**:如果您的应用程序和我的一样简单，并且不接受任何用户请求，您可能需要移除 [OpenShift 健康检查](https://developers.redhat.com/blog/2020/07/20/best-practices-using-health-checks-in-the-openshift-4-5-web-console/)准备就绪探测器。

### 构建配置

接下来，我们将查看构建配置。要开始，请运行命令:

```
$ oc get bc

```

此命令返回应用程序的构建配置。一个*构建配置*描述了一个构建定义和一组创建新构建的触发器。 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview) (OCP)使用 Kubernetes 从构建映像创建容器，并将它们推送到容器映像注册中心。它支持基于可选类型的其他构建策略，这些策略在构建 API 中指定。关键的构建策略是:

*   码头工人建造
*   源到映像(S2I)构建
*   定制构建
*   管道

有关这些构建策略的更多信息，请参见映像构建的 [OpenShift 文档](https://docs.openshift.com/container-platform/4.5/builds/understanding-image-builds.html)。要找出应用程序使用的构建策略，请运行以下命令:

```
$ oc describe bc django-ex | grep Strategy

```

### 部署配置

运行该命令时:

```
$ oc get dc

```

当您输入`new-app`命令时，您将看到为应用程序创建的部署配置。

部署是管理应用程序的一种方式。Kubernetes 中的部署对象和 OpenShift 中的 DeploymentConfigs 通过在模板中定义 pod 的所需状态来管理应用程序。这两个平台在这个过程中使用的方法和 API 对象略有不同。我将介绍 OpenShift 添加到 Kubernetes 工作流中的几个有用的特性。

#### 生命周期挂钩

您可以使用生命周期挂钩将定制行为添加到部署策略中。例如，您可以使用 Git 挂钩在每次更新被推送到 GitHub 时自动部署应用程序。您所要做的就是从您的 OpenShift 集群中复制 Git hook URL，并将其粘贴到您的存储库的`settings -> webhooks`部分。使用以下命令获取 webhook URL:

```
$ oc describe bc <build-config-name>

```

在 webhook 中，使用以下命令中的秘密替换`<secret>`:

```
$ oc get bc <build-config-name> -o yaml

```

现在，当您向您的项目推送更新时，您可以看到 OpenShift 自动开始一个新的部署。

#### 图像流

Kubernetes 中 DeploymentConfigs 和 deployment objects 的另一个区别是，OpenShift 支持*图像流*作为一种方便管理容器图像的方式。

在 Kubernetes 中，您使用外部工具如`skopeo`来管理容器图像。如果没有`skopeo`，您必须下载整个图像，在本地更新，然后像在 Git 中的任何应用程序一样推回它。您还必须更新容器标记和部署对象定义。

使用图像流，只需上传一次容器图像，然后在 OpenShift 中管理它的虚拟标签。基于项目的开发阶段，您更改标签来跟踪图像的版本，通常将最新版本设置为`latest`。当将`ImageStream`与`DeploymentConfig`一起使用时，您可以设置一个触发器，以便在新图像出现或标签改变其引用时启动部署。这样，每当构建新版本时，应用程序都会进行部署。

您可以使用以下命令获取项目中的`imageStreams`:

```
$ oc get is

```

## 结论

我希望这篇文章有助于理解 Kubernetes 和 OpenShift 上的开发之间的差异，也有助于开始使用 OpenShift 部署应用程序。参见 [OpenShift 文档](https://docs.openshift.com/container-platform/4.5/welcome/)和以下参考资料，了解更多关于本文中讨论的图像流标签和其他 OpenShift 特性的信息:

*   [OpenShift 集装箱平台 4.5](https://docs.openshift.com/container-platform/4.5/welcome/index.html)
*   [理解图像构建](https://docs.openshift.com/container-platform/4.5/builds/understanding-image-builds.html)
*   【OpenShift 和 Kubernetes 最重要的十个区别
*   [OpenShift 和 Kubernetes:有什么区别？](https://www.redhat.com/en/blog/openshift-and-kubernetes-whats-difference)
*   [企业 Kubernetes 与 OpenShift(上)](https://www.openshift.com/blog/enterprise-kubernetes-with-openshift-part-one)

*Last updated: August 12, 2020*