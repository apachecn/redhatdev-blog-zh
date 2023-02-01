# 。红帽平台上的 NET Core

> 原文：<https://developers.redhat.com/blog/2020/01/17/net-core-on-red-hat-platforms>

在这篇文章中，我们看看各种方式。NET Core 在 Red Hat 平台上可用。我们首先概述可用的平台，然后展示如何安装。每个上面都有网芯。

## 平台概述

先说概述。如果您已经熟悉这些平台，您可以跳到介绍特定平台的章节。

### 操作系统

从操作系统的角度来看，我们将看到四个发行版:

*   Fedora 是一个社区维护的发行版，发展迅速。对于希望获得最新开发工具和最新内核的开发人员来说，这是一个很好的选择。
*   [红帽企业版 Linux](http://developers.redhat.com/rhel8/) (RHEL)是红帽基于 Fedora 的长期支持(LTS)发行版。
*   [CentOS](https://www.centos.org/) 是社区维护的 Red Hat Enterprise Linux 的下游重建。
*   [CentOS Stream](https://www.redhat.com/en/blog/transforming-development-experience-within-centos) 是未来 Red Hat Enterprise Linux 版本的滚动预览版。

所有这些发行版*都是免费的，就像 Freedom* 中的一样，所以软件是可用的，并且可以改变。软呢帽和 CentOS 没有成本(*免费啤酒*)。对于 Red Hat Enterprise Linux，您需要向 Red Hat 支付支持订阅费。然而，出于开发目的，您可以使用[免费 Red Hat Developer subscription](https://developers.redhat.com/blog/2016/03/31/no-cost-rhel-developer-subscription-now-available/) 并免费下载 Red Hat Enterprise Linux 以及我们的其他产品和工具。

### 容器

大多数公共云供应商支持创建基于 Red Hat Enterprise Linux 和 CentOS 的虚拟机(VM)。基于 RHEL 的虚拟机由 Red Hat 支持，基于 CentOS 的虚拟机由 CentOS 社区支持。

有了容器技术，操作系统现在也打包到容器映像中。这些映像包含来自操作系统的包，但不包含内核。Red Hat 提供 Red Hat Enterprise Linux 7 和 RHEL 8 的映像。基于 Red Hat Enterprise Linux 8 的映像基于[通用基础映像](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) (UBI)。对于 Red Hat Enterprise Linux 7，有些映像是作为 UBI 映像提供的，而其他映像则是非 UBI 映像。RHEL 7 非 UBI 映像托管在 [registry.redhat.io](https://registry.redhat.io) 上，需要获取订阅凭证。UBI 图片托管在 registry.access.redhat.com[的](https://registry.access.redhat.com)，下载无需订阅。这些图片*可以在没有红帽订阅的情况下，在任何平台*上用于生产。然而，当 UBI 映像在 Red Hat Enterprise Linux 平台上运行时，订阅者可以获得 Red Hat 的支持。CentOS 社区还提供托管在[registry.centos.org](https://registry.centos.org)上的图片。

### OpenShift

对于运行基于容器的应用，红帽提供了[红帽 OpenShift](http://developers.redhat.com/openshift/) ，它基于 [Kubernetes](http://developers.redhat.com/topics/kubernetes/) 。Kubernetes 提供了跨多台机器调度容器的核心功能。OpenShift 将 Kubernetes 打包成一种可用、可部署和可维护的形式。Red Hat 提供了一个受支持的 OpenShift 版本，可以部署在内部和公共云中(例如 Azure、AWS 和 GCP)。红帽 OpenShift 基于开源，上游 [`okd`](https://www.okd.io/) 项目。要在您的开发机器上运行 OpenShift，您可以为 OpenShift 4.x 使用[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview)(CRC)，为 OpenShift 3.x 使用[Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview)(CDK)

## 。每个平台上的 NET Core

现在，我们来看看如何安装。NET 核心到每个红帽平台上。

### 。Fedora 上的 NET Core

。Fedora 上的 NET Core 是由 [Fedora 构建的。净特殊利益集团](https://fedoraproject.org/wiki/SIGs/DotNet) (SIG)。因为。NET Core 不符合 Fedora 打包指南，它是从一个单独的存储库分发的。

来安装。NET Core，您需要启用`copr`存储库并安装`sdk`包:

```
$ sudo dnf copr enable @dotnet-sig/dotnet
$ sudo dnf install dotnet-sdk-2.1

```

如果可能的话，SIG 还会构建。网芯。这些预览版本是从一个单独的存储库中分发的:

```
$ sudo dnf copr enable @dotnet-sig/dotnet-preview
$ sudo dnf install dotnet-sdk-3.1

```

有关运行的更多信息。NET Core on Fedora，参见 [Fedora。NET 文档](https://developer.fedoraproject.org/tech/languages/csharp/dotnet-installation.html)。

### 。Red Hat Enterprise Linux 7 和 CentOS 7 上的 NET Core

在红帽企业版 Linux 7 和 CentOS 7 上，。NET 核心版本被打包到它们自己的软件集合(SCL)中。这种包装方法允许。NET Core 包含比基本操作系统提供的库更新的库。

来安装。NET Core 在红帽企业 Linux 上，机器需要注册[红帽订阅管理系统](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/quick_registration_for_rhel/registering-machine-ui)。根据操作系统的特点(例如，服务器、工作站或 HPC 计算节点)，您需要启用适当的。NET 核心存储库:

```
$ sudo subscription-manager repos --enable=rhel-7-server-dotnet-rpms
$ sudo subscription-manager repos --enable=rhel-7-workstation-dotnet-rpms
$ sudo subscription-manager repos --enable=rhel-7-hpc-node-dotnet-rpms

```

接下来，需要安装 SCL 工具:

```
$ sudo yum install scl-utils

```

现在我们可以安装特定版本的。网络核心:

```
$ sudo yum install rh-dotnet21 -y

```

使用。NET 核心 SCL，我们首先需要启用它。例如，我们可以通过运行以下命令在 SCL 运行`bash`:

```
$ scl enable rh-dotnet30 bash

```

有关运行的更多信息。NET Core on Red Hat Enterprise Linux 7，参见[。网芯入门指南](https://access.redhat.com/documentation/en-us/net_core)。

### 。Red Hat Enterprise Linux 8/CentOS 8 上的 NET Core

在更新的 RHEL 8 号上。NET Core 包含在默认情况下启用的 AppStream 存储库中。的一个版本。NET Core 可以通过运行以下命令来安装:

```
$ sudo dnf install dotnet-sdk-2.1

```

有关运行的更多信息。NET Core on Red Hat Enterprise Linux 8，参见[。网芯入门指南](https://access.redhat.com/documentation/en-us/net_core)。

### 。集装箱中的净芯

每个不同的提供两个图像。NET 核心版本:包含运行. NET 核心应用程序所需的一切的运行时映像，以及包含运行时和构建应用程序所需工具的 SDK 映像。

表 1 显示了。网芯 2.1。对于其他版本，只需更改名称中的版本号。

Table 1: .NET Core images for version 2.1.

| 基础图像 | SDK/运行时映像 |
| `rhel7` | registry.redhat.io/dotnet/dotnet-21-rhel7:2.1
registry.redhat.io/dotnet/dotnet-21-runtime-rhel7:2.1 |
| `centos7` | registry.centos.org/dotnet/dotnet-21-centos7:latest
registry.centos.org/dotnet/dotnet-21-runtime-centos7:latest |
| `ubi8` | registry.access.redhat.com/ubi8/dotnet-21:2.1
registry.access.redhat.com/ubi8/dotnet-21-runtime:2.1 |

**注意:**视版本而定，图像可能(还)不适用于特定的基础。例如，在 Red Hat 创建了 Red Hat Enterprise Linux 映像之后，CentOS 社区可以使用 CentOS 映像。

下面的示例打印出中可用的 SDK 版本。网芯 2.1 `ubi8`图片:

```
$ podman run registry.access.redhat.com/ubi8/dotnet-21:2.1 dotnet --version
2.1.509

```

有关的更多信息。NET Core 图片，见 [`s2i-dotnetcore` GitHub repo](https://github.com/redhat-developer/s2i-dotnetcore) ，和[。NET Core 入门指南](https://access.redhat.com/documentation/en-us/net_core)。

### 。OpenShift 上的 NET Core

的。NET 核心映像提供了一个运行和构建的环境。NET 核心应用程序。它们还兼容 OpenShift 的[源到图像](https://docs.openshift.com/container-platform/4.2/builds/build-strategies.html#build-strategy-s2i_build-strategies) (S2I)构建策略。这个因素意味着 OpenShift 容器平台可以从源代码构建一个. NET 核心应用程序。

。使用 OpenShift CLI 客户端(`oc`)和来自 [`s2i-dotnetcore` GitHub repo](https://github.com/redhat-developer/s2i-dotnetcore) 的图像流定义文件将. NET 图像导入 OpenShift。该文件因基础映像而异，如表 2 所示。

Table 2: .NET Core image stream definitions by base image.

| 基础图像 | 图像流定义 |
| `rhel7` | https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json |
| `centos7` | https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams_centos.json |
| `ubi8` | https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams_rhel8.json |

检索 Red Hat Enterprise Linux 7 映像需要身份验证。[注册表认证](https://access.redhat.com/articles/3399531)描述了如何设置拉取密码。

可以使用`oc`导入图像流。例如，要导入基于`ubi8`的图像，运行:

```
$ oc create -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams_rhel8.json

```

**注意:**如果已经有图像流存在。NET Core，必须用`replace`而不是`create`。

`s2i-dotnetcore`存储库包含一个脚本，用于在 Windows、Linux 和 macOS 上安装这些图像流。更多信息参见[安装](https://github.com/redhat-developer/s2i-dotnetcore#installing)。该脚本还可以创建提取 Red Hat Enterprise Linux 7 映像所需的提取秘密。

一旦安装了图像流，OpenShift 就可以直接从 Git repo 构建和部署应用程序，例如:

```
$ oc new-app dotnet:3.1~https://github.com/redhat-developer/s2i-dotnetcore-ex#dotnetcore-3.1 --context-dir=app

```

有关使用的更多信息。NET Core on OpenShift，参见[中的相应章节。网芯入门指南](https://access.redhat.com/documentation/en-us/net_core)。

## 结论

在本文中，您了解了支持的 Red Hat 平台的概况。NET 核心，以及如何安装。每个上面都有网芯。

*Last updated: June 29, 2020*