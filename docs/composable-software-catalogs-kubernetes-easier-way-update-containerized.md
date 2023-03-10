# Kubernetes 上的可组合软件目录:更新容器化应用程序的更简单方法

> 原文：<https://developers.redhat.com/articles/2021/08/20/composable-software-catalogs-kubernetes-easier-way-update-containerized>

最近，我一直在尝试如何在 Kubernetes 上构建和使用可组合的软件目录。类似于用于 Red Hat Enterprise Linux 的[Red Hat Software Collections](/products/softwarecollections/overview)，但是适用于[容器](/topics/containers)上下文，可组合软件目录允许开发人员添加工具，而无需构建新的容器映像。

本文解释了可组合软件目录如何使用现有的容器技术来构建软件集合模型，以及它们如何潜在地为容器用户提供更多的选项，简化构建，并减小容器映像大小。

## 集装箱化世界中的软件集合

为了理解对可组合软件目录的需求，让我们回到过去。

你还记得[软件集合](https://www.softwarecollections.org/en/)吗？这个由 Red Hat 支持的项目的口号是:*你系统中任何软件的所有版本。一起。*

承诺是在同一系统上构建、安装和使用多个版本的软件，而不影响系统范围内已安装的软件包。关键点是在不影响系统范围的安装包的情况下创建这种多重环境。换句话说，它提供了额外的工具，而没有对整个操作系统的当前状态做任何改变。软件集合在当时运行良好，甚至在 DeveloperWeek 2014 上赢得了顶级创新者奖。

**注意** : Red Hat Software Collections 适用于 Red Hat Enterprise Linux 7 和早期受支持的版本。从 Red Hat Enterprise Linux 8 开始，[应用流取代了软件集合](https://access.redhat.com/support/policy/updates/rhscl)。

### 一个新的景观，但同样的需求

从 2014 年开始，事情发生了变化。容器革命突然出现，带来了诸如执行隔离、文件系统分层和卷安装等特性。这解决了相当多的问题。多亏了容器，人们可以说旧的软件集合已经过时了。但是容器编排器也出现了(在本文中，我将坚持使用 [Kubernetes](/topics/kubernetes) )。将工作负载作为容器部署在 pod 中已成为标准。最后，甚至像构建管道或 ide 这样的工作负载也转移到了云上，也在容器中运行。

但是容器本身也有局限性。在某种程度上，开发人员开始体验到容器内部的相同类型的需求，而软件集合曾经试图在操作系统级别解决这些需求。

### 为什么要重访软件集合？

一个*容器*基于一个单独的*容器映像*，它就像是多个相同容器的模板。并且容器图像可选地基于单个*容器图像* *父*。要构建容器映像，通常从基本的操作系统映像开始。然后，您一个接一个地添加层，每一层都在前一层之上，以在您的容器中提供您需要的每个附加工具或功能。因此，每个容器都基于一个图像，该图像的层覆盖在单个继承树中。容器映像是操作系统在给定时间点的当前状态的快照。

对于容器映像，软件集合的旧承诺将是有用的。在容器上下文中，提供额外工具*而不改变操作系统*的当前状态的目标变成了*，而不必构建新的容器映像*。

### 组件的组合爆炸

让我们举个例子:

*   我想在开发模式下直接从源代码中运行一个 [Quarkus](/products/quarkus) 应用程序——比如说[入门示例](https://github.com/quarkusio/quarkus-quickstarts/tree/master/getting-started)。我至少需要一个 JDK 版本和一个 Maven 版本的基础操作系统。
*   我还想用 JDK、Maven 和基本操作系统的最广泛的可用版本和风格来测试应用程序。

对于这三个组件(JDK、Maven 和操作系统)的可能变体的每种组合，我需要构建一个专用的容器映像。如果我还想用尽可能多的 Gradle 版本进行测试呢？更不用说包括原生构建用例，它需要 GraalVM。现在想象一下，如果我决定包含所有我喜欢的工具的任意版本，将会发生的组合爆炸。

### 继承与组合

当我们需要的是*组合*时，当前构建容器的方式将我们局限于*单继承模型*。有时候，能够在一个容器中组合额外的特性或工具，而不必构建新的容器映像，这将是非常棒的。事实上，我们只需要在运行时用*组合容器图像*。显然，允许完全通用似乎很难实现(如果不是不可能的话)，至少考虑到 Kubernetes 和容器的当前状态。但是，在更有限的情况下，我们只能将外部的自包含工具或只读数据注入到现有的容器中，那该怎么办呢？

## Kubernetes 上的可组合软件目录

如果你想到诸如 [Java](/topics/enterprise-java) ，Maven，Gradle，甚至 [Node.js](/topics/nodejs) ，NPM，Typescript，以及越来越多的自包含 Go 实用程序，如 Kubectl 和 Helm，以及 Knative 或 Tekton CLI 工具，那么在运行时将外部自包含工具或只读数据注入容器显然是特别相关的。严格地说，它们都不需要“安装”过程。为了开始在给定平台的大多数 [Linux](/topics/linux) 变种上使用它们，你只需要下载并解压它们。

### 结合两种容器技术

现在让我们介绍两种容器技术，它们将允许我们在运行时实现这个工具注入:

*   [集装箱存储接口(CSI)](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/) ，更具体地说 [CSI 内联短命卷](https://kubernetes.io/blog/2020/01/21/csi-ephemeral-inline-volumes/)
*   [Buildah](https://buildah.io/) 集装箱

#### 集装箱存储接口

根据 [Kubernetes 文档](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/#why-csi):

*CSI 是作为一种标准开发的，用于将任意块和文件存储系统暴露给 Kubernetes 等容器编排系统(COs)上的容器化工作负载。随着容器存储接口的采用，Kubernetes 卷层变得真正可扩展。使用 CSI，第三方存储提供商可以编写和部署插件，在 Kubernetes 中公开新的存储系统，而不必接触 Kubernetes 的核心代码。这为 Kubernetes 用户提供了更多的存储选择，并使系统更加安全可靠。*

CSI 为在 Kubernetes 中实现和集成存储解决方案打开了许多大门。最重要的是， [CSI 临时内联卷](https://kubernetes.io/blog/2020/01/21/csi-ephemeral-inline-volumes/)功能目前仍处于测试阶段，它允许您直接在 pod 规范中指定 CSI 卷及其参数，并且只能在 pod 规范中指定。这对于允许直接在 pod 内部引用注入到 pod 容器中的工具的名称来说是完美的。

#### buildhr 容器

`buildah`工具是一个众所周知的 CLI 工具，它有助于构建开放容器倡议(OCI)容器映像。在许多其他功能中，它提供了两个我们非常感兴趣的功能:

*   (从映像)创建一个容器，该容器在开始时不执行任何命令，但可以被操作、完成和修改，从而可能从中创建一个新的映像。
*   挂载这样的容器以获得对其底层文件系统的访问。

## 将 hr 容器构建为 csi 卷

将 CSI 和`buildah`结合的第一次尝试始于 Kubernetes-CSI 贡献者的原型示例，[CSI-driver-image-populator](https://github.com/kubernetes-csi/csi-driver-image-populator)。这是我在这篇文章中展示的工作的主要灵感。

提供一个非常轻量级和简单的 CSI 驱动程序，带有`image.csi.k8s.io`标识符，`csi-driver-image-populator`允许容器映像作为卷挂载。该驱动程序与 DaemonSet 一起部署，在 Kubernetes 集群的每个工作节点上运行，并等待卷装载请求。在以下示例中，容器图像引用在 pod 中被指定为`image.csi.k8s.io` CSI 卷的参数。使用`buildah`，相应的 CSI 驱动程序提取映像，从中创建一个容器，并挂载其文件系统。因此,`buildah`容器文件系统可以作为 pod 卷直接挂载。最后，pod 容器可以引用这个 pod 卷并使用它:

```
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
    - name: main
      image: main-container-image
      volumeMount:
    - name: composed-container-volume
      mountPath: /somewhere-to-add-the-composed-container-filesystem
  volumes:
    - name: composed-container-volume
      csi:
        driver: image.csi.k8s.io
        volumeAttributes:
          image: composed-container-image
```

移除 pod 后，驱动程序卸载 pod 卷，并且移除`buildah`容器。

### 使 CSI 驱动程序适应可组合软件目录

`csi-driver-image-populator`原型的某些方面不适合我们的可组合软件目录用例:

*   我们不需要 pod 中的容器来对合成的映像卷进行写访问。本文的整体思想是通过 CSI 内联卷将*只读*工具和数据注入 pod 容器。
*   坚持只读用例允许我们为一个给定的工具映像使用一个单独的`buildah`容器，并与所有引用它的 pods 共享其挂载的文件系统。`buildah`容器的数量仅取决于 CSI 驱动程序端的软件目录提供的图像数量。这为额外的性能优化打开了大门。
*   出于性能和安全原因，我们应该避免自动提取作为 CSI 内联卷挂载的容器映像。让我们通过 CSI 驱动程序之外的外部组件来提取图像。并让 CSI 驱动程序只显示已经提取的图像。因此，我们将挂载的映像限制在一个明确定义的已知映像列表中。换句话说，我们坚持使用*托管软件目录*。
*   最后，对于使用符合 OCI 标准的容器运行时(例如， [cri-o](https://cri-o.io/) )的 Kubernetes 集群，我们应该能够重用集群节点上已经由 Kubernetes 容器运行时提取的图像。这将利用 Kubernetes 发行版的映像拉取功能，并遵循其配置，而不是使用专用的、独特的机制和配置来拉取新的映像。

为了验证本文中描述的想法，刚刚列出的更改在新创建的名为 [csi-based-tool-provider](https://github.com/katalogos/csi-based-tool-provider) 的 CSI 驱动程序中实现，从`csi-driver-image-populator`原型开始引导代码。

### 提供专用的工具图像

一般来说，新的`csi-based-tool-provider`驱动程序能够挂载任何容器映像的任何文件系统子路径，作为一个 pod 只读卷。但是，为容器映像定义一个典型的结构来填充这样一个软件目录还是很有用的。对于像 Java 这样的“免安装”软件，它只是作为一个归档文件提供以供提取，填充目录的最直接的方法是使用“从头开始”的映像，软件直接在文件系统的根目录下提取。用于 [OpenJDK](/products/openjdk/overview) 11 图像的`Dockerfile`的一个例子是:

```
FROM registry.access.redhat.com/ubi8/ubi as builder
WORKDIR /build
RUN curl -L https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.9.1%2B1/OpenJDK11U-jdk_x64_linux_hotspot_11.0.9.1_1.tar.gz | tar xz

FROM scratch
WORKDIR /
COPY --from=builder /build/jdk-11.0.9.1+1 .

```

这同样适用于我们前面提到的 Quarkus 示例所需的 Maven 发行版。接下来，我们将使用 Quarkus 示例作为概念验证(POC)。

## 通过 quartus 使用可组合软件目录

现在让我们回到 Quarkus 的例子。我只想为我的容器使用一个可互换的基本操作系统，而不构建任何专用的容器映像。现在，我可以通过新的可组合软件目录中的映像上的 CSI 卷装载来管理其他工具。

完整的部署如下所示:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: csi-based-tool-provider-test
spec:
  selector:
    matchLabels:
      app: csi-based-tool-provider-test
  replicas: 1
  template:
    metadata:
      labels:
        app: csi-based-tool-provider-test
    spec:
      initContainers:
      - name: git-sync
        image: k8s.gcr.io/git-sync:v3.1.3
        volumeMounts:
        - name: source
          mountPath: /tmp/git
        env:
        - name: HOME
          value: /tmp
        - name: GIT_SYNC_REPO
          value: https://github.com/quarkusio/quarkus-quickstarts.git
        - name: GIT_SYNC_DEST
          value: quarkus-quickstarts
        - name: GIT_SYNC_ONE_TIME
          value: "true"
        - name: GIT_SYNC_BRANCH
          value: 'main'
      containers:
      - name: main
        image: registry.access.redhat.com/ubi8/ubi
        args:
          - ./mvnw
          - compile
          - quarkus:dev
          - -Dquarkus.http.host=0.0.0.0
        workingDir: /src/quarkus-quickstarts/getting-started
        ports:
          - containerPort: 8080
        env:
          - name: HOME
            value: /tmp
          - name: JAVA_HOME
            value: /usr/lib/jvm/jdk-11
          - name: M2_HOME
            value: /opt/apache-maven-3.6.3
        volumeMounts:
        - name: java
          mountPath: /usr/lib/jvm/jdk-11
        - name: maven
          mountPath: /opt/apache-maven-3.6.3
        - name: source
          mountPath: /src
      volumes:
      - name: java
        csi:
          driver: toolprovider.csi.katalogos.dev
          volumeAttributes:
            image: quay.io/dfestal/csi-tool-openjdk11u-jdk_x64_linux_hotspot_11.0.9.1_1:latest
      - name: maven
        csi:
          driver: toolprovider.csi.katalogos.dev
          volumeAttributes:
            image: quay.io/dfestal/csi-tool-maven-3.6.3:latest
      - name: source
        emptyDir: {}

```

为了从 GitHub 克隆示例源代码，我重用了我的 Kubernetes `Deployment`的一个`initContainer`中的 [git-sync](https://github.com/kubernetes/git-sync) 实用程序，但这只是出于懒惰，与当前工作无关。

### 使工具可用

实现的第一个真正有趣的部分是:

```
      ...
      volumes:
      - name: java
        csi:
          driver: toolprovider.csi.katalogos.dev
          volumeAttributes:
            image: quay.io/dfestal/csi-tool-openjdk11u-jdk_x64_linux_hotspot_11.0.9.1_1:latest
      - name: maven
        csi:
          driver: toolprovider.csi.katalogos.dev
          volumeAttributes:
            image: quay.io/dfestal/csi-tool-maven-3.6.3:latest
      ...

```

这个配置使用新的 CSI 驱动程序将我的两个[工具映像](https://github.com/katalogos/csi-based-tool-provider/tree/master/examples/catalog)公开为 CSI 只读卷。

### 安装工具

以下配置使 Java 和 Maven 安装可用于主 pod 容器，以便将它们安装在所需的位置:

```
      ...
      containers:
      - name: main
        ...
        env:
          ...
          - name: JAVA_HOME
            value: /usr/lib/jvm/jdk-11
          - name: M2_HOME
            value: /opt/apache-maven-3.6.3
        volumeMounts:
        - name: java
          mountPath: /usr/lib/jvm/jdk-11
        - name: maven
          mountPath: /opt/apache-maven-3.6.3
        ...

```

注意，pod 容器拥有 Java 和 Maven 安装的最终路径。所以 pod 容器也可以将相关的环境变量设置为正确的路径。

### 使用安装的工具

最后，将在开发模式下构建和运行应用程序源代码的容器可以基于裸操作系统映像，并且除了调用[推荐的启动命令](https://quarkus.io/guides/getting-started#running-the-application)之外别无其他事情可做:

```
      ...
      - name: main
        image: registry.access.redhat.com/ubi8/ubi
        args:
          - ./mvnw
          - compile
          - quarkus:dev
          - -Dquarkus.http.host=0.0.0.0
        workingDir: /src/quarkus-quickstarts/getting-started
        ...

```

该示例将从一个 [Red Hat Universal Base Image](/products/rhel/ubi) 开始。但是最棒的是你可以做一些改变，比如切换到一个 Ubuntu 镜像，服务器会以同样的方式启动和运行，没有任何其他的改变。而如果想切换到另一个版本的 Maven，只需在`maven` CSI 卷中更改对相应容器映像的引用即可。

如果您将这个部署扩展到 10 个 pod，将会使用相同的底层 Java 和 Maven 安装。不会在磁盘上复制任何文件，也不会在群集节点上创建其他容器。只有额外的绑定装载会在群集节点上发出。节省的空间是一样的，但是许多工作负载在这个节点上使用 Java 和 Maven 工具映像。

### 性能呢？

在第一个实现中，新的`csi-based-tool-provider`驱动程序运行 [buildah manifest](https://github.com/containers/buildah/blob/master/docs/buildah-manifest.md) 命令，在 [OCI 清单](https://github.com/opencontainers/image-spec/blob/master/manifest.md#oci-image-manifest-specification)中存储与挂载的映像相关的各种元数据，以及相关的容器和卷。尽管这种设计有助于 POC 快速工作，但它需要在整个 CSI 安装和卸载过程中使用硬锁(`NodePublishVolume`和`NodeUnpublishVolume` CSI 请求)，以避免同时修改这个全局索引并确保一致性。此外，如果有必要的话，`buildah`容器最初是在挂载时动态创建的，一旦给定的工具不再被任何 pod 容器挂载，相应的`buildah`容器就会被 CSI 驱动程序移除。

这种设计可能会导致几秒钟的装载延迟，尤其是在第一次装载映像时。代替那个设计，驱动程序现在使用一个可嵌入的、高性能的、事务性的键值数据库，名为 [BadgerDB](https://github.com/dgraph-io/badger) 。这种选择允许更好的性能和更少的读写锁引起的争用。此外，暴露给驱动程序的容器映像列表现在通过一个[挂载的配置图](https://kubernetes.io/docs/concepts/configuration/configmap/#mounted-configmaps-are-updated-automatically)进行配置。图像及其相关的`buildah`容器在后台任务中被管理、创建和清理。这两个简单的变化已经将平均卷装载延迟减少到几分之一秒，如图 1 中相关的[普罗米修斯](https://prometheus.io/)度量图所示。

[![Composable software catalogs on Kubernetes: Average volume mount delay for a tool is between 15 and 20 ms.](img/338cf53888f5c4599d6af573f6606da6.png)](/sites/default/files/lJzq7mN.png)Figure 1: The average volume mount delay for updated containers.

Figure 1: Average volume mount delay for updated containers.

在本地 [Minikube](https://minikube.sigs.k8s.io/docs/) 安装中，对于一个简单的 pod，它包含一个带有前面提到的 JDK 映像的已挂载 CSI 卷和一个非常简单的容器(除了列出已挂载卷的内容然后休眠之外什么也不做)，在 Pod 中挂载 JDK 所需的平均延迟在 15 到 20 毫秒之间波动。与整个 pod 启动持续时间(1 到 3 秒之间)相比，这是微不足道的。

### 测试示例

相关代码可以在[基于 csi 的工具提供者](https://github.com/katalogos/csi-based-tool-provider) GitHub 存储库中找到，包括如何使用预构建的容器映像测试它的说明。

## 可组合软件目录的其他用例

除了本文中使用的例子，我们可以预见这种工具注入有用的具体用例。首先，它减少了必须在单个容器映像中管理底层系统和所有各种独立于系统的工具的版本和生命周期的组合爆炸效应。因此，它可以减少存储在 Kubernetes 集群节点上的影像图层的整体大小。

### Red Hat OpenShift Web 终端

[Red Hat OpenShift Web 终端](https://www.openshift.com/blog/a-deeper-look-at-the-web-terminal-operator-1)就是一个可以从软件目录中获益的工具的例子。当打开一个 web 终端时，OpenShift 控制台启动一个 pod，其中包含一个嵌入了所有通常需要的 CLI 工具的容器。但是如果您需要额外的工具，您将不得不用您自己定制的容器映像来替换这个默认的容器映像。如果我们可以在一个基本容器中以卷的形式提供所有 CLI 工具，那么这个构建就没有必要了。可组合的软件目录也将减轻[持续集成](/topics/ci-cd) (CI)的负担，即每次有一个工具需要更新时都必须重新构建一体化容器映像。更进一步，目录应该允许在 web 终端中使用与底层 OpenShift 集群版本完全相同的 Kubernetes 相关命令行工具版本(如`oc`和`kubectl`)。

### 泰克顿管道公司

我还想象可组合软件目录如何被用来将现成的构建工具注入到 [Tekton 任务步骤](https://github.com/tektoncd/pipeline/blob/master/docs/tasks.md#defining-steps)中。同样，在这里，当您每次想要使用不同的构建工具变体或版本运行管道时，也不再需要更改和重新构建步骤容器映像。

### 云 IDEs

最后但同样重要的是，可组合的软件目录可以让各种支持云的 ide 受益，比如 [Eclipse Che。](https://www.eclipse.org/che/)目录可以让您非常轻松地:

*   在工作区中高效地切换 Java 或 Maven 安装
*   在不同的容器之间共享这些安装
*   同时有几个版本

在这方面，这种新方法也可以大大减轻 CI 负担。我们可以停止为底层操作系统和工具的每种组合构建和维护容器映像。并且可组合的软件目录将最终根据开发人员的需求在运行时解锁开发人员工具的组合。

## 接下来呢？

尽管本文中介绍的概念验证还处于早期的 alpha 阶段，但我们已经可以想象出推进它的一些后续步骤。

### 欢迎卡塔罗格斯

很多都可以建立在`csi-based-tool-provider`的基础上。但是作为第一步，我们当然应该建立一个更广泛的致力于 Kubernetes 可组合软件目录的项目。CSI 驱动程序将是它的第一个核心组件。所以我们称这个项目为 Katalogos，这个词来自古希腊语，意为目录:一个登记簿，尤其是用于注册的登记簿。

### 将项目打包为完整的解决方案

一旦更广泛的 Katalogos 项目启动，下面这些步骤就会浮现在脑海中:

*   添加一个软件目录管理器组件，将映像作为软件目录进行组织、提取和管理，并使它们可用于每个群集节点上的 CSI 驱动程序。
*   构建一个操作员来安装 CSI 驱动程序以及配置软件目录管理器。
*   定义一种方法，根据注释轻松地将所需的 CSI 卷以及相关的环境变量注入到 pod 中。
*   提供相关的工具和过程，以便轻松地构建软件目录，为软件目录经理提供信息。
*   扩展该机制以支持更复杂的情况，即软件包本身不是独立的。

### 获得反馈并建立社区

本文介绍了一个项目的一些想法，并提供了最少的概念证明，我相信这个项目能够满足当前云原生开发状态下的实际需求。这篇文章也是为了获得反馈，激发兴趣，并收集适合这个概念的其他用例。所以，请评论，尝试例子，公开问题，分叉 GitHub 库...或者简单地启动它。

*Last updated: September 19, 2022*