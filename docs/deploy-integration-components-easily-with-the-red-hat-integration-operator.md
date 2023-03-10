# 使用 Red Hat Integration Operator 轻松部署集成组件

> 原文：<https://developers.redhat.com/blog/2021/04/01/deploy-integration-components-easily-with-the-red-hat-integration-operator>

任何开发人员都知道，当我们谈论集成时，我们可以指许多不同的概念和架构组件。集成可以从 API 网关开始，扩展到事件、数据传输、数据转换等等。很容易忽略哪些技术可以帮助您解决各种业务问题。[红帽集成](https://developers.redhat.com/integration/)的 Q1 [发布](https://www.redhat.com/en/blog/red-hat-delivers-new-change-data-capture-capabilities-and-enhances-user-experience-latest-red-hat-integration-release)引入了一个针对这一挑战的新特性:红帽集成操作符。

Red Hat Integration 操作符帮助开发人员轻松探索和发现 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 中可用的组件。一个单一的运营商现在是获得最新的红帽集成组件的切入点。操作员使用一个 [Kubernetes](/topics/kubernetes/) 定制资源声明性地管理我们想要启用的组件、我们想要部署的名称空间以及它们在 Red Hat OpenShift 集群中的作用域。

现在，在正式上市时，Red Hat Integration Operator 允许您选择并安装以下任何一种操作器:

*   3 刻度
*   3 规模预测
*   AMQ 经纪人
*   AMQ 互联
*   AMQ 溪流
*   API 设计器
*   骆驼 K
*   保险丝控制台
*   在线融合
*   服务注册表

在本文中，我们将回顾 Red Hat Integration 操作符管理的不同组件，并了解如何在带有 OperatorHub 的 OpenShift 中安装操作符。最后，我们将使用一个定制的安装文件安装 3scale APIcast 和 AMQ Broker Operators。

## 使用 OperatorHub 安装 Red Hat 集成操作符

要开始使用带有 [OperatorHub](https://docs.openshift.com/container-platform/4.7/operators/understanding/olm-understanding-operatorhub.html) 的 OpenShift 上的 Red Hat Integration 操作符，请遵循以下步骤:

1.  在 OpenShift 容器平台 web 控制台中，以管理员身份登录并导航到 **OperatorHub** 。
2.  键入`Red Hat Integration`在可用运算符中进行筛选。
3.  选择**红帽积分**操作员。
4.  查看操作员详细信息，点击**安装**。
5.  在**安装操作员**页面，接受所有默认选择并点击**安装**。
6.  当安装状态显示操作器可以使用时，点击**查看操作器**。

[![Locating Red Hat Integration in the OpenShift OperatorHub via a filter search](img/34674be2ec42adf278f1cc069884998a.png "rhi-operator-hub")](/sites/default/files/blog/2021/03/rhi-operator-hub.png)

Figure 1: Searching for Red Hat Integration Operator in the OpenShift OperatorHub

现在您已经准备好为 Red Hat 集成组件安装操作器了。关于从 OperatorHub 安装操作器的更多信息，参见 [OpenShift 文档](https://docs.openshift.com/container-platform/4.7/operators/admin/olm-adding-operators-to-cluster.html#olm-installing-operators-from-operatorhub_olm-adding-operators-to-a-cluster)。

## 为 Red Hat 集成组件安装运算符

在操作员页面上，您会看到操作员提供了一个新的[自定义资源定义](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) (CRD)，名为**安装**。您可以点击**创建实例**生成一个新的[自定义资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CR)来触发操作员安装。**创建安装**页面显示了可用于安装的整套 Red Hat Integration 操作符。默认情况下，安装时会启用所有操作员。

如果您愿意，在执行安装之前，您可以从表单或 YAML 视图中配置安装规格。这样做可以让您包含或排除运算符、更改命名空间以及在运算符支持的模式之间切换。

以下是 3 倍规模的 APIcast 和 AMQ 流的配置示例:

```
apiVersion: integration.redhat.com/v1

kind: Installation

metadata:

  name: rhi-installation

spec:

  3scale-apicast-installation:

    enabled: true 

    mode: namespace

    namespace: rhi-3scale-apicast

  amq-streams-installation:

    enabled: true

    mode: namespace 

    namespace: rhi-streams

```

当您导航到 **Installed Operators** 视图时，您应该看到已安装操作符及其托管名称空间的列表(参见图 2)。

[![A list of installed Operators in the OpenShift Container Platform web console](img/fb2682d25d4a33512fa848ebd1dca2bf.png "installed_operators")](/sites/default/files/blog/2021/03/isntalled_operators.png)

Figure 2: Viewing installed Operators in the OpenShift Container Platform web console

Red Hat Integration Operator 还会升级已安装的 Red Hat Integration 组件的操作程序。它默认为自动批准策略，因此组件被升级以使用最新的可用版本。

查看完整的[文档](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q4/html-single/installing_the_red_hat_integration_operator_on_openshift/index)以获得关于安装 Red Hat Integration 操作器的更多信息。

## 结论

在前面的小节中，我们探讨了 Red Hat Integration 操作员可以使用 OpenShift 上的安装定制资源安装的不同组件。我们还回顾了一种使用 OperatorHub 安装操作符的简单方法，并使用一个定制文件在它们的名称空间范围内只安装两个组件。如您所见，现在在 OpenShift 上开始使用 Red Hat 集成比以往任何时候都容易。

Red Hat Integration 提供了一个敏捷的、分布式的、以 API 为中心的解决方案，组织可以使用它在数字世界中的应用程序和系统之间连接和共享数据。首先导航到 OpenShift OperatorHub 并安装 Red Hat Integration Operator。

Red Hat Integration Operator 在 [Red Hat Marketplace](https://marketplace.redhat.com/en-us/products/red-hat-integration) 也有售。

如果你想了解更多关于 Red Hat Integration Operator 的信息，请观看 Christina Lin 的视频。

*Last updated: March 31, 2021*