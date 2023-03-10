# VS 代码的 OpenShift 连接器 0.2.0 扩展中的 Devfiles 和 Kubernetes 集群支持

> 原文：<https://developers.redhat.com/blog/2020/11/16/devfiles-and-kubernetes-cluster-support-in-openshift-connector-0-2-0-extension-for-vs-code>

我们很高兴地宣布，Visual Studio 代码 (VS 代码)的 [OpenShift 连接器扩展的新版本现已推出。0.2.0 版本提供了在](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector) [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 集群上快速开发和部署代码的新特性。OpenShift 连接器现在支持使用 devfiles 的组件部署，利用了幕后的 [odo 2.0 命令行接口](https://developers.redhat.com/blog/2020/10/06/kubernetes-integration-and-more-in-odo-2-0/)。

在这个版本中，扩展现在支持连接到 vanilla Kubernetes 集群，并包括一个通过[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview)(CRC)在本地创建 OpenShift 4 集群的新选项。在本文中，我们将介绍这些新特性，并展示在 OpenShift Connector 0.2.0 中使用 CodeReady 容器的工作流程。

## 安装 OpenShift 连接器 0.2.0

1.  [直接从 Visual Studio 代码市场安装 OpenShift 连接器插件](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)。
2.  或者，通过点击左侧任务栏中的方形图标，选择 VS 代码中的**扩展**视图。搜索 **OpenShift 连接器**插件，点击**安装**。
3.  安装完扩展后，OpenShift 图标将被添加到左侧的活动栏中，并可供使用。

## Odo 2.0 in OpenShift 连接器 0.2.0

除了源到图像(S2I)文件，OpenShift 连接器现在还支持 [odo 2.0](https://odo.dev/blog/) 。使用`odo`，您可以创建基于 devfile 的组件，并将它们部署在您的 OpenShift 和 Kubernetes 集群上。

### 在 vs 代码中使用 desfiles

devfile 是一个 YAML 文件，用于在 [Eclipse Che](https://developers.redhat.com/videos/youtube/S3auoOqwDS8) 中定义开发人员工作区。现在 odo 2.0 支持 devfiles，它是可移植开发环境中易于配置和可复制的定义。每个 devfile 都是开发人员工作区的声明性抽象。它包括运行时环境和项目源代码，映射到您编码、构建、测试、运行和调试应用程序所需的存储库、工具、插件和命令。使用 devfiles 使您的开发人员工作区可复制。您可以在 devfile 中使用 OpenShift 应用程序定义。

#### 列出设计文件组件

OpenShift 连接器现在允许您在集成的 VS 代码终端窗口中查看当前支持的 devfile 组件列表。右键单击 OpenShift 或 Kubernetes 集群，并选择**列出目录组件**。您将在终端中看到以下输出(参见图 1)。

[![The terminal displays a list of current devfile components.](img/f38311759a1685348a694f5c6a17fd83.png "odo-catalog-components")](/sites/default/files/blog/2020/10/odo-catalog-components.png)odo catalog

Figure 1: Select 'List Catalog Components' to see a list of currently supported devfile components.

#### 列出基于运营商的服务

您还可以使用 OpenShift 连接器(和 odo 2.0)在 VS 代码终端窗口中查看基于操作符的服务列表。右键单击 OpenShift 或 Kubernetes 集群，并选择**列出目录服务**。您将在终端中看到下面的列表(参见图 2)。

[![The terminal displays a list of Operator-based services in the cluster.](img/a6d93674e7b17336c920d4fbcbed3d5d.png "odo-services")](/sites/default/files/blog/2020/10/odo-services.png)

Figure 2: Select 'List Catalog Services' to see a list of Operators in the cluster.

**注意** : OpenShift 连接器对 odo 2.0 的支持与使用 devfiles 作为 Red Hat 开发人员工具组合的通用定义格式的趋势相一致。

### 设计文件集成的工作演示

本演示视频将指导您完成在远程 OpenShift 集群上使用 Node.js devfile 部署配置的工作流程。

[https://www.youtube.com/embed/HEsYgDqD1rM?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/HEsYgDqD1rM?autoplay=0&start=0&rel=0)

## 本地运行 OpenShift 集群

OpenShift 连接器现在允许您直接从 VS 代码中的扩展创建 OpenShift 4 集群。我们已经添加了这个特性来为 OpenShift 上的开发者改善[内循环开发体验](https://developers.redhat.com/devnation/tech-talks/odo-iterative-container-based-development)。有几个基础设施选项可用，包括使用 CodeReady 容器在本地运行 OpenShift 4 集群。

要访问这个特性，在 VS Code 的 Application Explorer 视图中选择新的**Add open shift Cluster**(+icon)命令，如图 3 所示。

[![The 'Add OpenShift Cluster' option is a link in the upper-right corner of the screen.](img/02b7d44736869be9bfc47757f7547237.png "add-openshift-cluster")](/sites/default/files/blog/2020/10/add-openshift-cluster.png)

Figure 3: Select the 'Add OpenShift Cluster' option in VS Code's Application Explorer view.

然后，您将看到图 4 中显示的两个选项。

[![](img/639d378d17a26046db09032eb36b5843.png "add-cluster-crc")](/sites/default/files/blog/2020/10/Screenshot-2020-10-20-at-2.51.59-AM.png)

Figure 4: Deploy your cluster locally or in a public cloud.

### 在本地创建集群

OpenShift Connector 利用 [CodeReady 容器](https://code-ready.github.io/crc/)让您在笔记本电脑或台式机上创建一个最小的、预配置的 OpenShift 4 集群，用于开发和测试。OpenShift 连接器 0.2.0 扩展支持 CodeReady 容器 1.16.0，该容器目前运行 OpenShift 4.5.9。(我们即将发布对 OpenShift 4.6 的支持。)

选择**创建/刷新集群**选项会打开一个向导视图，您可以在其中配置集群内存、CPU 内核、名称服务器等。完成配置后，您将能够根据需要启动、停止和刷新集群。然后，您可以查看集群在本地系统中运行时的状态。所有三个平台都支持这一功能，包括 [Windows](https://code-ready.github.io/crc/#_microsoft_windows) 、 [Linux](https://code-ready.github.io/crc/#_linux) 和 [macOS](https://code-ready.github.io/crc/#_macos) 。

**注意**:code ready Containers open shift 集群是短暂的，并不打算用于生产。参见 CodeReady 容器[入门指南](https://code-ready.github.io/crc/#differences-from-production-openshift-install_gsg)了解更多详情。

### 在公共云中部署集群

如果您喜欢在公共云中部署您的集群，您可以在受支持的[公共云提供商](https://cloud.redhat.com/openshift/install#public-cloud)的帐户中安装 OpenShift 4。公共云选项包括亚马逊网络服务(AWS)、微软 Azure 和谷歌云。

## 如何使用 OpenShift 连接器 0.2.0 运行 CodeReady 容器

您可以使用 OpenShift Connector 0.2.0 快速设置和运行 CodeReady 容器。

1.  **下载 CodeReady 容器包**:下载并解压适用于您的操作系统的 CodeReady 容器 [1.16.0](http://mirror.openshift.com/pub/openshift-v4/clients/crc/1.16.0/) 包，并将二进制文件放入您的`$PATH`中。在提供路径时，一定要提供完整的可执行文件位置(例如，`~/Downloads/crc-macos-1.16.0-amd64/crc`)。图 5 显示了下载 1.16.0 包和设置二进制文件位置的选项。

[![](img/d926b714caee6fcfa067103da0d63407.png "download-binary-view")](/sites/default/files/blog/2020/11/Screenshot-2020-11-09-at-2.14.33-AM.png)

Figure 5: Download the CodeReady Containers binary and add the binary location.

2.  **为图像 pull secret** 设置文件路径:下一步是提供 pull secret 的文件位置，如图 6 所示。你可以使用[你的红帽开发者账号](https://cloud.redhat.com/openshift/install/crc/installer-provisioned)下载和上传一个拉密文件。

[![Select pull secret file location](img/801c6684342535a1e6120ceaf8f2bc26.png "crc-pull-secret-view")](/sites/default/files/blog/2020/11/Screenshot-2020-11-09-at-2.46.38-AM.png)

Figure 6: Provide the pull secret's file location.

3.  **Select the following optional CRC configurations**:
    *   CPU 核心:分配给 OpenShift 集群的 CPU 核心数量(缺省值为 4)。
    *   Memory:分配给 OpenShift 集群的内存的 MiB 值(默认值为 9，216)。
    *   Nameserver:用于 OpenShift 集群的名称服务器的 IPv4 地址。

    图 7 显示了可选配置。

[![](img/73cfa787cd2883acb266353b45709125.png "Screenshot 2020-11-09 at 3.00.04 AM")](/sites/default/files/blog/2020/11/Screenshot-2020-11-09-at-3.00.04-AM.png)

Figure 7: Enter the optional configurations for CodeReady Containers.

4.  **设置 CodeReady 容器**:点击**设置 CRC** ，该命令将在 VS 代码集成终端执行，如图 8 所示。此操作为 CodeReady Containers 虚拟机设置您的主机环境。如果`~/.crc`目录不存在，将创建该目录。

[![](img/8aaa04d89d9bed943cacd648cb41e132.png "Screenshot 2020-11-09 at 3.05.56 AM")](/sites/default/files/blog/2020/11/Screenshot-2020-11-09-at-3.05.56-AM.png)

Figure 8: Set up your host operating system for the CodeReady Containers virtual machine.

5.  **启动集群**:这个命令，如图 9 所示，启动 CodeReady Containers 虚拟机，并在您的笔记本电脑或台式电脑上创建一个最小的 OpenShift 4.5.9 集群。

[![](img/9583b8a4ac32411f88354ab4a894a9cd.png "Screenshot 2020-11-09 at 3.10.30 AM")](/sites/default/files/blog/2020/11/Screenshot-2020-11-09-at-3.10.30-AM.png)

Figure 9: Start CodeReady Containers.

一旦集群运行，将显示 OpenShift 集群状态，如图 10 所示。

[![The terminal shows that the OpenShift cluster is running.](img/637261884c0a636c0da6ed1aa6b0235e.png "crc-status")](/sites/default/files/blog/2020/10/Screenshot-2020-10-21-at-3.23.38-AM.png)

Figure 10: Confirm that the OpenShift cluster is running in CodeReady Containers.

## 查看和管理 Kubernetes 集群

OpenShift 连接器扩展依赖于微软的 [Kubernetes 扩展](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)，它是和 OpenShift 连接器一起自动安装的。这个扩展允许您查看所有的 Kubernetes 集群，并简化了 Kubernetes 资源的管理。图 11 显示了作为 OpenShift 连接器依赖项之一的 Kubernetes 扩展。

[![](img/36e90d982773db98ca538b8928047912.png "Screenshot 2020-11-09 at 6.16.03 AM")](/sites/default/files/blog/2020/11/Screenshot-2020-11-09-at-6.16.03-AM.png)

Figure 11: Dependencies for OpenShift Connector.

使用 Kubernetes 扩展的公共 API，Kubernetes 集群视图显示特定于 OpenShift 的资源，如项目、路线、部署配置、图像流、模板等等。这些资源仅对 OpenShift 集群可见。OpenShift 连接器扩展还提供了一个 **Use Project** 命令，用于在 Kubernetes 集群视图中的多个 OpenShift 项目之间进行切换。

## 支持 OpenShift 连接器 0.2.0

如果您需要支持修复错误或使用 OpenShift 连接器 0.2.0 中的新功能，请联系我们。如果你想为这个 VS 代码扩展提出一个新的特性，我们也很高兴收到你的来信。您可以通过以下方式与 OpenShift Connector 0.2.0 开发团队联系:

*   从 VS 代码命令面板或扩展标题视图中选择 **OpenShift:报告扩展问题**。
*   直接在 [VS Code OpenShift Tools GitHub 库](https://github.com/redhat-developer/vscode-openshift-tools/issues)上提交问题。
*   在 [Gitter](https://gitter.im/redhat-developer/openshift-connector) 上与开发团队和社区讨论您的问题。

## 尝试 OpenShift 连接器 0.2.0

OpenShift 连接器 0.2.0 扩展可从 [VS 代码市场](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-openshift-connector)和【open-vsix.org 注册中心安装。我们已经在为未来的版本开发新的特性。敬请关注更新！

*Last updated: October 4, 2022*