# Red Hat CodeReady Workspaces 2.2 中的性能和可用性增强

> 原文：<https://developers.redhat.com/blog/2020/07/10/performance-and-usability-enhancements-in-red-hat-codeready-workspaces-2-2>

[Red Hat CodeReady work spaces 2.2](https://developers.redhat.com/products/codeready-workspaces/overview)现已推出。对于这个版本中的改进，我们专注于性能和配置，并更新了 CodeReady Workspaces 2.2，以使用最流行的运行时和堆栈的新版本。我们还增加了只分配 IDE 插件所需 CPU 的能力，并且引入了一个新的诊断特性，允许您在调试模式下启动一个工作区。

CodeReady Workspaces 2.2 在 [OpenShift 3.11](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.0/html/installation_guide/installing-codeready-workspaces-on-openshift-3-using-the-operator_crw?extIdCarryOver=true&sc_cid=701f2000000RmAOAA0) 和 [OpenShift 4.3](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.1/html/installation_guide/installing-codeready-workspaces-on-ocp-4?extIdCarryOver=true&sc_cid=701f2000000RmAOAA0#installing-the-codeready-workspaces-operator-in-openshift-4-web-console_installing-codeready-workspaces-on-openshift-4-from-operatorhub) 及更高版本上可用，包括对 OpenShift 4.5 的技术预览支持。

**注**:基于 [Eclipse Che](https://www.eclipse.org/che/getting-started/cloud/?sc_cid=701f2000000RtqCAAS) ，CodeReady Workspaces 是一个[红帽 open shift](https://developers.redhat.com/openshift/)-原生开发者环境，支持云原生开发。

## 更快的工作空间加载

每当您启动一个工作空间时，底层集群会获取并部署组成该工作空间的所有远程映像。此活动可能会导致工作空间可用之前等待更长时间。

在 CodeReady Workspaces 2.2 中，我们添加了一个新的可选的[操作符](https://developers.redhat.com/topics/kubernetes/operators/)，它可以减少启动工作空间所需的时间。Image Puller 操作员会预拉为您的工作区指定的远程图像，从而缩短等待时间。您所需要做的就是在运行 CodeReady 工作区的同一个集群上安装 Image Puller 操作符。图 1 显示了 [OperatorHub](https://operatorhub.io/) 中的新操作员。

[![A screenshot showing the new Operator in the Operator Hub.](img/e4f8342a0ba5f22842635edfd6d37d30.png "image5")](/sites/default/files/blog/2020/06/image5.png)

Figure 1: The Image Puller Operator in the Operator Hub.

图 2 显示了安装页面，其中包含安装图像拉出器操作器的说明。

[![A screenshot of the installation page for the Image Puller Operator.](img/0861abd901e1003abaad59689b791ece.png "image2")](/sites/default/files/blog/2020/06/image2-1.png)

Figure 2: Install the Image Puller Operator.

## 支持多个 devfile 注册表

现在可以用多个 devfile 注册表配置 CodeReady 工作区。使用这个特性，组织可以为开发人员工作区提供他们管理的 devfiles 的多个来源。图 3 显示了在 CodeReady Workspaces 集群中配置的 devfile 注册表。

[![A screenshot of the devfile registry in the CodeReady Workspaces cluster.](img/a90c1c4ebf28f3baf41c45523c627fec.png "image1")](/sites/default/files/blog/2020/06/image1-1.png)

Figure 3: The devfile registry setting in the CR for CodeReady Workspaces instance.

图 4 显示了可用于在 CodeReady 工作区中创建定制工作区的 devfiles 列表。

[![A screenshot of the page to create a custom workspace in CodeReady Workspaces 2.2.](img/3d1c7237f4e78108c1db1a7a2e6a086e.png "image6")](/sites/default/files/blog/2020/06/image6-1.gif)

Figure 4: List of devfiles across all of the registries specified in the CR for the CodeReady Workspaces instance.

## CR 定义中已弃用的`selfSignedCert`设置

CodeReady Workspaces 现在会自动检测路由器证书是否是自签名的。如果发现证书是自签名的，它将被传播到 CodeReady Workspaces 服务器及其组件。因此，CodeReady 工作区实例的 CR 中的`selfSignedCert`设置现在被忽略，不需要指定。

## 核心运行时和堆栈的更新

我们还将 CodeReady 工作区提供的一组 devfiles 更新到了更高的版本。CodeReady Workspaces 2.2 更新了以下运行时映像和堆栈:

*   **Maven 3.6** :此次更新包括对内存和 CPU 消耗的改进。
*   **MongoDB 3.6** :此次更新包含了与安全性和稳定性相关的增强。
*   **代码示例**:我们更新了各种代码示例，以修复与使用相关的问题。

## 在 IDE 插件上设置 CPU 限制

CodeReady Workspaces 现在允许您使用`cpuLimit`和`cpuRequest`为 IDE 插件分配 CPU 限制。与允许开发人员分配内存限制的现有功能类似，该功能允许您为插件声明系统限制。设置 CPU 限制可以防止工作区窗格过载。图 5 显示了一个 devfile，它带有为 IDE 插件设置 CPU 限制的新选项。

[![A screenshot of the config file with the new option to set CPU limits for IDE plugins.](img/7897d5d6d7e6eea02e796a116f3652f4.png "image3")](/sites/default/files/blog/2020/06/image3.png)

Figure 5: Set CPU limits for IDE plugins.

## 以调试模式运行您的工作区

CodeReady 工作区现在允许您在调试模式下运行工作区，这意味着您可以在工作区启动时查看日志。访问日志有助于解决启动过程中遇到的问题。图 6 显示了在调试模式下启动工作区的新选项。

[![A screenshot of the startup screen with the new option to restart in debug mode.](img/a67f0bc2027b744953ffd8dcb75b870d.png "image4")](/sites/default/files/blog/2020/06/image4.gif)

Figure 6: Start your workspace in debug mode.

## 立即获取 CodeReady Workspaces 2.2！

CodeReady Workspaces 2.2 现已在 OpenShift 3.11 和 OpenShift 4.3 及更高版本中提供:

*   如果您正在使用 OpenShift 3.11，请按照 [OpenShift 3.11 安装说明](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.0/html/installation_guide/installing-codeready-workspaces-on-openshift-3-using-the-operator_crw)安装 CodeReady Workspaces 2.2。
*   如果您使用的是 OpenShift 4.3 或更高版本，可以直接从 OpenShift OperatorHub 安装 CodeReady Workspaces 2.2。遵循本程序的 [OpenShift 4.3 文档](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.1/html/installation_guide/installing-codeready-workspaces-on-ocp-4#installing-the-codeready-workspaces-operator-in-openshift-4-web-console_installing-codeready-workspaces-on-openshift-4-from-operatorhub)。

## 额外资源

对于不熟悉 CodeReady 工作区的开发人员，我们推荐以下附加资源:

*   下载 Red Hat [CodeReady Workspaces 命令行界面](https://developers.redhat.com/products/codeready-workspaces/download) (CLI)
*   访问 Red Hat [CodeReady Workspaces 产品页面](https://developers.redhat.com/products/codeready-workspaces)。
*   访问【Red Hat CodeReady 工作区入门页面。

*Last updated: July 9, 2020*