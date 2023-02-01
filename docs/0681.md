# 在 Kubernetes 部署中使用 Red Hat OpenShift 图像流

> 原文：<https://developers.redhat.com/blog/2019/09/20/using-red-hat-openshift-image-streams-with-kubernetes-deployments>

本文展示了一个应用程序更新场景，它利用了 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 图像流和标准的 [Kubernetes](http://developers.redhat.com/topics/kubernetes/) 本地资源。它还展示了图像流如何在更新其容器图像后自动重新部署应用程序容器。

最重要的是，用 OpenShift 图像流增强的 Kubernetes 资源仍然与标准的 Kubernetes 集群兼容。这个事实使得可以使用相同的资源定义来支持多个 Kubernetes 发行版，同时利用 OpenShift 独有的特性。

在本文的最后，我们提出了一些关于使用图像 id 和图像名称标签来管理回滚到应用程序以前版本的能力的考虑。

## 图像流的优势

OpenShift 在上游 Kubernetes 上提供的一个主要特性是图像流资源。使用图像流有很多好处，包括:

*   **可移植性**:图像流使你的 pod 独立于注册服务器。您可以将容器映像从 internet 上的公共注册表复制到组织内部的私有注册表中。不需要在集群节点上更改映像引用或容器引擎配置。
*   **一致性**:图像流名称和图像流标签可以遵循适合您组织的任何标准。容器图像名称和标签遵循不同的约定，根据供应商的不同有不同的含义。
*   **再现性**:图像流使得通过图像 id 引用不可变容器图像变得容易。图像名称和标签是可变的，可以在不同的时间指向不同的图像。图像流确保窗格使用已知的固定图像 ID。
*   **稳定性**:图像流确保所有副本容器使用相同的不可变图像。来自同一个副本集的 pod，每个都使用名称和标签引用图像，可能最终运行不同的容器图像。根据服务或入口发送每个请求的 pod，用户可能会得到不同的结果。
*   **可逆性**:如果映像名称和标签发生变化，新的容器映像出现问题，没有可靠的方法可以回滚到最后一个已知良好的容器映像。因为图像流保留了图像 id、代和时间戳的历史(流),所以如果需要，您可以识别并回滚到更旧的已知工作图像。
*   **自动化**:图像流生成图像更改事件，这些事件触发从引用图像流的工作负载资源中重新部署 pod。该特性允许简单的连续部署(CD)场景，而不需要 Jenkins 和其他复杂的工具。

## OpenShift 图像流基础

图像流是对容器图像的命名引用。OpenShift 扩展资源使用图像流间接引用容器图像。Kubernetes 标准资源通过注册表、名称和标签直接引用容器图像。

Kubernetes 用户可以通过正确管理图像标签来避免稳定性、再现性和可逆性问题。在演示结束时，您应该了解这些问题如何影响 Kubernetes 部署，以及一些防止这些问题的策略。不管怎样，这个任务是可以完成的，但是使用 OpenShift 图像流更容易。

图像流是 OpenShift 扩展 API 的一部分。其他 OpenShift 扩展资源，比如构建配置和部署配置，提供了对图像流的本地支持。OpenShift 工具，比如`oc`命令，提供了易于使用的命令来管理图像流资源，以及其他扩展 API 资源。

OpenShift 使用标准的 Kubernetes 扩展机制来添加它的扩展 API，比如自定义资源定义(CRD)和准入插件。这个特性允许 OpenShift 支持使用图像流和标准的 Kubernetes 工作负载 API 资源，比如部署、状态集和作业。

## 演示场景

也许你还没有准备好完全切换到 OpenShift 扩展。也许您需要继续支持其他 Kubernetes 发行版。不要担心，您可以以一种非侵入性的方式为标准 Kubernetes 资源启用图像流支持。这些增强了 OpenShift 的资源可以与标准的 Kubernetes 发行版一起使用，这些发行版会自动忽略 OpenShift 扩展，因为使用注释对这些资源进行了必要的修改。

Kubernetes 注释允许向任何 Kubernetes 资源添加非标识性元数据。任何作为注释存储的数据都不会改变 Kubernetes 资源的模式和语义。

OpenShift 使用注释这一事实意味着，当利用 OpenShift 集群上的图像流时，同一个 YAML 文件可以在任何标准的 Kubernetes 集群上不加修改地工作。当您将 YAML 文件与 OpenShift 一起使用时，它会处理注释。当您将同一个 YAML 文件与不带图像流的 Kubernetes 发行版一起使用时，就好像注释不存在一样。

### 选择测试容器图像

为了模拟应用程序更新场景，我们需要两个版本的容器映像。其中一个是旧形象，另一个是新形象。为了保持简单并与 Red Hat Enterprise Linux(RHEL)EULA 兼容，本演示使用了基本通用基本映像(UBI)。

首先为 Red Hat 的 UBI 基本图像找到当前可用的标签。您可以使用`skopeo inspect`命令，如果愿意，还可以使用`jq` JSON 解析器过滤输出:

```
$ skopeo inspect docker://registry.access.redhat.com/ubi8/ubi \ | jq -r '.RepoTags' -
[
  "8.0",
  "8.0-122",
  "8.0-126",
  "8.0-129",
  "8.0-154",
  "latest"
]
```

较小的数字表示 UBI 基础映像的较旧版本，较大的数字表示较新的版本。在下面的演示中，使用了标签`8.0-122`和`8.0-154`。

这个演示说明了 Red Hat 管理图像标签的方法。Red Hat 从不覆盖名为 *major-minor-build* 的标签。其他标签，如*主次*(如`8.0`和`latest`)都是浮动标签。浮动标签是指随着时间的推移通过 ID 指向不同容器图像的标签。

### 部署基本的 Kubernetes

为了确保在演示过程中控制容器映像更新，请将旧的容器映像复制到公共注册表中。例如，本演示中使用了 [Quay.io](https://quay.io/) 。你可以在 Quay.io 免费注册，并在那里发布你的集装箱图片，供大家消费。

使用 Quay.io web 界面在您的 Quay.io 帐户上创建一个公共存储库。将存储库命名为`ubi`。

使用`podman`登录 Quay.io，并使用`skopeo`复制旧的 UBI 映像。将以下命令中的`flozanorht`替换为您的 Quay.io 帐户，并将目标映像标记为`8`:

```
$ podman login -u *flozanorht* quay.io
Password:
Login Succeeded!
$ skopeo copy docker://registry.access.redhat.com/ubi8/ubi:8.0-122 \
docker://quay.io/*flozanorht*/ubi:8
...
Writing manifest to image destination
Storing signatures
```

下面的清单显示了引用旧映像的 Kubernetes 部署资源。这是一个有目的的裸部署，不包括运行时资源中通常包含的重要功能，如健康探测和资源限制，以便专注于添加对图像流的支持。

记得更改`image`属性以引用您的 Quay.io 帐户:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: quay.io/*flozanorht*/ubi:8
          command:
            - "/bin/bash"
            - "-c"
            - "while true; do ls /root/buildinfo/; sleep 30; done"
```

这个部署创建了永远循环的 pod，记录了`/root/buildinfo`的内容。Red Hat 使用每个新 UBI 基础映像的内部版本号更新该文件夹下的文件名。我们将使用该日志来验证正在运行的 pod 是否确实使用了适当的图像。

### 以 Kubernetes 的方式部署 OpenShift

登录到您的 OpenShift 集群，它可以是 OpenShift 3.x 的最新版本，也可以是 OpenShift 4.x 的最新版本，并且可以托管在云提供商或您的本地笔记本电脑上，使用 [Minishift](https://www.okd.io/minishift/) 或 [CodeReady Containers](https://github.com/code-ready/crc) 。对于这些选项中的每一个，本演示都应该以相同的方式进行。

我使用的是 OpenShift `oc`命令行工具，但是下面的步骤使用`kubectl`命令和标准的 Kubernetes 集群也是一样的。当我创建一个图像流时，我将只使用 OpenShift 独有的特性。

创建一个新项目，并从前面引用的 YAML 文件中创建 Kubernetes 部署资源:

```
$ oc create -f myapp.yaml
deployment.apps/myapp created
```

几分钟后，带有一个容器的运行容器就准备好了:

```
$ oc get deployment
NAME    READY UP-TO-DATE   AVAILABLE AGE
myapp   1/1   1            1         6s
$ oc get pod
NAME                    READY   STATUS    RESTARTS   AGE
myapp-75c97cd8f-m5pjk   1/1     Running   0          10s
```

检查 pod 日志，替换上述命令输出中的 pod 名称，以查看其容器映像的内部版本号。它应该与您作为旧图像选取的标签相匹配:

```
$ oc logs myapp-75c97cd8f-m5pjk | tail -n1
Dockerfile-ubi8-8.0-122
```

验证正在运行的窗格是否通过其名称和标签引用了旧的容器图像:

```
$ oc get pod myapp-75c97cd8f-m5pjk -o jsonpath='{.spec.containers[0].image}{"\n"}'
quay.io/flozanorht/ubi:8
```

### 创建图像流

到目前为止，我们可以使用任何 Kubernetes 发行版及其相关的`kubectl`命令行工具。现在我们切换到特定于 OpenShift 的特性。

创建一个指向您在 Quay.io 上的旧图像的图像流。还包括一个图像流标记资源，它指向旧图像名称的当前图像 ID 及其引用的标记。

请记住在以下命令中更改映像名称，以引用您的 Quay.io 帐户:

```
$ oc import-image ubi --confirm --all --from quay.io/*flozanorht*/ubi
imagestream.image.openshift.io/ubi imported
...
$ oc get istag
NAME    IMAGE REF                                   UPDATED
ubi:8   quay.io/flozanorht/ubi@sha256:a17a...e8e1   6 seconds ago
```

记录图像 ID，因为稍后将引用它:它是在`sha256`文本之后的随机十六进制数字的字符串。这个字符串是不可变容器映像的真实 ID。它将在容器映像更新后与您获得的 ID 进行比较。

### 向 Kubernetes 部署添加 OpenShift 注释

现在开始真正的乐趣。`oc`工具提供了方便的命令`oc set triggers`,该命令通过 workloads API 向 Kubernetes 资源添加映像更改触发器注释。该命令采用图像流的名称和标签:

```
$ oc set triggers deployment/myapp --from-image ubi:8 -c myapp
deployment.extensions/myapp triggers updated
```

以原始 YAML 格式列出部署资源，以查看具有“魔力”的注释

```
$ oc get deployment myapp -o yaml | grep -A2 annotations:
  annotations:
    deployment.kubernetes.io/revision: "2"
    image.openshift.io/triggers: '[{"from":{"kind":"ImageStreamTag","name":"ubi:8"},"fieldPath":"spec.template.spec.containers[?(@.name==\"myapp\")].image"}]'
```

这个注释使用一个 [JSONpath](https://github.com/json-path/JsonPath) 表达式来更新 Kubernetes 资源中的图像引用。如果您想将图像流与其他种类的 Kubernetes 资源一起使用，比如 cron 作业，那么您需要适当地修改 JSONpath 表达式。如果你使用`oc set triggers`命令，OpenShift 会为你提供表达式。

添加此注释会触发一个图像更改事件，以确保当前运行的窗格使用图像流中的容器 ID。过了一会儿，新的 pod 已经准备好并正在运行。检查它现在是指图像流中的同一个图像 ID:

```
$ oc get pod
NAME                     READY STATUS RESTARTS   AGE
myapp-58cc598448-qr2xn   1/1 Running 0     10s
$ oc get pod myapp-58cc598448-qr2xn -o jsonpath='{.spec.containers[0].image}{"\n"}'
quay.io/flozanorht/ubi@sha256:a17a..e8e1
```

## 映像更新时自动重新部署

将您的“新”容器映像复制到 Quay.io，覆盖引用旧映像的相同名称和标签:

```
$ skopeo copy docker://registry.access.redhat.com/ubi8/ubi:8.0-154 \
docker://quay.io/*flozanorht*/ubi:8
...
```

验证 pod 仍在使用旧的容器映像 ID:

```
$ oc get pod myapp-58cc598448-qr2xn -o jsonpath='{.spec.containers[0].image}{"\n"}'
quay.io/flozanorht/ubi@sha256:a17a..e8e1
```

更新您的图像流以指向新图像，并验证图像流标签现在是否指向新的图像 ID:

```
$ oc import-image ubi --confirm --all
imagestream.image.openshift.io/mysql imported
...
$ oc get istag
NAME    MAGE REF                                   UPDATED
ubi:8   quay.io/flozanorht/ubi@sha256:985e..286e   34 seconds ago
```

更新图像流会触发重新部署。几分钟后，将参照新的图像 ID 创建一个新的 pod:

```
$ oc get pod
NAME                     READY  STATUS   RESTARTS   AGE
myapp-6b6c9c9787-t8kmd   1/1    Running  0          24s
$ oc get pod myapp-6b6c9c9787-t8kmd -o jsonpath='{.spec.containers[0].image}{"\n"}'
quay.io/flozanorht/ubi@sha256:985e..286e
```

最后，新 pod 的日志显示了新容器映像的内部版本号:

```
$ oc logs myapp-6b6c9c9787-t8kmd | tail -n1
Dockerfile-ubi8-8.0-154
```

Kubernetes 部署上的事件将重新部署显示为其副本集的放大和缩小事件:

```
$ oc describe deployment myapp
...
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  18m    deployment-controller  Scaled up replica set myapp-75c97cd8f to 1
  Normal  ScalingReplicaSet  13m    deployment-controller  Scaled up replica set myapp-58cc598448 to 1
  Normal  ScalingReplicaSet  12m    deployment-controller  Scaled down replica set myapp-75c97cd8f to 0
  Normal  ScalingReplicaSet  2m53s  deployment-controller  Scaled up replica set myapp-6b6c9c9787 to 1
  Normal  ScalingReplicaSet  2m41s  deployment-controller  Scaled down replica set myapp-58cc598448 to 0
```

记住，您得到了两次重新部署:第一次是在添加注释时，第二次是在更新图像流时。您可能想知道，如果您已经创建了包含注释的部署资源，而不是在以后添加它，这会阻止一次重新部署吗？答案是否定的。

OpenShift 在创建部署资源后触发映像更改事件。到那时，Kubernetes 部署已经创建了一个副本集和一个 pod。唯一的变化是，第一次部署的豆荚再也看不到了。在达到运行状态之前，它们将被新的吊舱所取代。

您可以描述图像流以查看其图像 id 的历史(或流),按时间倒序排列:

```
$ oc describe is ubi | grep -A5 tagged
  tagged from quay.io/flozanorht/ubi:8

  * quay.io/flozanorht/ubi@sha256:985e..286e
      10 minutes ago
    quay.io/flozanorht/ubi@sha256:a17a..e8e1
      10 minutes ago
```

您可以强制您的图像流使用旧的图像 ID，从而安全地回滚到以前已知良好的容器图像。

## 关于图像流的更多信息

这个演示只涉及了 OpenShift 中图像流的基础知识。图像流的其他优秀特性包括:

*   **具有多个标签的图像流**:每个标签可以引用不同的注册表、图像名称和标签。
*   **图像流的预定更新**:更多内置自动化。
*   **引用其他图像流的图像流**:多级间接。
*   **共享项目中的图像流和 pull secrets**:让您的团队更容易使用您的企业注册中心。
*   **立即部署代码变更**:将 OpenShift 构建推送到图像流。

## 用于管理图像 id 的图像流的替代方案

引言中暗示的稳定性、可再现性和可逆性问题是使用浮动容器图像标签的结果。如果您的组织(或您的供应商)通过为每个新图像提供一个新的非浮动标签来管理他们的标签，就像 Red Hat 为其容器图像提供的那样，您可以依靠这些标签来避免您的应用程序的多个 pods 运行不同的容器图像。

当然，对于开发人员来说，部署`ubi8/ubi:8.0`映像比部署`ubi8/ubi:8.0-122`映像更方便。标准的 Kubernetes 没有提供自动的机制来获取浮动标签，并在不同的时间点记录与之相关的图像 ID。由于图像流，OpenShift 使得使用浮动标签变得安全。

如果创建 Kubernetes 部署来引用非浮动标记，那么在更新应用程序时需要更新部署资源。您可以使用像[詹金斯](https://jenkins.io)或[泰克顿](https://tekton.dev)这样的工具，将这些更新作为 CI/CD 管道的一部分来实现。

基于操作符的软件，如[Red Hat open shift Container Platform 4](https://developers.redhat.com/products/openshift)cluster operators，将它们的部署配置为引用映像 id，并且不依赖任何标签。新版本依靠[运营商生命周期管理器](https://docs.openshift.com/container-platform/4.1/applications/operators/olm-understanding-olm.html)来更新其部署。

请注意，单独使用图像 id 并不能免除您管理非浮动标签的责任。您的策略取决于您的注册服务器软件。OpenShift 内部注册表为一个映像名称保留了可配置数量的旧映像 id。旧图像将被[修剪](https://docs.openshift.com/container-platform/3.11/admin_guide/pruning_resources.html)，并且不可能回滚到它们。

Red Hat Quay 遵循一种不同的方法:其 id 没有被任何标签引用的所有图像都被后台任务修剪。遵循这种方法的注册服务器要求您维护非浮动标记，以防止图像删减。

### 了解更多信息

*   [使用 Kubernetes 资源的图像流](https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html#using-is-with-k8s)来自 Red Hat OpenShift 容器平台 3.11 产品文档，但同样适用于 Red Hat OpenShift 容器平台 4.x。
*   [管理图片](https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html)来自 Red Hat OpenShift 容器平台 3.11 产品文档，但大部分内容同样适用于 Red Hat OpenShift 容器平台 4.x。
*   [如何使用 OpenShift 图像流简化 Kubernetes 中的容器图像管理](https://blog.openshift.com/image-streams-faq/)
*   要使用您的笔记本电脑来尝试这个演示，可以获得 OpenShift 3.x 的 Minisift，作为[容器开发工具包](https://developers.redhat.com/products/cdk/overview)的一部分，或者获得 OpenShift 4.x 的 [CodeReady 容器](https://cloud.redhat.com/openshift/install/crc/installer-provisioned)
*   [OKD 项目](https://www.okd.io/)是红帽 OpenShift 容器平台 3.x 的上游
*   GitHub 的 openshift api 服务器操作器是众多为 Red Hat OpenShift 容器平台 4.x 提供 Kubernetes 扩展 api 的项目之一。

感谢 Adan Kaplan、Andrew Block、Ben Browning 和 Raffaele Spazzoli 对本文草稿的评论和意见。

*Last updated: July 1, 2020*