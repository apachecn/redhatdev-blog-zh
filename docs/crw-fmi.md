# 与 Red Hat OpenShift Dev Spaces(以前的 CodeReady 工作区)共享您的容器开发环境

> 原文：<https://developers.redhat.com/crw-fmi>

“它在我的机器上工作。”如果你和其他人一起写代码，为其他人写代码，或者和其他人一起写代码，你至少已经说过这些话一次了。几个月前，我在我的机器上设置了一个库或包或环境变量或*什么的*,从那以后我就再也没想过这个问题。所以代码对我来说是有效的，但是可能要花很长时间才能弄清楚你的机器上缺少了什么。

## 与工厂共享工作空间

Red Hat OpenShift Dev Spaces 建立在开源 Eclipse Che 项目(T1)的基础上，通过提供安全、可共享的开发人员(T2)工作区(T3)解决了这个问题(以及我们稍后将讨论的其他几个问题)。这些工作区包括编码、构建、测试、运行和调试应用程序所需的所有工具和依赖项。整个产品运行在一个 OpenShift 集群中(本地或云中)，所以不需要在您的机器上安装任何东西。或者我的。

OpenShift Dev Spaces 也让入职变得简单。您可以创建一个 [devfile](https://devfile.io/docs/devfile/2.1.0/user-guide/) 来描述您项目的工作空间。您团队中的任何授权人员都可以加载 devfile 来获得该工作区的新副本。这个 devfile 可以定义要克隆的其他项目、要运行的命令以及构建、运行和部署应用程序所需的开发环境。

## 通用显影剂图像

为了增强您的工作空间，OpenShift Dev Spaces 附带了一个多用途的一体化通用开发人员映像(UDI ),它取代了对一组堆栈映像的需要。OpenShift Dev Spaces UDI 映像包括语言运行时、编译器、包管理器、工具和实用程序，支持诸如 [Java](https://developers.redhat.com/topics/java) (包括 [Quarkus](https://developers.redhat.com/products/quarkus/overview) )、 [Node.js](https://developers.redhat.com/topics/nodejs) 、 [Python](https://developers.redhat.com/topics/python) 、 [Golang](https://developers.redhat.com/topics/go) 等语言。它还支持红帽环境和运行时，如[红帽 Fuse](https://developers.redhat.com/products/fuse/overview) 和[红帽 JBoss 企业应用平台](https://developers.redhat.com/products/eap/overview)。

想用自己的内容扩展 UDI 吗？您可以基于 UDI 映像构建一个容器映像，包括您的定制开发工具，省略您不需要的东西，并且一旦发布到容器注册中心，就可以通过 devfile 在您的团队中使用这个新映像。假设您团队中的每个人都使用那个定制映像，您知道每个人都有相同的库、运行时和工具——以及相同级别的安全补丁和错误修复！

OpenShift Dev Spaces 提供了一个全功能的 IDE，包括代码完成、语法突出显示、重构、调试和您期望从一个优秀的开发环境中得到的一切。

很快，我们将增加对更多 ide 的支持，包括 IntelliJ IDEA Community Edition 和 VSCodium，这是 Visual Studio 代码的完全开源版本。

顺便说一下:OpenShift Dev Spaces 提供了一个全功能的 IDE，包括代码完成、语法突出显示、重构、调试和您期望从一个优秀的开发环境中得到的一切。(尽管您可以使用 SSH 从 Eclipse 和其他工具访问您的工作区，如果您愿意的话。)

## 保护您的代码

另一个开发问题是保护存储在笔记本电脑上的源代码和其他知识产权。使用 OpenShift Dev Spaces，代码的工作副本在开发人员工作区运行的 OpenShift 集群上集中管理。本地机器上不再有任何东西。您的笔记本电脑可能在任何地方，但您总是知道您的源代码在哪里。

## 开始吧！

最重要的是，注册测试版很容易。[访问产品页面](/products/openshift-dev-spaces/overview)获取代码和您需要了解的关于产品的一切信息。

OpenShift Dev Spaces 为您提供了更高的安全性、更快的上线速度，并确保您的代码也能在他们的机器上运行。

*Last updated: August 4, 2022*