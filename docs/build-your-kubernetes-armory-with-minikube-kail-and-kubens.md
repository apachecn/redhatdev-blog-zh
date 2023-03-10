# 用迷你库贝、凯尔和库本斯建造你的库本斯军械库

> 原文：<https://developers.redhat.com/blog/2019/04/16/build-your-kubernetes-armory-with-minikube-kail-and-kubens>

[Kubernetes](https://developers.redhat.com/topics/kubernetes/) 已经成长为构建云原生应用的事实上的开发平台。作为开发人员，我们希望从单词 go 开始，或者说，从单词*代码开始，就能高效地工作。但是为了提高效率，我们必须配备合适的工具。* 在这篇文章中，我将看看应该成为你的 Kubernetes 工具箱或军械库的一部分的三个重要工具。

## 迷你库贝

当您想到 Kubernetes 时，可能会想到几个问题，包括:

1.  我如何设置每一个 Kubernetes 组件？
2.  如何正确配置 Kubernetes？
3.  即使我正确地设置和配置了 Kubernetes，我也必须每次都这样做吗？

答案是 [Minikube，](https://kubernetes.io/docs/setup/minikube/)它可以帮助您轻松设置一个适合您日常开发需求的 Kubernetes 节点。

开始之前，`minikube start`命令只是启动一个单节点 Kubernetes 环境，其中有 *2GB RAM、2 个 CPU*和一个 *20GB 的磁盘*。

运行命令:

```
minikube ip
```

会让您知道 Kubernetes 节点的 IP，该节点是由 Kubernetes 启动的虚拟机的一部分。

我能够使用默认配置部署小型轻量级应用程序，但是有一天我需要构建一个更大的 Kubernetes 节点，内存为 8GB RAM，4 个 CPU，磁盘大小为 50GB。要获得这种配置，您可以使用:

```
minikube start --memory=8192 --cpus=4 --disk-size=50g
```

随着您成为更有经验的 Kubernetes 用户，您将拥有一个用于一组应用程序的 Kubernetes 环境，另一个用于另一组应用程序，等等。使用 Minikube 中所谓的“个人资料”,这非常简单。让我们创建一个名为*剑*的概要文件，它将包含我们所有的“尖端”技术应用程序。要创建你的*剑*档案，你只需运行相同的`minikube start`，但使用`-p`就像:

```
minikube start -p sword --memory=8192 --cpus=4 --disk-size=50g
```

默认情况下，Minikube 将*剑*配置文件的所有文件、配置和其他相关设置存储在*$ MINIKUBE _ HOME/profiles/sword*下。 *$Minikube_HOME* 是一个环境变量，MINIKUBE 认为它是存储 MINIKUBE 虚拟机工件(如配置、磁盘等)的地方。,

良好的第一步！现在，我要教你一个窍门。下面是调用 Kubernetes 服务的方法，例如，一个使用 Minikube 的“客户”:

```
minikube service customer
```

当学习如何使用这些工具时；然而，我们也需要考虑其他工具。

## 忽必烈:切换忽必烈命名空间

Kubernetes 默认有很多名称空间，比如*默认*、 *kube-public* 和 *kube-system* 。您创建的任何没有名称空间的资源都是在名称空间*默认*中创建的。

例如，在没有`--namespace`或`-n`选项的情况下执行以下命令，将在*默认*名称空间中创建“dagger”配置图。

```
 kubectl create configmap dagger --from-literal size=small
```

在考虑应用程序开发优先级时，我们经常会忽略这种微妙的行为。这时，我们开始度过不眠之夜和周末，想知道，为什么这对我不起作用？![](img/e353d017f737371111808965262e3095.png)

通过将[库本斯](https://github.com/ahmetb/kubectx)添加到我们的军械库，我们可以设置创建资源的名称空间。假设我们有一个名为 *vikings* 的名称空间；我们可以通过运行`kubens vikings`命令将其设置为所有资源的名称空间。当我们试图在没有`--namespace`或`-n`选项的情况下创建任何资源时，Kubernetes 将默认在 *vikings* 名称空间中创建它。

如果您必须回到之前的名称空间，只需运行命令`kubens -`(类似于 Bash `cd -`)即可。

## 凯尔:看日志

我们的工具箱正在被很好地填满，但是我们仍然缺少重要的工具。作为我们日常工作的一部分，我们需要分析 pod 的日志。

Kubectl 确实有 log 命令，但那是原始的。我们需要一个工具来帮助我们根据一些标准过滤日志，例如，属于 Kubernetes 部署的 pod。

通过将 [Kail](https://github.com/boz/kail) 添加到我们的工具包中，我们将通过过滤器获得更大的能力来查询和分析日志:

*   按部署的 pod—例如，从属于名称空间 sword 中的部署 dagger 的 pod 获取日志:

```
kail -n sword -d dagger
```

*   按服务分组—获取服务战中分组的日志:

```
kail -n sword --svc battle
```

*   按标签分类的 pod—获取所有“尺寸=大”的 pod 的日志:

```
kail -n sword --label size=large
```

还可以组合过滤器来进一步过滤日志。例如，假设您想要查看没有“size=large”的服务“battle”的所有日志:

```
kail -n sword --svc battle --ignore size=large
```

Kail 有许多其他过滤器，只需运行`kail --help`即可获得更多详细信息。我刚开始用 Kail 的时候，有时候看不到日志，但是后来我意识到我的主机时间和 Minikube 时间是不一样的，所以我必须在我的命令中加上`--since`标志。例如，要查询最近 2 小时(Kubernetes 节点时间)的 pod 日志，请使用:

```
kail -n sword --svc battle --since=2h
```

本文到此为止。有了这些工具，我们就可以在代码战中取得成功了。在我的下一篇文章中，我们将为我们的军械库添加更多的工具，并进一步扩展我们的技能。在那之前，编码快乐！

## 教程

以下是我们库伯内特战士的教程:

1.  伊斯迪奥:[github.com/redhat-developer-demos/istio-tutorial](https://github.com/redhat-developer-demos/istio-tutorial)
2.  knative:[github.com/redhat-developer-demos/knative-tutorial](https://github.com/redhat-developer-demos/knative-tutorial)

*Last updated: September 3, 2019*