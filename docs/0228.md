# 使用运营商生命周期管理器捆绑包部署 Kubernetes 运营商

> 原文：<https://developers.redhat.com/blog/2021/02/08/deploying-kubernetes-operators-with-operator-lifecycle-manager-bundles>

本文展示了一个使用[Operator life cycle Manager](https://olm.operatorframework.io/)(OLM)bundle 部署架构来部署一个 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/getting-started) 或其他 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 操作员的例子。您将学习如何一起使用 OLM 和[操作符 SDK](https://sdk.operatorframework.io/)(Kubernetes[操作符框架](https://github.com/operator-framework)的两个组件)来部署操作符。

## 关于这个例子

我在 OpenShift 4.6 集群(通过[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview))和本地 Kubernetes [Kind 1.18](https://kind.sigs.k8s.io/) 集群上测试了 Operator Lifecycle Manager 示例。我在这个例子中使用了[运营商 SDK 1.0](https://sdk.operatorframework.io/) 和 OLM 0.15.0。

## 脚手架操作员

一个*包*是一个操作符打包结构，它包含一个操作符定义和清单，用于确定操作符如何部署到 Kubernetes 集群上。原始的 OLM 包清单格式已迁移为捆绑包格式。

在这个例子中，我们将使用 [Operator SDK](https://sdk.operatorframework.io/) 1.0 来生成一个操作符包，并为部署创建操作符。输入以下`operator-sdk`命令开始搭建样本操作器:

```
$ operator-sdk init --domain=example.com --repo=github.com/example-inc/doo-operator

$ operator-sdk create api --group cache --version v1 --kind Doo --resource=true --controller=true

```

## 在 Operator SDK 中使用容器图像

当`operator-sdk`生成或搭建一个操作符时，它不包含任何关于应用程序的逻辑。您将需要包括如何安装和管理您的应用程序的信息。像任何其他操作符一样，生成的代码被构建到一个容器映像中。在本例中，操作员图像被命名为`quay.io/username/doo-operator:v0.0.1`。

输入`operator-sdk init`命令创建一个项目结构。该命令还创建了一个 Makefile，您可以用它来完成各种构建和部署任务。输入以下内容，使用 Makefile 构建操作员映像:

```
$ make docker-build docker-push IMG=quay.io/username/doo-operator:v0.0.1

```

**注**:关于使用`operator-sdk`和 Golang 构建操作符的更多信息，请参见[操作符 SDK 文档](https://sdk.operatorframework.io/docs/building-operators/golang/)。

## 构建运营商捆绑包映像

除了操作符图像，*操作符包*是 OLM 规定的格式，用于保存操作符元数据。元数据包含 Kubernetes 使用操作符需要知道的一切，包括它的定制资源定义(CRD)、必需的基于角色的访问和控制(RBAC)角色和绑定、依赖树等等。

### 生成捆绑包

您可以使用`operator-sdk`生成的 Makefile 为您的操作员创建一个包:

```
$ make bundle IMG=quay.io/username/doo-operator:v0.0.1

```

这个命令生成一组磁盘上的包清单。以下是生成的包的目录结构示例:

```
bundle

├── manifests

│   ├── cache.example.com_dooes.yaml

│   ├── doo.clusterserviceversion.yaml

│   └── doo-metrics-reader_rbac.authorization.k8s.io_v1beta1_clusterrole.yaml

├── metadata

│   └── annotations.yaml

└── tests

    └── scorecard

        └── config.yaml

```

`make bundle`命令还创建了一个 Dockerfile ( `bundle.Dockerfile`，用于构建一个*包映像*。捆绑包映像是一个符合[开放容器倡议](https://github.com/opencontainers/image-spec) (OCI)的映像，它保存生成的磁盘上的捆绑包清单和元数据文件。

### 创建并推送包映像

在本例中，我们将按如下方式命名操作符包映像:

```
quay.io/username/doo-operator-bundle:v0.0.1

```

要创建和推送映像，请运行以下 Makefile 目标:

```
$ make bundle-build BUNDLE_IMG=quay.io/username/doo-operator-bundle:v0.0.1

$ make docker-push IMG=quay.io/username/doo-operator-bundle:v0.0.1

```

## 构建操作员索引映像

另一个 OLM 概念是*索引图像*。这个容器映像提供了一个应用程序编程接口(API ),它描述了关于示例操作符的信息。通过运行操作员注册表命令`opm`，索引映像包含来自您的包映像的信息:

```
$ opm index add --bundles quay.io/username/doo-operator-bundle:v0.0.1 --tag quay.io/username/doo-operator-index:v0.0.1

$ podman push quay.io/username/doo-operator-index:v0.0.1

```

索引映像包含一个带有包定义的 [SQLite](https://www.sqlite.org) 数据库。当镜像被执行时，它还运行一个 [gRPC](https://grpc.io/docs/what-is-grpc/core-concepts/) 服务。gRPC 服务让用户可以查询 SQLite 数据库中索引包含的操作符。

**注意**:可以从运营商框架的[运营商注册表](https://github.com/operator-framework/operator-registry)下载`opm`命令。

## 运行捆绑包索引映像

此时，您应该已经在 Kubernetes 集群上安装了 OLM。缺省情况下，OLM 安装在 OpenShift 集群上。您可以使用 Operator SDK 的 [olm install](https://sdk.operatorframework.io/docs/cli/operator-sdk_olm_install/) 命令在任何其他 Kubernetes 集群上手动安装 olm。

一旦有了索引映像，就通过创建一个 OLM `CatalogSource`资源来部署它。对于我们的示例操作符，我们如下创建`CatalogSource`:

```
$ cat <<EOF | kubectl create -f -

kind: CatalogSource

metadata:

  name: doo-operator

  namespace: operators

spec:

  sourceType: grpc

  image: quay.io/username/doo-operator-index:v0.0.1

EOF

```

这个`CatalogSource`是在一个现有的名称空间中创建的，由 OLM 创建，名为`operators`。在 OpenShift 集群上，这个名称空间被称为`openshift-operators`。当您创建`CatalogSource`时，它会将索引图像作为一个 pod 来执行。您可以按如下方式查看:

```
$ kubectl -n operators get pod --selector=olm.catalogSource=doo-operator

NAME                                                              READY   STATUS      RESTARTS   AGE

doo-operator-79x8z                                                1/1     Running     0          136m

```

您可以查看 pod 的日志，以确保映像为 gRPC API 提供服务:

```
$ kubectl -n operators logs pod/doo-operator-79x8z

time="2020-10-05T13:17:04Z" level=info msg="Keeping server open for infinite seconds" database=/database/index.db port=50051

time="2020-10-05T13:17:04Z" level=info msg="serving registry" database=/database/index.db port=50051

```

## 部署操作员

我们使用 OLM *订阅*资源来触发特定的运营商部署。使用下面的命令，我们创建一个`Subscription`来触发我们的示例操作符的部署:

```
$ cat <<EOF | kubectl create -f -

apiVersion: operators.coreos.com/v1alpha1

kind: Subscription

metadata:

  name: doo-subscription

  namespace: operators 

spec:

  channel: alpha

  name: doo

  source: doo-operator

  sourceNamespace: operators

EOF

```

请注意，`Subscription`是在预先存在的`operators`名称空间中创建的，因此 OLM 将在相同的名称空间中创建示例运算符。(再次注意，在 OpenShift 集群上，名称空间被称为`openshift-operators`。)

## 验证操作员

我们可以使用以下命令来验证示例操作符是否正在运行。让我们从验证`Subscription`已经创建开始:

```
$ kubectl -n operators get subscription

NAME               PACKAGE   SOURCE         CHANNEL

doo-subscription   doo       doo-operator   alpha

```

接下来，验证操作员 CSV 是否已成功部署:

```
$ kubectl -n operators get csv

NAME         DISPLAY        VERSION   REPLACES   PHASE

doo.v0.0.1   doo-operator   0.0.1                Succeeded

```

最后，验证操作员正在运行:

```
$ kubectl -n operators get pod

NAME                                      READY   STATUS    RESTARTS   AGE

doo-controller-manager-6c4bdf7db6-jcvpn   2/2     Running   0          10m

```

## 测试操作员

我们可以通过创建一个样本操作者正在观察的`CustomResource`来测试样本操作者。
在`default`名称空间中创建`CustomResource`，如下所示:

```
$ cat <<EOF | kubectl -n default create -f -

{

           "apiVersion": "cache.example.com/v1",

           "kind": "Doo",

           "metadata": {

             "name": "doo-sample"

           },

           "spec": {

             "foo": "bar"

           }

         }

 EOF

```

您的样品操作员应通过检查样品操作员日志来响应`CustomResource`的创建:

```
$ kubectl -n operators logs pod/doo-controller-manager-6c4bdf7db6-jcvpn -c manager

2020-10-05T13:29:52.175Z DEBUG controller Successfully Reconciled{"reconcilerGroup": "cache.example.com", "reconcilerKind": "Doo", "controller": "doo", "name": "doo-sample", "namespace": "default"}

```

## 创建唯一的命名空间和 OperatorGroup

到目前为止，示例使用了安装 OLM 时预先创建的名称空间。在某些情况下，您可能希望将操作员部署隔离到您创建的名称空间中。为此，您需要创建一个惟一的名称空间和`OperatorGroup`。本例中我的名称空间是`jeff-operators`:

```
$ kubectl create namespace jeff-operators

$ cat <<EOF | kubectl create -f -

apiVersion: operators.coreos.com/v1

kind: OperatorGroup

metadata:

  name: jeff-operators

  namespace: jeff-operators

status:

  lastUpdated: "2020-10-07T13:44:54Z"

  namespaces:

  - ""

EOF

```

请注意，在创建您的`CatalogSource`和订阅之前，您创建了唯一的名称空间*。您需要在包含您的`OperatorGroup`的名称空间中创建这些组件。*

**注意**:有关`OperatorGroup`的更多信息，请参见 OLM GitHub 存储库中的[Operator multi tenancy with Operator groups](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/design/operatorgroups.md)。

## Operator SDK 中的 OLM 捆绑包自动化

`operator-sdk`包括一个新的`run bundle`命令，该命令为本文描述的许多步骤使用一个临时包索引映像。对于需要使用 OLM bundle 架构测试其操作符的开发人员来说,`run bundle`命令非常有用。下面是该命令的一个示例:

```
$ operator-sdk run bundle quay.io/username/doo-operator-bundle:v0.0.1

```

## 结论

Operator Lifecycle Manager 的 bundle 架构是一种高级机制，用于在 Kubernetes 集群上描述、发布和部署操作员。本文向您介绍了如何使用 OLM 的 bundle 部署架构和 Operator SDK 来部署 Kubernetes 操作员。

## 资源

要了解有关 OLM 和运算符框架的更多信息，请参阅以下资源:

*   请访问 [OLM GitHub 知识库](https://github.com/operator-framework/operator-lifecycle-manager)和 [OLM 主页](https://olm.operatorframework.io)了解更多关于使用 Operator Lifecycle Manager 在 Kubernetes 集群中安装、管理和升级操作员及其依赖项的信息。
*   了解更多关于操作员组的[操作员多租户的信息。](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/design/operatorgroups.md)
*   参见[使用 Operator SDK](https://sdk.operatorframework.io/docs/building-operators/golang/) 构建基于 Golang 的操作符指南，快速创建基于 Go 的操作符等。
*   了解关于 SQLite 和 SQLite 的[数据库浏览器的更多信息。](https://sqlitebrowser.org/)
*   使用 grpcurl 命令行工具与 gRPC 服务器交互，开始使用 [gRPC](https://grpc.io/) 和[。](https://github.com/fullstorydev/grpcurl)

## 承认

感谢红帽工程师 Eric Stroczynski 和 Jesus Rodriguez 审阅本文。

*Last updated: February 5, 2021*