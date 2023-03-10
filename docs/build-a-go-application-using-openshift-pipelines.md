# 使用 OpenShift 管道构建 Go 应用程序

> 原文：<https://developers.redhat.com/blog/2020/05/26/build-a-go-application-using-openshift-pipelines>

Go 是一种越来越流行的编程语言，经常被用来开发命令行实用程序。许多与 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 一起使用的工具都是用 [Go](https://developers.redhat.com/blog/category/go/) 编写的，包括 Tekton ( `tkn`)、OpenShift ( `oc`)和 Kubernetes ( `kubectl`)的命令行接口(CLI)。此外，开发人员可以针对各种操作系统将 Go 编译成一个可执行文件。因此，在将应用程序放入容器并在 OpenShift 中运行这些容器之前，很容易开发和桌面测试应用程序。

在某种程度上，这是一篇关于教程的文章，我将向你展示如何使用 OpenShift 管道构建和交付一个小型的 Go RESTful 服务。你现在可以直接跳到教程，但是我建议你先阅读这篇文章。我将快速介绍本教程的工作环境，并解释我以这种方式设置本教程的逻辑。

## OpenShift 管道研讨会教程包括什么

OpenShift Pipelines Workshop 教程包括两个 GitHub 库。一个存储库包含您将用于示例应用程序的教程和两个 YAML 文件。另一个存储库包含您将构建的 Go 服务。本教程还引用了 [OpenShift 管道目录](https://github.com/openshift/pipelines-catalog)，这是一个可重用管道资产的开源库。这个目录是开源世界以及它如何产生有价值的社区范围的解决方案的一个很好的例子。

在[教程库中](https://github.com/redhat-developer-demos/openshift-pipelines-workshop.git)，你会发现两个 YAML 文件:`qotd-pipeline.yaml`和`sub.yaml`。`sub`文件创建 OpenShift 管道操作符，而`qotd-pipeline`定义要使用的管道。

在[源代码库](https://github.com/redhat-developer-demos/qotd.git)中，您将找到服务的 Go 代码。您会发现一个 Dockerfile 文件，您可以使用它来创建图像。您还会在`/k8s`目录中找到三个 YAML 文件。这些文件定义了您将要创建的`DeploymentConfig`、`Service`和`Route`。将这些工件保存在与源代码相同的存储库中是有意义的，它将所有相关的部分放在一个容易访问的地方。

## 构建环境

OpenShift Pipelines 依赖于 Tekton，这是 Knative 基于容器的构建组件。Tekton 使用任务来完成工作，例如构建符合 Linux 开放容器倡议(OCI)的映像。在这种情况下，我称这个映像为“OCI 兼容的”,因为我们不使用`docker`命令来构建任何东西。相反，我们将使用开源的`buildah`系统。`buildah`任务包含在前面提到的 OpenShift 管道目录中，并且在安装 OpenShift 管道操作符时包含在内。这是我们将在这个项目中使用的几个集群范围的任务之一。运行`tkn clustertask ls`可以看到所有任务的列表。

**关于 OCI** : [开放容器倡议(OCI)](https://www.opencontainers.org/about) 是一个开放的治理结构，用于围绕容器格式和运行时创建开放的行业标准。OCI 说，“按照这些标准建立一个形象，它就会像承诺的那样运行。”

### buildah 的优点是什么

让您控制要构建什么以及如何构建。例如，我的源代码中有一个 docker 文件，我可以使用它在本地机器上构建和桌面测试代码，而不管我使用的是什么操作系统。当我将所有东西都转移到 OpenShift 并使用管道时，我可以使用同一个 Dockerfile 来执行构建。我可以放心(没有双关语)创建的可执行文件和映像将与我的机器上的相匹配。没有了，“但它在我的机器上工作。”使用`buildah`，我可以在星期五下午 4:59 部署我的代码，然后无忧无虑地回家。(我加上最后一句话只是为了激怒运营部门的人。)

## 结论

这篇简短的文章是我的长篇教程的介绍。现在你知道在哪里可以找到你需要的组件，以及我为什么把它们放在那里。是时候去 GitHub 工作室开始学习教程了！

*Last updated: June 26, 2020*