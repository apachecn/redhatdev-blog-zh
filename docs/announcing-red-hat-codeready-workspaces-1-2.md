# 宣布 Red Hat CodeReady Workspaces 1.2

> 原文：<https://developers.redhat.com/blog/2019/06/05/announcing-red-hat-codeready-workspaces-1-2>

我们很高兴推出 Red Hat CodeReady Workspaces 版本 1.2，它提供了一个云开发人员工作区服务器和为团队和组织构建的基于浏览器的 IDE。 [CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview) 包括针对大多数流行编程语言、框架和 Red Hat 技术的现成开发人员堆栈。

## **发布概述**

Red Hat CodeReady Workspaces 1.2 引入了:

*   与[红帽 OpenShift 4.1](https://developers.redhat.com/openshift/) 的兼容性
*   基于 UBI 8 的全新开发人员堆栈

Red Hat CodeReady Workspaces 1.2 今天在 [Red Hat 容器目录](https://access.redhat.com/containers/)中可用。从 3.11 版本开始，按照[管理指南中的说明，将其安装在](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/1.2/html/administration_guide/installing_codeready-workspaces) [OpenShift 集装箱平台](https://developers.redhat.com/openshift/)或 [OpenShift 专用](https://www.openshift.com/products/dedicated/)上。

对于 OpenShift 4.0 或 4.1，CodeReady Workspaces 1.2 今天可以从 OperatorHub 获得。基于利用操作员生命周期管理器的新操作员，安装流程变得更加简单，无需离开 OpenShift 控制台即可完成。如果您已经有了 Red Hat OpenShift 4.0 或 4.1，请从 OperatorHub 获取 CodeReady 工作区，并遵循[专用文档](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/1.2/html/administration_guide/installing-codeready-workspaces-from-operator-hub)。

## **显著增强**

### **与 OpenShift 4.x 的兼容性**

CodeReady Workspaces 现在与 OpenShift 4.0 和 OpenShift 4.1 兼容，并有一名专门的操作员利用操作员生命周期管理器。现在可以在开发者预览版中使用，CodeReady 工作区可以从 OpenShift 4.0 操作中心安装。

### **基于 UBI 8 的新开发者堆栈**

CodeReady Workspaces 现在为基于 Red Hat Universal Base Image (UBI)的不同技术提供默认的开发人员堆栈。所有现有的堆栈已经更新到 UBI 8，所以你可以利用最新和最大的官方红帽容器图像。

![CodeReadyWorkspaces stacks](img/14ad3ea3c535a9ff57d66eb71780d4ce.png)

除了这些变化，我们还为 Node.js 10 添加了一个新的开发人员堆栈。其他默认堆栈已经用不同嵌入式技术的最新版本进行了更新。

## **支持的环境**

从版本 3.11 开始，OpenShift 的 Red Hat CodeReady 工作区可以安装在 Openshift 容器平台或 OpenShift 专用平台上。

*Last updated: June 18, 2019*