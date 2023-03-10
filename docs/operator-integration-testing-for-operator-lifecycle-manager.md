# 运营商生命周期管理器的运营商集成测试

> 原文：<https://developers.redhat.com/blog/2021/01/18/operator-integration-testing-for-operator-lifecycle-manager>

[Operators](https://developers.redhat.com/topics/kubernetes/operators) 是在 [Red Hat OpenShift](https://developers.redhat.com/openshift) 上打包、部署、管理应用分发的方式之一。开发者创建一个操作符后，下一步就是让操作符在 [OperatorHub.io](https://operatorhub.io/) 上发布。这样做允许用户在他们的 OpenShift 集群中安装和部署操作符。操作员被安装、更新，管理生命周期由[操作员生命周期管理器(OLM)](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-olm.html) 处理。

在本文中，我们将探讨测试运营商的 OLM 集成所需的步骤。为了进行演示，我们使用一个简单的操作符将测试消息打印到 shell 中。该操作符以最近推出的[包格式](https://docs.openshift.com/container-platform/4.5/operators/olm-packaging-format.html#olm-bundle-format_olm-packaging-format)打包。

对于我们的本地开发环境，我们需要访问以下工具包:

*   [Red Hat CodeReady 容器(CRC)](https://developers.redhat.com/products/codeready-containers)
*   本地机器上运行的一个 Docker 守护进程
*   Operator SDK 工具包，1.0.0 版或更高版本(可选)
*   [运营商套餐经理(OPM)](https://github.com/operator-framework/operator-registry/releases/tag/v1.13.8)
*   OpenShift 容器平台，集群版本 4.5 或更高版本

我们需要 CRC，因为它提供了方便的单节点最小 OpenShift 集群，主要用于帮助开发人员进行测试。OPM 提供了我们的本地桌面环境。

**注意**:在下载 OPM 之后，为了方便使用，我们通过设置二进制文件的权限来重命名`opm`二进制文件，至少使其可读和可执行。

一旦你设置好一切，在 [Red Hat Quay.io](https://quay.io/) 创建一个免费账户。我们使用 Quay 来构建、分析、分发和托管集装箱图像。在这种情况下，注意`quay_username`并设置以下环境变量以便于使用:

```
> export quay_username=<your_quay_user_name>
> echo $quay_username
```

现在我们已经准备好开始我们的 OLM 集成测试了。

## 步骤 1:下载操作员包

首先，让我们克隆下面的 Git 存储库。这个存储库包含我们的示例 Operator bundle 包，我们将它部署到集群中的本地 OperatorHub 实例。

```
>  git clone https://github.com/taneem-ibrahim/olm-testing-bundle-format-index.git
```

这将创建以下目录结构:

```
foo-operator % tree
├── bundle
│   ├── manifests
│   │   ├── example.com_foobars.yaml
│   │   ├── foobar-operator-metrics-reader_rbac.authorization.k8s.io_v1beta1_clusterrole.yaml
│   │   └── foobar-operator.clusterserviceversion.yaml
│   ├── metadata
│   │   └── annotations.yaml
│   └── tests
│       └── scorecard
│           └── config.yaml
├── bundle.Dockerfile
├── catalogsource.yaml
└── subscription.yaml
```

## **第二步:构建并推送操作符包映像**

我们的下一步是构建操作符包映像，并将其发布到容器注册中心。在我们的例子中，我们使用 Quay 来托管我们的容器图像。从操作员的根目录`foobar-operator`，让我们运行以下命令。用`<quay_username>`标签替换你自己的`quay_username`。

> **注意**:使用 Podman 时，只需将 Docker 一词替换为 Podman 即可运行以下命令。例如:

```
> docker login quay.io -u $quay_username
> docker build -f bundle.Dockerfile -t quay.io/$quay_username/foobar-operator:v0.0.1 .
> docker push quay.io/$quay_username/foobar-operator:v0.0.1
```

默认情况下，当我们在 Quay 中创建新的存储库时，存储库的可见性被设置为 private。为简单起见，我们登录 Quay 门户网站，看到[存储库对公众开放](https://docs.quay.io/guides/repo-view.html)。这可以通过存储库的设置选项卡或转到`https://quay.io/repository/`进行设置。然后相应地替换`quay_username`或存储库名称。

当我们希望保持库可见性私有时，我们可以添加一个[图像提取秘密](https://docs.openshift.com/container-platform/4.5/openshift_images/managing_images/using-image-pull-secrets.html#images-update-global-pull-secret_using-image-pull-secrets)。操作员面板在`openshift-marketplace`名称空间中使用默认的[服务帐户](https://docs.openshift.com/container-platform/4.5/authentication/using-service-accounts-in-applications.html)。

这个操作符是一个简单的图像，它循环并打印以下消息`v0.0.1`到控制台:

```
spec:
....
              containers:
              - command: [ "/bin/sh", "-c", "while true ; do echo v0.0.1; sleep 10; done;" ]
                image: docker.io/busybox
```

## **步骤 3:验证运营商捆绑包(可选)**

这是一个可选步骤。让我们验证一下我们刚刚构建和推送的运营商捆绑包。以下命令需要以消息结尾:`all validation tests have completed successfully`

```
> operator-sdk bundle validate quay.io/$quay_username/foobar-operator:v0.0.1
```

## **步骤 4:构建并推送一个索引映像**

推送操作员捆绑包映像后，下一步是创建和发布索引映像，使用户可以使用 OLM 将操作员安装到集群中。[索引映像](https://docs.openshift.com/container-platform/4.5/operators/olm-managing-custom-catalogs.html#olm-creating-index-image_olm-managing-custom-catalogs)是一个指向操作员清单内容的指针数据库，它使 OLM 能够查询操作员映像版本并获得集群上安装的所需操作员版本。

我们使用 OPM 工具为我们的`foobar-operator`包创建索引图像。构建图像后，我们将图像推送到 Quay。当我们使用 Podman 时，我们不需要添加`--build-tool docker`，因为`opm`默认为构建工具的 Podman:

```
> opm index add --bundles quay.io/$quay_username/foobar-operator:v0.0.1 --tag quay.io/$quay_username/foobar-operator-index:latest --build-tool docker
> docker push quay.io/$quay_username/foobar-operator-index:latest
```

和以前一样，为了简单起见，我们需要在 Quay 上将`foobar-operator`索引的存储库可见性设置为 public。

## **步骤 5:创建一个定制的 CatalogSource 对象**

[CatalogSource](https://docs.openshift.com/container-platform/4.5/operators/olm-managing-custom-catalogs.html#olm-creating-catalog-from-index_olm-managing-custom-catalogs) 表示操作符元数据，OLM 可以通过查询来发现和安装操作符及其依赖项。我们创建以下 CatalogSource 资源。不要忘记根据`$quay_username`用适当的`quay_username`替换码头上的索引图像位置。将文件另存为`catalogsource.yaml`:

```
spec:
  sourceType: grpc
  image: quay.io/<substitute_quay_username>/foobar-operator-index:latest
  displayName: Custom Catalog
  updateStrategy:
    registryPoll: 
      interval: 5m
```

让我们创建目录源对象:

```
> oc create -f catalogsource.yaml
```

我们使用上面的`openshift-marketplace`名称空间，因为它是一个全局名称空间。这意味着在任何命名空间中创建的订阅都能够在集群中解析。但是，我们可以在这里选择任何自定义名称空间，只要它与下一步中创建的相关订阅对象名称空间相匹配。

此外，CatalogSource 对象的名称决定了操作员在 OperatorHub 控制台上显示在哪个目录注册中心下。在上面的例子中，我们使用了显示名称设置为 CustomCatalog 的`custom`。我们还将目录配置为每五分钟为操作员自动轮询一次最新版本的索引图像。

我们可以通过查询 pod 日志来验证 pod 的部署:

```
> oc get pods -n openshift-marketplace | grep “custom”
> oc logs custom-<pod_id> -n openshift-marketplace
> … level=info msg="serving registry" database=/database/index.db port=50051
```

## **第六步:创建订阅对象**

[订阅](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-olm.html#olm-subscription_olm-understanding-olm)是一个定制资源，描述了运营商订阅的频道，以及运营商是否需要手动或自动更新。我们在 alpha 通道中安装操作符，因为我们的包清单通道在 annotations.yaml 文件中被设置为`alpha`。另一个可用的通道是`stable`。

我们可以通过运行以下命令来验证通道:

```
> oc get packagemanifests foobar-operator -o jsonpath='{.status.defaultChannel}'
> alpha
```

GitHub 存储库提供了一个 subscription.yaml 文件。从`foobar-operator`根目录，我们可以运行以下命令来创建订阅对象:

```
> oc create -f subscription.yaml
```

## **第七步:从 OperatorHub 安装操作员**

让我们登录到 OpenShift 管理控制台，导航到 OperatorHub，并搜索我们的操作员，如图 1 所示。

[![](img/9933b041f27873adf5f8e6d746f7961d.png "step-8")](/sites/default/files/blog/2020/09/step-8.png)

Figure 1\. Use the OperatorHub to search and locate our Operator.

现在我们可以安装这个操作符，如图 2 所示。

[![](img/42248ae25e2a2fd56f6546494c09d520.png "step8-1")](/sites/default/files/blog/2020/09/step8-1.png)

Figure 2\. Click Install and select openshift-marketplace namespace.

我们还可以查询 Operator pod 日志，看看它是否像我们在部署容器规格中为 Operator 映像所做的那样打印了`v0.0.1`:

```
> oc logs -f foobar-operator-controller-manager-<pod_id> -n openshift-marketplace
> v0.0.1
```

就是这样。我们成功验证了运营商与 OLM 的集成。

## **升级操作员版本**

现在我们做一个简单的操作符升级测试，只需将 CatalogService 版本(CSV)文件中的操作符版本标签从`0.0.1`更新为`0.0.2`。让我们从`foobar-operator`的根目录运行以下命令:

```
> sed 's/0.0.1/0.0.2/g' ./bundle/manifests/foobar-operator.clusterserviceversion.yaml > ./bundle/manifests/foobar-operator.clusterserviceversion.yaml
```

接下来，我们重复上面的步骤 1 和 2 来构建并可选地验证新的操作符包映像，并将 Docker 推送命令中的映像版本标记分别替换为`0.0.2`而不是`0.0.1`。

我们现在准备将新的操作员版本添加到注册表中。我们可以通过使用`opm add`命令来实现。注意我们是如何通过插入`from-index`参数来累积添加升级的操作符版本的。如果使用 Podman，那么我们不必传递`--build-tool docker`选项，因为`opm`默认为 Podman 构建工具:

```
> opm index add --bundles quay.io/$quay_username/foobar-operator:v0.0.2 --from-index quay.io/$quay_username/foobar-operator-index:latest --tag quay.io/$quay_username/foobar-operator-index:latest --build-tool docker
```

在 Quay 存储库中，我们可以通过运行 Docker images(或 Podman images)命令来验证操作符图像和索引图像的最新版本，如图 3 所示。

[![](img/1643de535f33334d4fd8779557a5798d.png "upgrade-1")](/sites/default/files/blog/2020/09/upgrade-1.png)

Figure 3\. Validate the latest Operator images and index images using the Docker (or Podman) images command.

我们将这个新版的索引图像推送到 Quay:

```
> docker push quay.io/$quay_username/foobar-operator-index:latest
```

CatalogSource 每五分钟自动轮询一次最新版本。五分钟后，我们可以转到 OperatorHub 控制台，验证 Operator 版本 0.0.2 是否可用，如图 4 所示。

[![](img/520b0c4cf6d9afe87142c1f5a10602fd.png "upgrade-2")](/sites/default/files/blog/2020/09/upgrade-2.png)

Figure 4\. Verify the OperatorHub console and validate that the available Operator version is 0.0.2.

让我们在`openshift-marketplace`名称空间中安装新的操作员版本，并验证操作员面板日志，以查看`v0.0.2`是否被回应:

```
> oc logs -f foobar-operator-controller-manager-<pod_id> -n openshift-marketplace

v0.0.2
```

既然您已经知道了如何为操作员测试 OLM 集成，那么就用您自己的项目来试试吧！

*Last updated: January 28, 2021*