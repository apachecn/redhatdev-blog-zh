# 使用 REST API 示例检查 Kubernetes 内部

> 原文：<https://developers.redhat.com/blog/2020/01/22/operator-pattern-rest-api-for-kubernetes-and-red-hat-openshift>

在本文中，我们将研究在任何已知框架中编写 REST API 与使用 Kubernetes 的客户端库编写操作符时的模式。本文的目的是通过比较 REST API 和 Operator 编写模式来解释 Kubernetes 的内部机制。我们不会讨论如何编写 REST API。

## 先决条件

必须安装以下设备:

*   Go 版本 13.4
*   [操作员-sdk](https://github.com/operator-framework/operator-sdk)
*   [代码就绪容器](https://github.com/code-ready/crc) (OpenShift 版本 4.2)
*   [oc](https://docs.openshift.com/container-platform/4.2/welcome/index.html) 和 [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
*   [代码回购](https://github.com/akoserwal/customer-api)

作为一名开发人员，如果您已经将 REST API 与 Quarkus/Spring (Java)、Express (Nodejs)、Ruby on Rails、Flask (Python)、Golang (mux)等框架一起使用，那么理解和编写操作符将会更加容易。我们将利用这种对其他语言或框架的经验来建立我们的理解。

通过编写一个操作符，您可以使用 Kubernetes 客户端库作为您的框架来实现一个 REST API(即，使用您的定制逻辑来扩展 Kubernetes APIs)。

如果使用 CRC/Minishift/Minikube 等在本地运行单集群 Kubernetes 实例。，在终端中键入以下命令:

```
$ kubectl proxy
```

Kubernetes API 端点如图 1 所示:

[![Kubernetes API endpoints.](img/7fe36d0e0e14d54e9a988f9fc1cbcf68.png "operatorpattern1")](/sites/default/files/blog/2019/12/operatorpattern1.png)Figure 1: Kubernetes API endpoints.

如您所见，Kubernetes 域空间中的每个实体都是一个公开的 API 端点。每当您触发一个部署或扩展一个 pod 时，您就在幕后使用这些 API。

我们可以从在熟悉的框架中编写 REST API 开始。我将使用 [Spring Boot](https://spring.io/projects/spring-boot) 来构建一个简单的客户 API。稍后，我们将编写一个`customer-api-operator`，用我们定义的客户 API 扩展 Kubernetes 的`api-server`。如果你不熟悉 Spring Boot，你可以继续。主要思想是理解使用任何语言编写 API 和使用 Kubernetes 客户端库遵循几乎相同的模式。

## REST API 示例

让我们通过定义图 2 所示的`Customer`模型类来开始这个比较的 REST API 部分:

[![Customer model class definition.](img/623f574835b22fa94924f626992cbd36.png "operatorpattern2")](/sites/default/files/blog/2019/12/operatorpattern2.png)Figure 2: Customer model class definition.

接下来，我们需要控制器，它公开了一个`Customer` REST 端点并处理请求/响应流程。让我们定义三个端点(`current`、`desired`和`reconcile`)和一个处理业务逻辑的`CustomerReconciler`服务，如图 3 所示:

[![CustomerReconciler service definition.](img/008011082c7136148e8fe382d5ab1633.png "operatorpattern3")](/sites/default/files/blog/2019/12/operatorpattern3.png)

通过输入以下内容启动您的 Spring Boot 应用程序:

```
$ mvn spring-boot:run
```

此命令在默认端口上启动应用程序。

让我们使用本演示的价值来了解客户的当前状态。通常，我们从其他 API 或数据库获取这些信息，但是对于这个例子，图 4 显示了我们的 Kubernetes pods 的当前状态:

[![](img/fb7a21f1fafd83d5bf2bbb8d2ab70895.png "operatorpattern4")](/sites/default/files/blog/2019/12/operatorpattern4.png)Figure 4: The artificially-created current pod states.

现在，让我们尝试使用图 5 中的 POST 方法和 JSON 数据用期望的状态更新我们的`Customer`的状态:

[![The JSON data.](img/08b2ad0f222e38bacd32c6e1dd49772c.png "operatorpattern5")](/sites/default/files/blog/2019/12/operatorpattern5.png)Figure 5: The JSON data.

在 Kubernetes 中，将状态从*当前*更新到*期望*是由`reconcile`端点处理的，这显示了当前状态和期望状态之间的差异。我们在图 6 所示的协调器循环中定义了处理这种状态的业务逻辑:

[![The reconciler state data.](img/873afdb50bbd3ac2b08f0ee819144ea3.png "operatorpattern6")](/sites/default/files/blog/2019/12/operatorpattern6.png)Figure

现在我们有了一个客户 API，它可以获取客户的当前状态，更新状态，并对其进行协调以处理更改。

## 自定义运算符示例

现在让我们看看如何通过使用[操作符 SDK](https://github.com/operator-framework/operator-sdk) 扩展 Kubernetes API 服务器来实现类似的 API。首先，生成操作符:

```
$ operator-sdk new *customer-api-operator*
```

该操作为我们的客户 API 操作符生成基本的样板代码，如图 7 所示:

[![Basic boilerplate code for our new API Operator.](img/67d0a1b0b9d15879b4501e3792b86772.png "operatorpattern7")](/sites/default/files/blog/2019/12/operatorpattern7.png)

首先添加模型数据类型(即`Customer`)，其中*种类*是类似 pod **、**节点、deployment **、**等的实体。接下来，为 Kubernetes 平台添加一个定制 API，这需要用 API 版本生成一个新的[定制资源定义](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CRD):

```
$ operator-sdk add api --api-version=akoserwal.io/v1alpha1 --kind=Customer
```

这个命令为我们的客户 API 生成一个基本的框架。SDK 已经完成了定义基本模型的一半工作。你可以查看一下文件 [`customer_types.go`](https://github.com/akoserwal/customer-api/blob/master/customer-api-operator/pkg/apis/customer/v1alpha1/customer_types.go) (与你的模型类相同，[【customer.java】](https://github.com/akoserwal/customer-api/blob/master/CustomerAPI/src/main/java/io/akoserwal/CustomerAPI/model/Customer.java)，在第一个例子中定义)*。*

在图 8 中，您可以看到`Spec`和`Status`已经使用生成的`Customer`结构进行了预定义，并且我们添加了来自客户模型类的新实体(即`FirstName`、`LastName`和`Email`):

[![The pre-defined Customer struct.](img/7909bcd6ce8a6db1decbfa34b76dd5ce.png "operatorpattern8")](/sites/default/files/blog/2019/12/operatorpattern8.png)Figure 8: The pre-defined Customer struct.

修改`customer_types.go`后，运行以下命令进行深度复制，并为客户 API 生成 OpenAPI 规范:

```
$ operator-sdk generate k8s
$ operator-sdk generate openapi
```

现在，生成控制器:

```
$ operator-sdk add controller --api-version=akoserwal.io/v1alpha1 --kind=Customer

```

生成的样板控制器代码将我们的客户 API 模式添加到 Kubernetes 模式中。接下来，我们添加两个观察器来观察任何变化或事件，这将触发协调器循环来处理变化，如图 9 所示:

[![The controller boilerplate code.](img/9c896f3aac82d90ff9cdc94e2fb433b6.png "operatorpattern9")](/sites/default/files/blog/2019/12/operatorpattern9.png)Figure 9: The controller boilerplate code.

我们可以将这个结果与 Spring Boot 示例的`CustomerController`和协调器端点联系起来。虽然，这个版本比我们的 Spring Boot 控制器做得更多。

您可以在图 10 中看到简化的流程:

[![The simplified API flow.](img/aefdd703c3b1122d36284b94a94ea643.png "operatorpattern10")](/sites/default/files/blog/2019/12/operatorpattern10.png)Figure 10: The simplified flow.

所以，重申一下，创建自定义操作符——而不是在您选择的框架中使用 REST API 流程如下:

1.  为我们的操作符定义一个名称空间(即`Customer`)。
2.  创建客户资源定义(CRD)，因此定义“种类”(即，在 OpenShift/Kubernetes 中将客户作为资源)。
3.  让控制器分析 CRD 并将该信息添加到 Kubernetes API 模式中。
4.  为 CRD 公开一个新的 API 端点。
5.  为要观察的名称空间添加观察器。
6.  运行一个流程/循环(即协调器循环)以根据所需的更改采取行动。
7.  将状态存储在 EtcD 中。

既然已经定义了自定义操作符，让我们继续。部署操作员:

```
$ crc start
```

登录 CRC 实例:

```
$ oc login -u kubeadmin -p <secret>
```

创建自定义 CRD:

```
$ oc create -f deploy/crds/customer.github.io_customers_crd.yaml
```

为客户添加角色、`role_binding`和服务帐户。这些设置定义了访问我们操作员的权限和规则:

```
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
$ kubectl create -f deploy/service_account.yaml

```

在本地运行我们的操作员:

```
$ export NAMESPACE=customer
$ operator-sdk up local
```

现在我们的控制器处于运行状态。它会观察`NAMESPACE`是否有任何变化。

通过创建自定义资源对象来创建 CRD 的实例:

```
$ oc create -f deploy/crds/customer.github.io_v1alpha1_customer_cr.yaml
```

创建后，CR 如下所示:

```
apiVersion: customer.github.io/v1alpha1
kind: Customer
metadata:
  name: customer-api
spec:
  *# Add fields here* size: 3
firstName: Abhishek
lastName: Koserwal
email: ak..@redhat.com
```

当我们创建一个 CR 对象时，会触发一个事件，将控制权交给协调器循环。协调为自定义对象读取该群集的状态，并根据它读取的状态进行更改。

图 11 显示了 OpenShift UI 中的客户定制资源:

[![The new Custom Resource in the OpenShift API.](img/4bf703c02dbd8242f6d79d2af147c966.png "operatorpattern11")](/sites/default/files/blog/2019/12/operatorpattern11.png)Figure 11: The new Custom Resource in the OpenShift API.

图 12 显示了原始 API 格式的客户定制资源:

[![The new Custom Resource as a raw API.](img/56bac185942c7955fae20338552c7c5d.png "operatorpattern12")](/sites/default/files/blog/2019/12/operatorpattern12.png)Figure 12: The new Custom Resource as a raw API.

基于我们的协调器循环中的逻辑，这个 CR 对象为实例`customer-api`创建了一个 pod:

```
// Define a new Pod object
pod := newPodForCR(instance)
```

图 13 显示了这样一个实例:

[![Additional update logic.](img/9944f5758abd7ae18abbb034caaa4b1c.png "operatorpattern14")](/sites/default/files/blog/2019/12/operatorpattern14.png)Figure 14:

现在，我们可以添加更新逻辑来定义现有定制资源的期望状态。让我们更新协调器，这样它将更新我们的 CR ( `firstName`、`lastName`和`Email`)，如图 14 所示:

[customer _ controller . go](https://github.com/akoserwal/customer-api/blob/master/customer-api-operator/pkg/controller/customer/customer_controller.go)[func(r *对账客户)对账]

[![Additional update logic.](img/9944f5758abd7ae18abbb034caaa4b1c.png "operatorpattern14")](/sites/default/files/blog/2019/12/operatorpattern14.png)Figure 14:

重新运行运算符:

```
$ operator-sdk up local
```

最后，在图 15 中，我们可以看到协调器已经更新了定制资源的期望状态:

[![The updated desired state.](img/e0e31058bfea6665701b7f9edcbbccc1.png "operatorpattern15")](/sites/default/files/blog/2019/12/operatorpattern15.png)Figure 15:

## **结论**

操作模式允许您使用 Kubernetes 平台功能来运行定制的业务逻辑。这些模式非常强大，超出了我在这里讨论的范围。在本文中，我主要关注 Kubernetes 的内部结构和操作模式，以及通过 REST API 的相关方法。编写这些类型的扩展并不是运算符模式的最佳用例。对于“构建 REST API 应用程序/服务”这样的用例，您可以在一个容器中运行一个 Spring Boot 或者任何其他语言的应用程序实际使用案例请访问 [operatorhub.io](https://operatorhub.io) 。

感谢您的阅读，希望这篇文章对您有所帮助。你可以[在这里](https://github.com/akoserwal/customer-api)找到代码。快乐编码。

### 参考

*   [操作员-sdk](https://github.com/operator-framework/operator-sdk)
*   [牛逼——运营商](https://github.com/operator-framework/awesome-operators)
*   [操作员框架](https://github.com/operator-framework)
*   [入门](https://github.com/operator-framework/getting-started)
*   [操作员](https://coreos.com/operators)
*   [OperatorHub](http://operatorhub.io)

*Last updated: November 8, 2022*