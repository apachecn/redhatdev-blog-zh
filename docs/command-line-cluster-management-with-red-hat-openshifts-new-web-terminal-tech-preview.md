# 使用 Red Hat OpenShift 的新 web 终端进行命令行集群管理(技术预览版)

> 原文：<https://developers.redhat.com/blog/2020/10/01/command-line-cluster-management-with-red-hat-openshifts-new-web-terminal-tech-preview>

Red Hat OpenShift 的 web 控制台简化了许多开发和部署工作，只需点击几下鼠标，但有时您需要命令行界面(CLI)来完成集群上的工作。无论您是在教程中通过剪切和粘贴来学习，还是在产品中排除深层错误(通常也是通过剪切和粘贴来完成)，您都可能需要在命令提示符下输入至少一两行。

从 4.5.3 版本开始，OpenShift 用户可以试用新的网络终端运营商的技术预览版。新的 OpenShift web 终端为 web 控制台带来了不可或缺的命令行工具，其 Linux 环境运行在部署在 OpenShift 集群上的 pod 中。web 终端消除了为本地终端安装软件、配置连接和验证的需要。这也使得在平板电脑和手机等设备上使用 OpenShift 变得更加容易，这些设备可能缺乏本地终端。

本文介绍了新的 OpenShift web 终端，包括如何安装和激活 web 终端操作器。

## 管理 OpenShift 集群的简单方法

新的 OpenShift web 终端包括用于处理集群的关键程序，包括:

*   用于全面 OpenShift 管理的`oc`工具。
*   `odo`，OpenShift 面向应用开发的精简工作流实用程序。
*   `kubectl`，核心 Kubernetes API 客户端。

用于 [Tekton CI/CD 框架](https://developers.redhat.com/blog/2020/08/14/introduction-to-cloud-native-ci-cd-with-tekton-kubecon-europe-2020/)、 [Helm 应用程序部署图](https://developers.redhat.com/blog/2020/07/20/advanced-helm-support-in-the-openshift-4-5-web-console/)和 [Knative 无服务器工作负载](https://developers.redhat.com/topics/serverless-architecture)的 CLI 客户端工具也已安装并准备运行。这些 OpenShift 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 工具受到通常的类 Unix、通用文本处理和 shell 脚本的支持。

一旦安装了 web 终端操作器，就可以从 OpenShift web 控制台报头上的命令提示图标( **> _** )访问 Web 终端，如图 1 所示。

[![A screenshot showing the command-prompt icon in the web console.](img/f544134f931bc94447cdfda7c86756f7.png "wticon")](/sites/default/files/blog/2020/09/wticon.png)Figure 1: The command prompt icon in the OpenShift web console.">

单击图标会在 OpenShift web 控制台的底部显示 web 终端框架，如图 2 所示。您可以调整终端的大小、位置或将其弹出到新的浏览器窗口或选项卡中。

[![A screenshot of the web terminal open in the web console.](img/832f230fb7297b65a9818a9061717699.png "Figure 2")](/sites/default/files/blog/2020/09/wtrunning.png)Figure 2: The new command-line terminal opens at the bottom of the OpenShift web console.">

## 如何激活 OpenShift 网络终端

OpenShift 版本 4.5.3 和更高版本支持新的 Web 终端操作符，该操作符管理集群上的终端环境。要激活 web 终端，请访问 **Web 控制台管理员体验**左侧栏中的 **OperatorHub** ，并搜索 **Web 终端**。安装 Web 终端操作器。

部署操作员后，以没有集群管理员角色的用户身份登录 web 控制台。单击屏幕右上角的命令提示符图标启动 web 终端。您已经准备好使用您最喜欢的`oc`或`odo`一行程序来构建、部署和管理您的集群工作负载，而无需离开您的浏览器。

注意:网络终端的技术预览版有一些限制。首先，集群管理员不能使用 web 终端，它只对特权较低的角色可用。其次，shell history 特性可以通过上下箭头键和其他 bash 机制调用以前的命令，但是在终端会话之间不会保存这些信息。

## 我们感谢您的反馈

社区反馈帮助我们不断改进 OpenShift 开发者体验。我们真的很想听到你的声音！参加我们的一个办公时间，或[完成这个调查](https://forms.gle/zDd4tuWvjndCRVMD8)，让我们知道您对 OpenShift web 控制台和新 web 终端的想法。您还可以加入 [OpenShift 开发者体验谷歌小组](https://groups.google.com/forum/#!forum/openshift-dev-users)，分享您的建议，获得不太适合您的帮助，并塑造 OpenShift 开发者体验的未来。

准备好开始了吗？[今天试试 OpenShift】。](http://www.openshift.com/try)

*Last updated: September 30, 2020*