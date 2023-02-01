# 用 odo 交互模式创建 OpenShift 组件

> 原文：<https://developers.redhat.com/blog/2019/08/14/openshift-development-with-interactive-odo>

如果你熟悉 OpenShift Do ( [odo](https://developers.redhat.com/blog/2019/05/03/announcing-odo-developer-focused-cli-for-red-hat-openshift/) )，一个为 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 开发的以开发人员为中心的命令行工具，那么你知道它的主要目标之一是使快速迭代开发变得更容易。然而，即使是有经验的 odo 用户也可能不熟悉 odo 的交互模式，这种模式进一步简化了创建组件和服务的过程。

## 组件的交互模式

在 odo 中，我们认为应用程序由一个或多个*组件*组成。你可以把一个组件想象成类似于一个[微服务](https://developers.redhat.com/topics/microservices/)。每个组件都有特定的类型，odo 支持多种组件类型，比如 Node.js、Java 和 Python。运行`odo catalog list components`将给出您的集群支持的组件类型列表。

要创建一个组件配置，您至少需要告诉 odo 在创建组件时使用哪种组件类型。但是，您也可以指定其他几个选项，比如组件名、要公开的端口、环境变量等等。交互模式指导您确定这些选项中哪些可能适用于您的组件。

阅读关于交互模式的内容并不公平，所以这里有一个使用它为 Node.js 组件创建组件配置的实例，并公开名称`frontend-kiosk`和`port 8080`。

如果您已经知道 odo 的命令语法，那么您可以在一个命令中运行这些命令:

```
odo create nodejs frontend-kiosk --port 8080
```

但是如果你是 odo 的新手，喜欢有指导的过程，或者不熟悉一些论点，交互模式会非常有帮助。

## 服务的交互模式

一个*服务*通常是一个组件链接或依赖的数据库或其他服务，比如 MongoDB 或 Jenkins。这些服务来自 OpenShift 服务目录，必须在集群上启用该目录才能使用该功能。运行`odo catalog list services`将为您提供可用服务的列表。

创建一个服务配置可能比创建一个组件配置更复杂，因为服务可能有更多的选项，并且这些选项根据所创建的服务类型而有所不同。

下面是一个使用交互模式创建 MongoDB 数据库服务的示例:

## 你自己试试

无论你是一个没有尝试过交互模式的经验丰富的 odo 用户，还是你从来没有使用过 odo，都可以获得最新版本的 odo 并尝试一下！Odo 目前处于测试阶段，我们希望听到您对交互模式或任何其他使该工具更易于使用的功能的反馈。

要安装 odo，请进入项目页面，阅读[安装说明](https://github.com/openshift/odo#installing-odo)。另一种尝试 odo 的方法是通过这个[互动教程](https://developers.redhat.com/courses/openshift/odo-command-line/)。如果您有任何反馈，请在[项目的 GitHub 资源库](https://github.com/openshift/odo)上发帖。在 developers.redhat.com/openshift 了解更多关于 openshift 的[开发。](https://developers.openshift.com/openshift)

*Last updated: October 14, 2019*