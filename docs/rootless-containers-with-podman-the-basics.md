# Podman 的无根容器:基础

> 原文：<https://developers.redhat.com/blog/2020/09/25/rootless-containers-with-podman-the-basics>

作为开发人员，您可能听说过很多关于容器的事情。一个 [*容器*](https://developers.redhat.com/topics/containers) 是一个软件单元，它提供了一个打包机制，该机制抽象出代码及其所有依赖关系，以使应用构建快速可靠。试验容器的一个简单方法是使用 Pod Manager 工具( [Podman](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools) )，这是一个无守护进程的开源 Linux 原生工具，它提供了一个类似于 docker 容器引擎的命令行界面(CLI)。

在这篇文章中，我将解释使用容器和 Podman 的好处，介绍无根容器以及它们为什么重要，然后用一个例子向你展示如何使用无根容器和 Podman。在我们深入实现之前，让我们回顾一下基础知识。

## 为什么是集装箱？

使用容器将您的应用程序与运行它们的各种计算环境隔离开来。它们变得越来越受欢迎，因为它们帮助开发人员专注于应用程序逻辑及其依赖关系，将它们绑定在一个单元中。运营团队也喜欢容器，因为他们可以专注于管理应用程序，包括部署，而不用担心软件版本和配置等细节。

容器在操作系统(OS)级别进行虚拟化。这使它们变得轻量级，不像虚拟机那样在硬件级别进行虚拟化。简而言之，使用容器的优势如下:

*   低硬件占用空间
*   环境隔离
*   快速部署
*   多环境部署
*   复用性

## 为什么是波德曼？

使用 Podman 可以很容易地找到、运行、构建、共享和部署使用[开放容器倡议](https://opencontainers.org/) (OCI)兼容的容器和容器映像的应用程序。波德曼的优势如下:

*   是*无梦*；与 docker 不同，它不需要守护进程。
*   它让你控制容器的层；有时候，你想要单层，有时候你需要 12 层。
*   它对容器使用 fork/exec 模型，而不是客户机/服务器模型。
*   它允许您以非 root 用户的身份运行容器，因此您不必在主机上给用户 root 权限。这显然不同于客户机/服务器模型，在客户机/服务器模型中，您必须打开一个套接字，连接到以 root 用户身份运行的特权守护进程，才能启动容器。

## 为什么是无根容器？

*无根容器*是没有管理员权限的用户可以创建、运行和管理的容器。无根容器有几个优点:

*   他们增加了新的安全层；即使容器引擎、运行时或 orchestrator 受损，攻击者也不会获得主机上的 root 权限。
*   它们允许多个非特权用户在同一台机器上运行容器(这在[高性能计算环境](https://www.redhat.com/en/blog/podman-paves-road-running-containerized-hpc-applications-exascale-supercomputers)中尤其有利)。
*   它们允许嵌套容器内部的隔离。

为了更好地理解这些优势，考虑传统的资源管理和调度系统。这种类型的系统应该由没有权限的用户运行。从安全角度来看，特权越少越好。使用无根容器，您可以像运行任何其他进程一样运行容器化的进程，而不需要提升任何用户的权限。没有守护进程；波德曼创建了一个子进程。

## 示例:使用无根容器

让我们开始使用 Podman 的无根容器。我们将从基本的设置和配置开始。

### 系统需求

我们需要[Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux)(RHEL)7.7 或更高版本来实现。假设您有了这些，我们就可以开始配置示例了。

### 配置

首先，通过输入以下命令在您的机器上安装`slirp4netns`和 Podman:

```
$ yum install slirp4netns podman -y

```

我们将使用`slirp4netns`以完全无根(或无特权)的方式将网络名称空间连接到互联网。

安装完成后，增加用户名称空间的数量。使用以下命令:

```
$ echo “user.max_user_namespaces=28633” > /etc/sysctl.d/userns.conf 	 
$ sysctl -p /etc/sysctl.d/userns.conf

```

接下来，创建一个新的用户帐户并命名为。在本例中，我的用户帐户名为 Red Hat:

```
$ useradd -c “Red Hat” redhat

```

使用以下命令为新帐户设置密码(注意，您必须插入自己的密码):

```
$ passwd redhat

```

该用户现在被自动配置为能够使用 Podman 的无根实例。

### 连接到用户

现在，尝试以刚刚创建的用户身份运行 Podman 命令。不要使用`su -`，因为该命令没有设置正确的环境变量。相反，您可以使用任何其他命令来连接到该用户。这里有一个例子:

```
$ ssh redhat@localhost

```

### 拉一个红帽企业 Linux 映像

登录后，尝试使用`podman`命令提取 RHEL 映像(注意`ubi`代表[通用基础映像](https://developers.redhat.com/products/rhel/ubi)):

```
$ podman pull ubi7/ubi

```

如果您需要有关映像的更多信息，请运行以下命令:

```
$ podman run ubi7/ubi cat /etc/os-release

```

要检查上述命令生成的映像以及系统上的任何其他映像，请运行命令:

```
$ podman images

```

无根用户也可以从这些图像中创建一个容器，但是我将在另一篇文章中讨论。

### 检查无根配置

最后，验证您的无根配置是否设置正确。运行以下命令显示如何将 uid 分配给用户命名空间:

```
$ podman unshare cat /proc/self/uid_map

```

## 结论

本文演示了如何用 Podman 设置无根容器。以下是使用无根容器的一些技巧:

*   作为非根容器用户，容器映像存储在您的主目录下(例如，`$HOME/.local/share/containers/storage`)，而不是`/var/lib/containers`。这个目录方案确保您有足够的存储空间来存放您的主目录。
*   运行无根容器的用户被授予特殊权限，可以使用一系列用户和组 id 在主机系统上运行。否则，他们对主机上的操作系统没有 root 权限。
*   在无根帐户中以 root 身份运行的容器可以在其自己的命名空间中打开特权功能。但是这并没有提供任何特权来访问主机上受保护的特性(除了拥有额外的 uid 和 GID 之外)。

点击这里了解更多关于[使用 Podman 设置无根容器的信息。](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html-single/managing_containers/index#set_up_for_rootless_containers)

*Last updated: April 1, 2022*