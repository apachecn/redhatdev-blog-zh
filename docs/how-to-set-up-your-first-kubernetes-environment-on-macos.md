# 如何在 macOS 上设置您的第一个 Kubernetes 环境

> 原文：<https://developers.redhat.com/blog/2019/04/15/how-to-set-up-your-first-kubernetes-environment-on-macos>

通过阅读[我在这个系列中的前一篇文章](https://developers.redhat.com/blog/2019/03/21/containers-kubernetes-and-microservices-start-here/)，你已经粉碎了整个[容器](https://developers.redhat.com/blog/category/containers/)的事情。这比你预期的要容易得多，而且你已经更新了你的简历。现在是时候进入聚光灯下，走红地毯，拥有整个 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 游戏了。在这篇博文中，我们将在 macOS 上启动并运行我们的 Kubernetes 环境，在容器中旋转一个图像，然后前往 [Coderland](https://developers.redhat.com/index.php/coderland/serverless/serverless-knative-intro/) 。

## 零件

不像我最近购买的某个品牌的橱柜，它只有简笔画和一些数字和箭头的说明，我们会用图表和文字来展示项目和步骤。我们需要:

1.  一种运行容器的方法
2.  库伯内特斯
3.  Kubernetes 命令行工具。
4.  Red Hat OpenShift 的`oc`命令行工具
5.  作为测试运行的图像。

## 一种运行容器的方法

我们需要某种环境来运行容器。选项包括 [Minikube](https://kubernetes.io/docs/tutorials/hello-minikube/) 、 [Minishift](https://www.okd.io/minishift/) 、 [CodeReady Containers](https://developers.redhat.com/products/codeready-containers) 和一个 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview/) 集群(运行在 AWS 上)。我们将通过选择 CodeReady 容器(缩写为“CRC”)来保持事情的简单性和前瞻性。这将在我们的本地机器上运行，同时给我们带来 Kubernetes(顺便说一下，还有 OpenShift)的强大功能，而无需花费任何金钱。

那总是好的。

所以让我们在 macOS 上安装 CodeReady 容器。它是五个步骤。

## 安装 CodeReady 容器

1.  访问 [CodeReady Containers 下载页面](https://cloud.redhat.com/openshift/install/crc/installer-provisioned?intcmp=7013a000002CtetAAC)并下载最新的 macOS 兼容版本。保持此页面打开；你会在第五步用到它。
2.  解压缩文件。最简单的方法是双击文件，或者使用`tar`。
3.  确保文件位于您希望它驻留的位置(例如，~/crc 目录)。
4.  确保该目录在您的系统路径中。
5.  当您在 CodeReady Containers 下载页面时，继续下载 pull secret 按钮就在页面下方一点。这将用于第一次启动 Kubernetes(实际上是 OpenShift)集群时的身份验证。

看起来是这样的:

![](img/58e8f04dfea1dd03689321a4ad6c66e5.png)

### 证明它

命令`crc version`应该产生:

![](img/fe9360308c1b50f72f647c4e5347acd8.png)

请注意，根据您执行此操作的时间，您可能拥有比此处显示的版本更新的版本。

## 正在安装 kubernetes

额外收获:它包含在 CRC 中。哇，那很容易。

## 安装 kubectl

Kubernetes 命令行工具 kubectl 很容易安装在 macOS 上:

```
brew install kubernetes-cli
```

(如果失败，在 [kubectl 安装页面](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl)上有更深入的解释。)

## 安装 oc

```
brew install openshift-cli
```

(或者，可以使用 [okd 页面](https://docs.okd.io/latest/cli_reference/get_started_cli.html#cli-mac)的 DIY 说明。OKD 是 OpenShift 的上游版本。这个上游会谈是关于什么的？这是[红帽战略](https://opensource.com/article/16/12/why-red-hat-takes-upstream-first-approach)的一部分。)

## 设置它

在开始使用 CRC 之前，您需要对它进行初始化。有几个选项你可以设置，比如多少内存和 CPU 的数量，但是为了简单起见，让我们使用默认值。使用以下命令:

`crc setup`

![](img/b108ce8fcce55a46144b64b677bf1fe5.png)

## 点燃它

是时候让您的集群开始运行了。这是一个命令，但是，你需要你的“拉秘密”第一。将它拷贝到 Mac 的剪贴板，以便您可以将其粘贴到终端窗口中。

这很简单。在命令行中，使用以下命令:

```
crc start
```

系统将提示您输入提取密码。将其粘贴到命令行。然后按回车键，CRC 将开始。

![](img/4ec5531bb2252dff4fd25596b26a4dfe.png)

注意:如果你想在任何时候用 CRC 重新开始，使用命令`crc stop`和`crc delete --force --clear-cache`。之后你需要跑`crc setup`。

耐心点，这需要几分钟。

完成后，我们需要一些命令来“连接”到我们的集群。我们要在这里作弊，使用一些 OpenShift 命令。这些命令是快捷方式。如果我们不使用它们，我们就必须修改我们的 Kubernetes 配置，创建一个用户并授予访问权限。我们可以省去很多步骤。如果你想只使用`kubectl`并成为一个纯粹主义者，你可以关注这篇博文“[使用 Kubectl](https://blog.christianposta.com/kubernetes/logging-into-a-kubernetes-cluster-with-kubectl/) 登录 Kubernetes 集群”幸运的是，登录说明就显示在屏幕上。为了方便起见，您可能想要保存它。如果没有，您需要登录，唯一的解决方法是停止，然后再次启动集群。

![](img/4d1c559e9437a3347b7d56918446ac4c.png)

```
eval $(crc oc-env)
oc login -u kubeadmin -p 7z6T5-qmTth-oxaoD-p3xQF https://api.crc.testing:6443
```

## 要运行的图像

最后，运行一个非常基本的映像作为 Kubernetes pod 来测试您的设置是一个好主意。为此，让我们在 pod 中运行一个映像，然后运行`curl`以确保它按预期工作。

使用此命令旋转 pod:

```
kubectl run qotd --image=quay.io/donschenck/qotd:v2 --port=10000
```

这将从我的公共存储库下载一个图像到您的系统，并使用 Kubernetes 运行它。

再详细一点:这将创建一个名为 *qotd* 的 [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) ，检索图像，在一个容器中启动图像，并使用端口 10000 路由到它。请注意，pod 名称和图像名称不需要匹配。这是一个你想把一些管理思想放在适当位置的领域。换句话说，如果你考虑不周，这是一个让事情变得非常混乱的好机会。别问我是怎么知道这个的。

请注意，等待此 pod 启动并运行可能需要几分钟时间，这取决于您的计算机的性能。在服务器或高性能 PC 上完成时，大约需要一分钟左右。我的配有 i5 处理器的 MacBook Air 大约需要四分钟。你可以通过运行`kubectl get pods`来检查它。

当 pod 启动并运行时，您无法从命令行访问它。这是为什么呢？因为它在您的 Kubernetes 集群“内部”运行。然而，Kubernetes 是智能的，它为 Kubernetes 中的 pods 提供了一个代理，因为可能有几个容器运行同一个应用程序；一箱集装箱。都是同一个 URI。当你跑`kubectl get pods`时，你可以看到你的 *qotd* 吊舱。

![](img/0218839d4bae51b905f27cfb6c4e2f2c.png)

## 到达应用程序

Kubernetes 创造的代理有两个方面。一个方面是代理本身。另一个方面是代理的公众形象，它允许你访问你的豆荚。换句话说，代理运行在端口 8001 上，而代理路由允许您访问应用程序。

测试代理及其对 pod 的访问分为两步。不用担心；这一点以后会变得更好、更容易。但是现在，我们必须启动代理，然后通过代理访问 pod。为此，您需要打开第二个终端窗口。

在第一个终端窗口中，运行以下命令:

```
kubectl proxy
```

代理正在运行。这将占用命令行(即，它以交互方式运行)，因此您需要第二个终端窗口来运行以下命令，该命令将返回代理路由列表:

```
curl http://127.0.0.1:8001
```

[![](img/5a7fd936405447a87ada4036b83b92c0.png "Screen Shot 2019-03-27 at 3.58.01 PM")](/sites/default/files/blog/2019/03/Screen-Shot-2019-03-27-at-3.58.01-PM.png)Kubernetes proxy routes">

哇哦。那些结果。这些都是 Kubernetes 代理中内置的路线。问题是:我们还没有达到我们的应用程序...只是代理。

通向我们的应用程序的端点是`/api/v1`。我们要的格式是`api/v1/namespaces/{our namespace}/pods/{pod name}/proxy/`。

在我们的特定实例中，{我们的名称空间}是`test`。

同样，可以通过运行`kubectl get pods`找到 pod 名称。在我们的例子中是 *qotd* 。

将这些部分放在一起，您就可以从您的第二个终端访问我们的应用程序(记住:`kubectl proxy`仍然在我们的第一个终端窗口中运行):

```
curl http://127.0.1:8001/api/v1/namespaces/test/pods/qotd/proxy/
```

## 让我们找点乐子

该应用程序有几个端点。给他们一个机会:

/版本

/报价

/报价/随机

/quotes/1

## 顺便问一下...

对了， *qotd* 是用 Go 写的。容器的魔力:(几乎)所有的开发语言都受欢迎。

## 等等，还有呢

虽然您现在已经在本地机器上运行了一个 Kubernetes 集群，但是还有很多需要了解和做的事情。一定有比在第二个终端中运行`kubectl proxy`更简单的方法来访问您的应用程序。必须有一种方法来运行多个容器或多个应用程序。一定有一种方法可以在代码运行时更新它——众所周知的“滚动更新”。

确实有。随着本系列的继续，我们将涵盖所有这些内容。同时，拥有那条红地毯。

附言:发音是“kube-crowd”

## 也阅读

[如何在 Windows 上设置您的第一个 Kubernetes 环境](https://developers.redhat.com/blog/?p=580387)

*Last updated: April 24, 2020*