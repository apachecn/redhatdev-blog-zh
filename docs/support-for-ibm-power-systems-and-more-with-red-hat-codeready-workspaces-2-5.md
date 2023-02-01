# 使用 Red Hat CodeReady Workspaces 2.5 支持 IBM Power Systems 等

> 原文：<https://developers.redhat.com/blog/2020/12/04/support-for-ibm-power-systems-and-more-with-red-hat-codeready-workspaces-2-5>

[Red Hat CodeReady work spaces 2.5](https://developers.redhat.com/products/codeready-workspaces/overview)现已推出。本文介绍了对 IBM Power Systems 的支持以及 CodeReady Workspaces 2.5 中新的单主机模式。我们还简要讨论了这个版本中对 [Red Hat OpenShift 4.6](https://developers.redhat.com/products/openshift/overview) 和语言更新的支持。

**注意**:code ready work spaces 2.5[在红帽 OpenShift 3.11 和红帽 OpenShift 4.5 及更高版本](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.5/html/release_notes_and_known_issues/installing_and_deploying_codeready_workspaces)上可用。

## 关于代码就绪工作区

CodeReady Workspaces (CRW)基于一个开源项目 [Eclipse Che](https://www.eclipse.org/che/getting-started/cloud/?sc_cid=701f2000000RtqCAAS) 。CodeReady 工作区通过近乎即时的入职和一致的、类似生产的开发环境，显著提高了开发人员的工作效率。开发人员可以在 Red Hat OpenShift 和其他类型的开发上使用 CodeReady 工作区进行云原生开发。

## 对 IBM Power 系统的支持

从 2.5 版本开始，您可以作为运行在 IBM Power Systems 上的[操作员](https://developers.redhat.com/topics/kubernetes/operators)来部署 CodeReady 工作区。CodeReady Workspaces 命令行界面(CLI)，`crwctl`在 [Windows](https://developers.redhat.com/blog/category/windows/) 、macOS 和 [Linux](https://developers.redhat.com/topics/linux) 机器上工作。您可以使用 CLI 将 CodeReady 工作区部署到运行在 IBM Power Systems 上的集群中。

注意，部署到 IBM Power Systems 上的 OpenShift 集群并在其上运行是一个技术预览特性，在 2.5 版本中并不完全支持。从这个版本开始，CodeReady Workspaces 不正式支持在 IBM Power Systems 上部署到断开连接的 OpenShift 集群。此外，CodeReady Workspaces 在 IBM Power 系统上提供了较少的 devfiles 和示例，因为一些语言(如[。IBM 架构不支持 NET Core](https://developers.redhat.com/topics/dotnet) 。

## 单主机模式

单主机模式使用名为`gateway`的子类型激活暴露策略，该子类型使用内部运行反向代理的特殊 pod 来路由请求。使用单主机策略时，所有工作区都部署到主 CodeReady 工作区服务器域的子路径中。当您处于单主机模式时，不需要使用通配符传输层安全性(TLS)证书。

## 支持 OpenShift 4.6

我们在这个版本中增加了对 [OpenShift 4.6](https://www.openshift.com/blog/red-hat-openshift-4.6-is-now-available) 的支持，包括将 sidecar 映像中的`oc`命令行二进制的嵌入式版本更新到 OpenShift 4.6。开发人员应该注意到，OpenShift 4.6 版本包括一种创建和使用操作符的新方法，它取代了旧方法。有关 OpenShift 4.6 中运算符的更多信息，请参见 OpenShift 4.6 [运算符框架打包格式](https://docs.openshift.com/container-platform/4.6/operators/understanding/olm-packaging-format.html)文档。

## 语言和平台更新

我们还更新了多用途`plugin-java8`和`plugin-java8-openj9`边车，以包括最新的 [OpenJDK 8](https://developers.redhat.com/products/openjdk/download) 和 [Eclipse OpenJ9 8](https://github.com/eclipse/openj9) 维护版本。作为额外的更新，我们用 [Node.js 12](https://nodejs.org/en/blog/release/v12.0.0/) 替换了 [Node.js](https://developers.redhat.com/blog/category/node-js/) 10。最后，请注意 CodeReady Workspaces 2.5 支持最新的 [Python](https://developers.redhat.com/blog/category/python/) 3.6 维护版本。

**注意**:参见 CodeReady Workspaces 2.5 文档，了解 CRW 2.5 版本中的[已知问题和建议解决方法](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.5/html-single/release_notes_and_known_issues/index#known-issues)。

## 尝试 CodeReady Workspaces 2.5

CodeReady Workspaces 2.5 现已在 OpenShift 3.11、OpenShift 4.5 和 OpenShift 4.6 上推出:

*   通过 CRW 命令行在 OpenShift 3.11 上为 AMD64 和 Intel 64 (x86_64) [安装 CodeReady 工作区。](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.5/html-single/installation_guide/index#installing_codeready_workspaces_on_openshift_container_platform_3_11)
*   从 OpenShift OperatorHub 安装 OpenShift 4.5 for IBM Z [上的 CodeReady 工作区。](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.5/html/installation_guide/installing-codeready-workspaces_crw#installing-codeready-workspaces-on-openshiftt-4-using-operatorhub_crw)
*   直接从 OpenShift OperatorHub 为 AMD64 和 Intel 64 (x86_64)以及 IBM Power Systems [安装 OpenShift 4.5 或 OpenShift 4.6 上的 CodeReady 工作区。](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.5/html-single/installation_guide/index#installing-codeready-workspaces_crw)

参见 CodeReady Workspaces 文档，了解其他受支持的平台。您还可以[下载 CodeReady Workspaces CLI](https://developers.redhat.com/products/codeready-workspaces/download) 并访问 Red Hat Developer 上的 [CodeReady Workspaces 主页](https://developers.redhat.com/products/codeready-workspaces)。

*Last updated: December 1, 2020*