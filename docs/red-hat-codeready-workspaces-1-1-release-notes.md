# Red Hat CodeReady Workspaces 1.1:发行说明

> 原文：<https://developers.redhat.com/blog/2019/04/25/red-hat-codeready-workspaces-1-1-release-notes>

我们很高兴推出[Red Hat code ready work spaces](https://developers.redhat.com/products/codeready-workspaces/overview)1.1 版，它提供了一个云开发人员工作区服务器和为团队和组织构建的基于浏览器的 IDE。Red Hat CodeReady Workspaces 1.1 为大多数流行的编程语言、框架和 Red Hat 技术提供了现成的开发人员堆栈。

这个版本的 Red Hat CodeReady 工作区引入了:

*   与 Red Hat OpenShift 4.0 的兼容性
*   在断开的环境中安装
*   OpenShift OAuth 和集群证书的简化配置

Red Hat CodeReady Workspaces 1.1 现在可以在 [Red Hat 容器目录](https://access.redhat.com/containers/)中获得。从 3.11 版本开始，按照[管理指南](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/1.1/html/administration_guide/installing_codeready-workspaces)中的说明，你可以在 OpenShift 容器平台或 OpenShift 专用平台上安装它。

对于 Red Hat OpenShift 4.0，CodeReady Workspaces 1.1 现已在 OperatorHub 的开发者预览版中提供。基于利用操作员生命周期管理器的新操作员，安装流程变得更加简单，无需离开 OpenShift 控制台即可完成。如果你已经有了 Red Hat OpenShift 4.0，从 OperatorHub 获取 Red Hat CodeReady Workspaces 1.1，并遵循[专用文档](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/1.1/html/administration_guide/installing_codeready-workspaces-from-operator-hub)。

## **Red Hat code ready work spaces 1.1 中的新特性**

### **使用 OpenShift OAuth 认证用户**

为了对用户进行身份验证，可以将 Red Hat CodeReady 工作区配置为依赖 OpenShift OAuth，因此它们只需要一次身份验证就可以访问 OpenShift 控制台或 CodeReady 工作区。

### **证书配置**

如果您正在将 Red Hat CodeReady 工作区部署到配置有证书(自签名或公共证书)的群集上，部署现在将自动检测和配置 CodeReady 工作区的那些证书。

### **断开安装**

安装和更新 Red Hat CodeReady 工作区会从 Red Hat 容器目录中提取图像。使用您自己的容器注册表配置 CodeReady 工作区现在更容易了，这简化了在不连接或受限环境中的安装。

## **红帽 OpenShift 4.0 开发者预览版**

Red Hat CodeReady Workspaces 现在与 Red Hat OpenShift 4.0 兼容，并有一个专门的操作员利用操作员生命周期管理器。现在可以在开发者预览版中使用，CodeReady 工作区可以从 OpenShift 4.0 操作中心安装。

## **运营商生命周期管理器对 Red Hat CodeReady 工作区的支持**

[Operator life cycle Manager](https://github.com/operator-framework/operator-sdk)(OLM)项目是 Operator Framework 的一个组件，Operator Framework 是一个开源工具包，以有效、自动化和可扩展的方式管理 Kubernetes-native 应用程序，称为 Operators。

在 Red Hat OpenShift 容器平台中，OLM 帮助集群管理员安装、升级和授予在集群上运行的操作员访问权限。OpenShift Container Platform web 控制台还提供了一个市场，管理员可以在其中访问一个管理运营商的目录，包括 Red Hat 支持的运营商。

OLM 使用户能够执行以下操作:

*   将应用程序定义为封装了需求和元数据的单一 Kubernetes 资源。
*   使用依赖解析自动安装应用程序，或者只使用 kubectl 手动安装应用程序。
*   使用不同的批准策略自动升级应用程序。

Red Hat CodeReady Workspaces 现在获得了一个利用 OLM 功能的 Red Hat 支持的操作符。
您可以在以下视频中了解更多信息:

https://www.youtube.com/watch?v=HwhkGYsgVqs

如果你已经有了 OpenShift 4.0，从 OperatorHub 获取 Red Hat CodeReady Workspaces 1.1，并遵循[专用文档](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/1.1/html/administration_guide/installing-codeready-workspaces-from-operator-hub)。

## **文档改进**

此版本还包括以下文档:

*   创作自定义工作空间堆栈
*   在受限环境中安装
*   使用多个工作空间运行 CRW 时调整设置
*   在 Red Hat CodeReady 工作区中使用 VCS

## **支持的环境**

从版本 3.11 开始，Red Hat code ready work spaces for OpenShift 可以安装在 Red Hat Openshift 容器平台或 open shift 专用平台上。

*Last updated: April 30, 2019*