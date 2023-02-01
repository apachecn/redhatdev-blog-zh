# 集装箱化。NET for Red Hat OpenShift: Windows 容器和。NET 框架

> 原文：<https://developers.redhat.com/blog/2021/04/22/containerize-net-for-red-hat-openshift-windows-containers-and-net-framework>

使用并瞄准微软[的开发者。NET](/topics/dotnet) 框架不再是开发[基于容器的应用](/topics/containers)的局外人。无论是移植现有的应用程序(例如，在 IIS 中运行的网站)还是创建新的[微服务](/topics/microservices)，或者介于两者之间的某个地方，现在都可以部署了——多亏了 Windows 容器。NET 框架应用程序到您的 [Kubernetes](/topics/kubernetes) 或 [Red Hat OpenShift](/products/openshift/overview) 集群。这篇文章探讨了跑步的选择。OpenShift 群集中 Windows 容器中的. NET Framework 应用程序。

**注意**:这篇文章是介绍[三种集装箱化方法的系列文章的一部分。红帽 OpenShift](/blog/2021/03/16/three-ways-to-containerize-net-applications-on-red-hat-openshift/) 上的. NET 应用。上一篇文章介绍了用于。网芯。

## Windows 容器？

一堂快速的历史课是必要的。虽然 [Linux](https://developers.redhat.com/topics/linux) 容器可以一直追溯到 1979 年创建的`chroot`系统调用(是的，你没看错:1979)，但是第一个公认的 Linux 容器的完整实现始于 2008 年的 LXC——Linux 容器。两个 LXC 实现 [Warden](https://github.com/cloudfoundry-attic/warden) 和 [LMCTFY](https://github.com/google/lmctfy) 取得了些许成功，但 Linux 容器真正腾飞是在 2013 年 Docker 的引入。随后，诸如[安全性](/topics/security)、可伸缩性、网络等问题不断涌现，并随着时间的推移而不断改进。Linux 容器已经达到了被接受和成熟的程度，可以说它们正在成为软件开发的新标准。

Windows 容器被引入到。随着 Windows Server 2016 的发布，允许开发人员构建、管理和处理。NET 框架应用程序就像 Linux 容器一样。像`docker build`和`docker run`这样的命令在 Windows 和 Linux 上是相同的。唯一的区别是底层的操作系统。

Red Hat 宣布在 2020 年 12 月下旬在 OpenShift 中全面提供(GA)和支持 Windows 容器。这意味着——我在重复我自己，因为这实在是太神奇了——你可以建立你的形象。NET 框架应用程序(如运行在 IIS 上的网站)并在 OpenShift 集群中运行。

只有一点需要考虑:您的集群中需要一个 Windows 节点。

## 在 OpenShift 中运行 Windows 容器

操作人员，请注意:为了在 OpenShift 中运行 Windows 容器，您需要一个包含能够运行 Windows 容器的 Windows 节点的集群。那是在 OpenShift 中运行 Windows 容器的“神奇调味汁”。目前，Windows Server 2019 是运行 Windows 容器的最佳选择。

开发者们，你们轻松了。作为比特的构建者，你不会真的看到太大的区别；您将创建您的应用程序，构建一个映像，它将被部署到 OpenShift。一件好事是，你不必担心应用程序运行在同一个端口上。OpenShift 建立在 Kubernetes 之上，Kubernetes 自动分配端口，并跟踪它(Kubernetes)公开的内容和您的应用程序使用的内容之间的映射。

## 成功的秘诀

一旦你有了一个能够运行 Windows 容器的集群，我已经创建了一个 [GitHub 仓库](https://github.com/redhat-developer-demos/netcandystore) (repo ),里面有代码和指令，让你尝试一下这项激动人心的技术。在那之前，你可以在这里阅读和跟随。有点像在你真正做那顿美妙的晚餐之前阅读一份食谱。

超级棒的奖励材料提醒:repo 包括在您的 OpenShift 集群中创建和构建 Microsoft SQL Server 数据库*的指令、脚本和数据，因为为什么不在晚餐时享用甜点呢？*

## 构建 Windows 容器映像

要构建 Windows 容器映像，我们需要以下组件:

1.  a 已编制。NET 框架应用程序—在这种情况下，是一个在 IIS 中运行的网站
2.  构建映像的配置文件“Dockerfile”，它将所有内容放在一起
3.  构建图像的命令
4.  一个图像注册表，我们可以在其中存储图像，我们最终会将它放入我们的 OpenShift 集群中

### 编译后的应用程序

我们正在建立一个名为“网络糖果店”的网站，这是我们的初创公司需要尽快启动并运行的 MVP(最小可行项目)。在这一点上，应用程序还没有完全发挥作用，但是我们希望立即开始构建和部署，并随着我们的进展进行微调。

使用我前面提到的 Git repo，我们将在 Visual Studio 中使用解决方案(`netcandystore.sln`)文件，如图 1 所示。一旦到了那里，我们可以使用**发布**选项来创建所需的位——二进制文件`netcandystore.dll`。

[![visual studio publish dialog box](img/fa0c213502fa4b401437c179532c3758.png "netCandyStore_publish")](/sites/default/files/blog/2021/03/netCandyStore_publish.png)Figure 1

Figure 1: The Publish Dialog box in Visual Studio.

该目标位置被复制并粘贴到我们的映像构建配置文件“Dockerfile”中

### 文档文件

现在，我们需要存储在文件“Dockerfile”中的`docker build`命令的指令。通常，除了语法，这只是所谓的“Dockerfile”，所以我们将继续。以下是 Dockerfile 文件的内容:

```
# The `FROM` instruction specifies the base image. You are
# extending the `microsoft/aspnet` image.

FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8

# The final instruction copies the site you published earlier into the container.
COPY ./bin/app.publish/ /inetpub/wwwroot

```

基本上，我们只需要做两件事:使用来自微软的基本映像，并将我们的二进制文件复制到新构建的映像。这是我见过的最简单的 docker 文件。

### 构建图像的命令

所有这些就绪后，我们使用`docker build`命令来构建图像。对于我们图像的名称和标签，我将使用指向图像注册中心的完全限定名称，稍后我将把图像放入该注册中心。我发送给 OpenShift 的命令将从该注册表中提取。如果你使用我之前提到的的[git repo 中的指令，你将使用](https://github.com/donschenck/netcandystore)[相同的图像](https://quay.io/repository/donschenck/netcandystore?tag=latest&tab=tags)。

记住这一点，下面是如何构建将在 OpenShift 中运行的 Windows 容器映像:

```
docker build -t quay.io/donschenck/netcandystore:2021mar8.1 .
```

### 存储图像的图像注册表

现在，在登录我的 quay.io 帐户后，我可以运行以下命令来使图像注册表可用:

```
docker push quay.io/donschenck/netcandystore:2021mar8.1
```

许可呢？简单地说:许可依赖于运行容器的主机。你可以在[这个微软常见问题页面](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/faq)上找到更多信息。

## 在 Windows 容器中部署映像

该出发了。我们准备在 OpenShift 上将这个映像部署到我们的 Windows 容器主机上，并开始享受我们的成果。NET 框架劳动。但是有一个小问题:如果您尝试部署到您的集群，部署可能会因超时错误而崩溃。有一个简单的解决方法，我已经在我的 Git repo 中包含了一个例子。诀窍是将有点大的 Windows 服务器映像(5.25 GB)从节点本身转移到集群的 Windows 节点。另外，如果您在同一个 Windows 节点上运行集群中的其他 Windows 容器，它们可以使用相同的服务器映像。换句话说，您可能只需要进行一次“预加载”。

这一步的细节在我的 Git 回购上，这里就不赘述了。概述如下:找到 Windows 节点的名称，并使用 SSH 在其中运行`docker pull`命令。一旦完成这一步——需要两三分钟——剩下的就是典型的 OpenShift 操作:创建一个指向应用程序映像的部署、一个关联的服务和一个公开它的路径。

## 结论:遵循的指南

如果你想遵循一步一步的指南，包括 Windows 容器的代码，在 OpenShift 上安装 SQL Server，以及安装. NET 5 ( [)。NET Core](/topics/dotnet) 应用程序运行在 Linux 容器中，遵循这个 repo:【https://github.com/redhat-developer-demos/netcandystore】。

*Last updated: October 14, 2022*