# 没有守护进程的容器:RHEL 7.6 和 RHEL 8 中提供的 Podman 和 Buildah

> 原文：<https://developers.redhat.com/blog/2018/11/20/buildah-podman-containers-without-daemons>

Kubernetes 安装可能很复杂，有多个运行时依赖项和运行时引擎。 [CRI-O](http://cri-o.io/) 的创建是为了给 Kubernetes 提供一个轻量级的运行时，它在集群和运行时之间增加了一个抽象层，允许各种 OCI 运行时技术。然而，您仍然存在依赖集群中的守护进程进行构建的问题——也就是说，如果您使用集群进行构建，您仍然需要 Docker 守护进程。

输入 *Buildah。Buildah 允许您拥有一个 Kubernetes 集群，运行时和构建时都不需要任何 Docker 守护进程。非常好。但是如果事情出错了呢？如果您想对集群中的容器进行故障排除或调试，该怎么办？Buildah 并不是为此而构建的，你需要的是一个用于处理容器的客户端工具，而我想到的是 Docker CLI——但之后你又会回到使用守护进程。*

这就是波德曼介入的地方。波德曼允许你在不依赖守护进程的情况下执行所有的 Docker 命令*。要查看 Podman 替换`docker`命令的示例，请参见 Alessandro Arrichiello 的[Podman](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/)简介和 Doug Tidwell 的[Podman——下一代 Linux 容器工具](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)。*

使用 Podman，您可以在 Kubernetes 集群中运行、构建(为此它在幕后调用 Buildah)、修改和故障排除容器。有了这两个项目，您就有了一个全面的解决方案来满足您的 OCI 集装箱形象和集装箱需求。

Buildah 和 Podman 可以通过`yum install buildah podman`轻松安装在 RHEL 7.6 和 RHEL 8 上。它们也可以在 Fedora 和最新的 Linux 平台上使用。在 RHEL 7.6 上，确保您启用了`rhel-7-server-extras-rpms`回购。

## 什么时候使用 Buildah，什么时候使用 Podman

Buildah 和 Podman 是驻留在 GitHub 上的两个互补的开源项目: [Buildah(容器/buildah)](https://github.com/containers/buildah) 和 [Podman(容器/libpod)](https://github.com/containers/libpod) 。Buildah 和 Podman 都是命令行工具，用于处理 OCI 图像和容器。这两个项目是相关的，但在专业上有所不同。

Buildah 专注于构建 OCI 图像。Buildah 的命令复制 Dockerfile 文件中的所有命令。Buildah 的目标也是提供一个底层的 coreutils 接口来构建容器映像，允许人们在不需要 Dockerfile 的情况下构建容器。Buildah 的另一个目标是允许您使用其他脚本语言构建容器映像，而不需要守护进程。

波德曼专门研究所有的命令和功能，帮助您维护和修改这些 OCI 集装箱图像，如拉和标记。它还允许您创建、运行和维护这些容器。如果您可以在 Docker CLI 中执行某个命令，那么您也可以在 Podman CLI 中执行相同的命令。事实上，你可以在你的机器上用`podman`代替`docker`，然后你可以构建、创建和维护容器映像和容器，而不需要守护进程，就像你一直做的那样。

虽然 Podman 使用 Buildah 的构建功能来创建容器映像，但这两个项目有所不同。Podman 和 Buildah 之间的主要区别是他们对容器的概念。Podman 允许用户创建传统的容器，这些容器的目的是在容器的整个生命周期中被控制(暂停、检查点/恢复等)。而 Buildah 容器实际上是为了允许内容添加到容器中而创建的*映像*。每个项目都有一个不共享的容器的单独内部表示。因此，你不能从 Buildah 中看到 Podman 容器，反之亦然。然而，Buildah 和 Podman 之间容器映像的内部表示是相同的。考虑到这一点，一方创建、拉取或修改的任何容器图像都可以被另一方看到和使用。

两个项目之间的一些命令明显重叠，但在某些情况下行为略有不同。下表说明了项目之间有一些重叠的命令。

| 命令 | 波德曼行为 | buildhr 行为 |
| --- | --- | --- |
| `build` | 通话次数`buildah bud` | 提供模拟 Docker 的 build 命令的 build-using-dockerfile (bud)命令。 |
| `commit` | 将 Podman 容器提交到容器映像中。不适用于 Buildah 容器。提交后，生成的图像可以由 Podman 或 Buildah 使用。 | 将 Buildah 容器提交到容器映像中。不适用于波德曼集装箱。提交后，生成的映像可以由 Buildah 或 Podman 使用。 |
| `mount` | 安装一个 Podman 容器。不适用于 Buildah 容器。 | 安装 Buildah 容器。不适用于波德曼集装箱。 |
| `pull`和`push` | 从容器映像注册表中提取或推送映像。功能上与 Buildah 相同。 | 从容器映像注册表中提取或推送映像。功能上和波德曼一样。 |
| `run` | 以与`docker run`相同的方式在新容器中运行流程。 | 以与 Dockerfile 文件中的 RUN 命令相同的方式运行容器。 |
| `rm` | 移除 Podman 容器。不适用于 Buildah 容器。 | 移除 Buildah 容器。不适用于波德曼集装箱。 |
| `rmi`、`images`、`tag` | 两个项目相当。 | 两个项目相当。 |
| `containers`和`ps` | `ps`用于列出波德曼集装箱。`containers`命令不存在。 | containers 用于列出 Buildah 容器。`ps`命令不存在。 |

总结这两个项目的区别的一个快速简单的方法是,`buildah run`命令在 Dockerfile 文件中模拟 RUN 命令，而`podman run`命令在功能上模拟`docker run`命令。

Buildah 是创建 OCI 映像的有效方法，而 Podman 允许您使用熟悉的容器 CLI 命令在生产环境中管理和维护这些映像和容器。他们一起形成了一个强大的基础，以支持您的 OCI 集装箱形象和集装箱的需求。更好的是，它们都是开源项目，非常欢迎您为其中一个或两个项目做出贡献。希望在那里见到你！

这篇文章的一个[早期版本出现在](https://podman.io/blogs/2018/10/31/podman-buildah-relationship.html) [podman.io 博客](https://podman.io/blogs/)上。

## 额外资源

*   [Red Hat Enterprise Linux 7.6 中的 Podman 简介](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/)
*   [pod man——下一代 Linux 容器工具](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)
*   [使用 Podman 管理集装箱化系统服务](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/)
*   [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)
*   [crictl vs poder man](https://blog.openshift.com/crictl-vs-podman/)
*   构建、运行和管理容器- Red Hat Enterprise Linux 8 文档
*   上游社区站点: [buildah.io](https://buildah.io/) 和 [podman.io](https://podman.io/)
*   GitHub:[Buildah(containers/Buildah)](https://github.com/containers/buildah)和[pod man(containers/lib pod)](https://github.com/containers/libpod)

*Last updated: January 14, 2022*