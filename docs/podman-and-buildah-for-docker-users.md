# 面向 docker 用户的 podman 和 buildhr

> 原文：<https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users>

最近有人在 Twitter 上要求我为熟悉 Docker 的人更好地解释一下 [Podman](https://github.com/containers/libpod) 和 [Buildah](https://github.com/containers/libpod) 。虽然有很多博客和教程，我将在后面列出，但我们社区并没有集中解释 Docker 用户如何从 Docker 迁移到 Podman 和 Buildah。还有 Buildah 扮演什么角色？波德曼是否在某些方面存在缺陷，以至于我们需要波德曼和 Buildah 来取代 Docker？

本文回答了这些问题，并展示了如何迁移到 Podman。

## Docker 是如何工作的？

首先，我们先明确 Docker 是如何工作的；这将有助于我们理解波德曼和 Buildah 的动机。如果你是一个 Docker 用户，你应该知道有一个守护进程必须运行来服务你所有的 Docker 命令。我不能声称理解这背后的动机，但我想象这在当时似乎是一个伟大的想法，在一个地方做 Docker 做的所有很酷的事情，并为未来的发展提供一个有用的 API。在下图中，我们可以看到 Docker 守护程序提供了以下所需的所有功能:

*   从图像注册表中提取和推送图像
*   在本地容器存储中制作图像的副本，并向这些容器添加层
*   提交容器并从主机存储库中删除本地容器映像
*   要求内核运行具有正确名称空间和 cgroup 的容器，等等。

本质上，Docker 守护进程完成了注册表、映像、容器和内核的所有工作。Docker 命令行界面(CLI)要求守护程序代表您完成这项工作。

[![How does Docker Work -- Docker architecture overview](img/97ddc680a10e00c25f12aa9087bee3ba.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/02/fig1.png)

本文没有详细讨论 Docker 守护进程的优缺点。有很多东西支持这种方法，我可以理解为什么在 Docker 的早期，这种方法很有意义。简单地说，随着使用量的增加，Docker 用户关注这种方法有几个原因。举几个例子:

*   单个流程可能是单点故障。
*   这个进程拥有所有的子进程(运行容器)。
*   如果出现故障，就会有孤立的进程。
*   构建容器导致了安全漏洞。
*   所有 Docker 操作都必须由具有相同完全根权限的用户执行。

可能还有更多。这些问题是否已经得到解决，或者您是否不同意这种描述，这不是本文要讨论的问题。我们社区认为，波德曼已经解决了许多这样的问题。如果你想利用波德曼的改进，那么这篇文章是给你的。

Podman 方法只是通过 runC 容器运行时进程(不是守护进程)直接与映像注册表、容器和映像存储以及 Linux 内核进行交互。

[![Podman architectural approach](img/d6dfecbf73df45d2d2d4089aa7f05ee1.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/02/fig2.png)

既然我们已经讨论了一些动机，现在是时候讨论这对迁移到 Podman 的用户意味着什么。这里有一些东西需要打开，我们将分别讨论:

*   你安装的是波德曼而不是多克。您不需要像 Docker 守护进程那样启动或管理守护进程。
*   您在 Docker 中熟悉的命令对 Podman 同样有效。
*   Podman 将其容器和图像存储在与 Docker 不同的地方。
*   波德曼和码头工人图像兼容。
*   对于 Kubernetes 环境来说，Podman 不仅仅是 Docker。
*   什么是 Buildah？我为什么需要它？

## 安装 Podman

如果你现在正在使用 Docker，你可以在决定转换时删除它。然而，你可能希望在试用波德曼的时候保留 Docker。有一些有用的[教程](https://github.com/containers/libpod/blob/master/docs/tutorials/podman_tutorial.md)和一个很棒的[演示](https://github.com/containers/Demos/tree/master/building/buildah_intro)，你可能希望先浏览一下，这样你就能更好地理解这个转变。演示中的一个例子需要 Docker 来显示兼容性。

要在[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/)7.6 或更高版本上安装 Podman，请使用以下命令:如果您使用 Fedora，则将`yum`替换为`dnf`:

```
# yum -y install podman
```

## Podman 命令与 Docker 命令相同

在构建 Podman 时，目标是确保 Docker 用户能够轻松适应。所以所有你熟悉的命令也存在于波德曼。事实上，有人声称，如果你有运行 Docker 的现有脚本，你可以为`podman`创建一个`docker`别名，并且你的所有脚本都应该工作(`alias docker=podman`)。试试看。当然要先停 Docker(`systemctl stop docker`)。你可以安装一个名为`podman-docker`的包来帮你转换。它在`/usr/bin/docker`删除了一个脚本，该脚本使用相同的参数执行 Podman。

你熟悉的命令— `pull`、`push`、`build`、`run`、`commit`、`tag`等。—都和波德曼一起存在。有关更多信息，请参见 Podman 的[手册页。一个显著的区别是，Podman 在一些命令中添加了一些便利标志。比如，波德曼为`podman rm`和`podman rmi`增加了`--all` ( `-a`)标志。许多用户会发现这非常有用。](https://github.com/containers/Demos/tree/master/building/buildah_intro)

您也可以在 Fedora 上的 Podman 1.0 中从您的普通非 root 用户运行 Podman。RHEL 支持针对 7.7 和 8.1 版本。用户空间安全性的增强使这成为可能。作为普通用户运行 Podman 意味着，缺省情况下，Podman 会将图像和容器存储在用户的主目录中。这将在下一节中解释。更多关于 Podman 如何作为非 root 用户运行的信息，请查看 Dan Walsh 的文章:[无根 Podman 如何工作？](https://opensource.com/article/19/2/how-does-rootless-podman-work)

[![](img/c2b9f06305c4b924feb10978643812c5.png)](https://che.openshift.io/f?url=https://raw.githubusercontent.com/redhat-developer/devfile/master/getting-started/java-maven/devfile.yaml/?sc_cid=7013a000002D1quAAC)

## 搬运工和集装箱图像

当你第一次输入`podman images`时，你可能会惊讶地发现你没有看到任何你已经拉下的 Docker 图片。这是因为波德曼的本地存储库在`/var/lib/containers`而不是`/var/lib/docker`。这不是任意的改变；这种新的存储结构是基于开放集装箱倡议(OCI)的标准。

2015 年，Docker、Red Hat、CoreOS、SUSE、Google 和 Linux 容器行业的其他领导者创建了开放容器计划，以便提供一个独立的机构来管理定义容器映像和运行时的标准规范。为了保持这种独立性，在 [GitHub](https://github.com/containers) 上创建了[容器/映像](https://github.com/containers/image)和[容器/存储](https://github.com/containers/storage)项目。

因为您可以在不是 root 的情况下运行`podman`，所以需要有一个单独的地方让`podman`可以写图像。Podman 使用用户主目录中的一个存储库:`~/.local/share/containers`。这避免了使`/var/lib/containers`成为全球可写的或者其他可能导致潜在安全问题的实践。这也确保了每个用户都有独立的容器和映像集，并且都可以在同一台主机上同时使用 Podman，而不会互相影响。当用户完成他们的工作后，他们可以将他们的图片推送到一个公共的注册中心与他人分享。

来到 Podman 的 Docker 用户发现，当你只想重新开始时，知道这些位置对于调试和重要的`rm -rf /var/lib/containers`是有用的。然而，一旦你开始使用波德曼，你可能会开始使用新的`-all`选项来代替`podman rm`和`podman rmi`。

## 容器映像在 Podman 和其他运行时之间是兼容的

尽管本地存储库有了新的位置，但 Docker 或 Podman 创建的图像符合 OCI 标准。波德曼可以从像 [Quay.io](https://quay.io/) 和 Docker hub 这样的受欢迎的集装箱注册中心以及私人注册中心推送和提取数据。例如，您可以从 Docker hub 获取最新的 Fedora 图像，并使用 Podman 运行它。不指定注册表意味着 Podman 将默认按照列出的顺序搜索`registries.conf`文件中列出的注册表。未修改的`registries.conf`文件意味着它将首先在 Docker hub 中查找。

```
$ podman pull fedora:latest
$ podman run -it fedora bash
```

由 Docker 推送到图像注册中心的图像可以由 Podman 下载并运行。例如，我使用 Docker 创建并使用 Docker 推送到我的 [Quay.io 存储库](https://quay.io/repository/ipbabble) (ipbabble)的一个映像(myfedora)可以通过 Podman 如下提取并运行:

```
$ podman pull quay.io/ipbabble/myfedora:latest
$ podman run -it myfedora bash
```

Podman 在其命令行`push`和`pull`命令中提供了将图像从`/var/lib/docker `优雅地移动到 `/var/lib/containers`的功能，反之亦然。例如:

```
$ podman push myfedora docker-daemon:myfedora:latest
```

显然，省略上面的`docker-daemon`将默认为推送到 Docker hub。使用`quay.io/myquayid/myfedora`将图像推送到 Quay.io 注册处(下面的`myquayid`是你的个人 Quay.io 账户):

```
$ podman push myfedora quay.io/myquayid/myfedora:latest
```

如果您准备删除 Docker，您应该关闭守护进程，然后使用您的包管理器删除 Docker 包。但是首先，如果你有你用 Docker 创建的图片，并且你想保存，你应该确保这些图片被推送到一个注册表中，这样你就可以在以后把它们下载下来。或者您可以使用 Podman 将每个图像(例如 fedora)从主机的 Docker 存储库中拖到 Podman 位于 OCI 的存储库中。使用 RHEL，您可以运行以下内容:

```
# systemctl stop docker
# podman pull docker-daemon:fedora:latest
# yum -y remove docker  # optional
```

## 波德曼帮助用户转移到 Kubernetes

Podman 提供了一些额外的特性来帮助 Kubernetes 环境中的开发人员和操作人员。Podman 提供了 Docker 中没有的额外命令。如果你熟悉 Docker，并且正在考虑使用 Kubernetes/ [OpenShift](http://openshift.com/) 作为你的容器平台，那么 Podman 可以帮助你。

Podman 可以使用`podman generate kube`基于运行的容器生成一个 Kubernetes YAML 文件。命令`podman pod`可以用来帮助调试正在运行的 Kubernetes pods 以及标准的容器命令。有关波德曼如何帮助你过渡到 Kubernetes 的更多细节，请参见 Brent Baude 的以下文章:[波德曼现在可以轻松过渡到 Kubernetes 和 CRI-O](https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml/) 。

## 什么是 Buildah，我为什么要使用它？

Buildah 实际上是第一名。也许这就是为什么一些 Docker 用户感到有点困惑。为什么这些波德曼福音传道者也谈论 Buildah？波德曼不做构建吗？

Podman 确实做构建，对于那些熟悉 Docker 的人来说，构建过程是相同的。您可以使用 Dockerfile 使用`podman build`来构建，或者您可以运行一个容器并进行大量更改，然后将这些更改提交给一个新的图像标签。Buildah 可以被描述为与创建和管理容器映像相关的命令的超集，因此，它对映像有更细粒度的控制。Podman 的`build`命令包含 Buildah 功能的一个子集。它使用与 Buildah 相同的代码进行构建。

使用 Buildah 最有效的方法是编写 Bash 脚本来创建图像——就像编写 docker 文件一样。

我喜欢用下面的方式来思考进化。当 Kubernetes 基于 OCI 运行时规范迁移到 [CRI-O](https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml/) 时，不需要运行 Docker 守护进程，因此，不需要在 Kubernetes 集群中的任何主机上安装 Docker 来运行 pod 和容器。Kubernetes 可以调用 CRI-O，也可以直接调用 runC。这又会启动容器进程。但是，如果我们想要使用相同的 Kubernetes 集群来进行构建，就像 OpenShift 集群的情况一样，那么我们需要一个新的工具来执行构建，它不需要 Docker 守护进程，也不需要安装 Docker。这样一个基于`containers/storage`和`containers/image`项目的工具，也将消除在构建期间开放 Docker 守护进程套接字的安全风险，这是许多用户所担心的。

Buildah(因 Dan Walsh 发音为“builder”时的波士顿口音而得名)符合这一要求。有关 Buildah 的更多信息，请参见 [buildah.io](https://buildah.io/) 并具体参见博客和教程部分。

关于 Buildah，从业者需要了解几件额外的事情:

1.  它允许更好地控制图像层的创建。这是许多容器用户长期以来一直要求的功能。向单个层提交许多更改是可取的。
2.  Buildah 的`run`命令和 Podman 的`run`命令不一样。因为 Buildah 是用于构建图像的，`run`命令*本质上与 Dockerfile* `RUN` *命令*相同。事实上，我记得这是明确的一周。我愚蠢地抱怨说，我正在尝试的一些端口或挂载没有像我预期的那样工作。Dan ( [@rhatdan](https://twitter.com/rhatdan) )加入进来说 Buildah 不应该以那种方式支持运行中的容器。没有端口映射。无卷装。那些旗子被拿走了。相反，`buildah run`用于运行特定的命令，以帮助构建容器映像，例如`buildah run dnf -y install nginx`。
3.  Buildah 可以从头开始构建图像，也就是说，图像中根本没有任何内容。没什么。事实上，查看由`buildah from scratch`命令创建的容器存储会产生一个空目录。这对于创建仅包含运行应用程序所需的包的轻量级映像非常有用。

临时构建的一个很好的用例是考虑 Java 应用程序的开发映像与登台或生产映像。在开发过程中，Java 应用程序容器映像可能需要 Java 编译器和 Maven 以及其他工具。但是在生产中，您可能只需要 Java 运行时和您的包。顺便说一下，你也不需要一个包管理器，比如 DNF/百胜或者 Bash。对于这个用例，Buildah 是一个强大的 CLI。见下图。有关更多信息，请参见[为 Kubernetes 构建 Buildah 容器映像](https://buildah.io/blogs/2018/03/01/building-buildah-container-image-for-kubernetes.html)以及这个 [Buildah 介绍演示](https://github.com/containers/Demos/tree/master/building/buildah_intro)。

[![Buildah is a powerful CLI](img/874c9c326062379d4d9df0560d379be5.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/02/fig3.png)

回到进化的故事...既然我们已经用 CRI-O 和 runC 解决了 Kubernetes 的运行时问题，并且用 Buildah 解决了构建问题，那么在 Kubernetes 主机上仍然需要 Docker 还有一个原因:调试。如果没有工具，我们如何在主机上调试容器问题呢？我们需要安装 Docker，然后我们又回到了我们在主机上使用 Docker 守护进程开始的地方。波德曼解决了这个问题。

波德曼成为解决两个问题的工具。它允许操作人员使用他们熟悉的命令检查容器和图像。而且还为开发者提供了同样的工具。因此，Docker 用户、开发人员或操作人员可以迁移到 Podman，完成他们熟悉的所有有趣的任务，还可以做更多的事情。

## 结论

我希望这篇文章对您有用，能够帮助您自信而成功地迁移到使用 Podman(和 Buildah)。

有关更多信息:

*   [Podman.io](https://podman.io/) 和 [Buildah.io](https://buildah.io/) 项目网站
*   [github.com/containers](https://github.com/containers)项目(参与进来，了解来源，看看正在开发什么):
    *   [利波德](https://github.com/containers/libpod)(波德曼)
    *   [buildah](https://github.com/containers/buildah)
    *   [图像](https://github.com/containers/image)(用于处理 OCI 容器图像的代码)
    *   [存储](https://github.com/containers/storage)(本地图像和容器存储代码)

## 相关文章

*   没有守护进程的容器:RHEL 7.6 和 RHEL 8 测试版中的 Podman 和 Buildah
*   [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)
*   使用 Podman 管理容器化系统服务(使用 systemd 管理您的 podman 容器)
*   [为 Kubernetes 构建 Buildah 容器映像](https://buildah.io/blogs/2018/03/01/building-buildah-container-image-for-kubernetes.html)
*   [波德曼现在可以轻松过渡到 Kubernetes 和 CRI-O](https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml/)
*   [容器运行时的安全考虑](https://developers.redhat.com/blog/2018/12/19/security-considerations-for-container-runtimes/)(来自 KubeCon 2018 的 Dan Walsh 演讲视频)
*   [通过 OpenShift 使用容器进行物联网边缘开发和部署:第 1 部分](https://developers.redhat.com/blog/2019/01/31/iot-edge-development-and-deployment-with-containers-through-openshift-part-1/)(使用 podman、qemu、binfmt_misc 和 Ansible 在 OpenShift 上构建和测试 ARM64 容器)

*Last updated: June 17, 2021*