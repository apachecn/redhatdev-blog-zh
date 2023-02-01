# 集装箱化的三种方式。NET 在 Red Hat OpenShift 上的应用

> 原文：<https://developers.redhat.com/blog/2021/03/16/three-ways-to-containerize-net-applications-on-red-hat-openshift>

当微软[在 2014 年 11 月](https://devblogs.microsoft.com/dotnet/net-core-is-open-source/)宣布[。NET 框架](/topics/dotnet)将成为[开源](/topics/open-source/)。NET 开发者的世界改变了。这不是一个新方向的轻微漂移；这是一场影响深远的构造运动。

首先，它的定位。NET 开发者参与到快速发展的 Linux 容器世界中来。随着容器开发的成熟，包括了诸如 [Kubernetes](/topics/kubernetes) 、 [Knative](/topics/serverless-architecture) 和 [Red Hat OpenShift](/products/openshift/overview) 等技术。NET 开发人员面临的机遇和挑战是将现有的应用程序转移到使用[微服务](/topics/microservices/)和[容器](/topics/containers)。

幸运的是，我们有选择。本文描述了。NET 开发 OpenShift: Linux 容器、Windows 容器和 OpenShift 容器-本机虚拟化。

## Linux 是容器，容器是 Linux

。NET Core 是通往。容器中的网。将. NET 核心应用程序容器化就像编写 Docker 文件并使用 Podman 或 Docker 构建映像一样简单。虽然它需要基于 Linux 的计算机或适当的基于 Windows 的环境(Hyper-V 或 Linux 2 的 Windows 子系统)，但不需要特殊的步骤或 OpenShift 技术。构建图像并将其导入 OpenShift。

使用 [Linux](https://developers.redhat.com/topics/linux) 容器似乎是新的(“绿地”)应用程序的首选途径，因为不需要移植或重写。如果你想移植现有的应用程序，有两个工具可以帮助你:微软的 [ApiPort](https://github.com/Microsoft/dotnet-apiport) 和亚马逊的 [Porting Assistant for。网](https://aws.amazon.com/porting-assistant-dotnet/)。

如果你想开始写作。NET 应用程序在容器中运行，同时保持现有的 monolith 在 Windows 中运行，我建议学习一下[扼杀者模式](https://github.com/redhat-developer-demos/cloud-native-compass/blob/master/strangler-pattern.md)。

如果您想要启动一个新的应用程序，或者如果您可以轻松地移植您当前的应用程序，或者如果您想要使用 strangler 模式。NET Core 运行 Linux 容器才是正确的选择。

但是如果你想移动一个现有的。NET 框架应用程序？

## Windows 容器

期待已久的 [Windows](https://developers.redhat.com/blog/category/windows/) containers 技术已经准备好了，您可以在 OpenShift 上运行它。使用与 Linux 容器相同的管理模型和管理平面，Windows 容器允许您在 Windows 操作系统上构建和运行映像。

这有什么实际意义？跑步能力怎么样。容器中的. NET 框架应用程序？是的，*框架*。不是核心；框架。让你的想象力尽情发挥一会儿吧。构建一个 Windows 容器映像，运行您的 IIS 网站就像`docker run -d -p 80:80 my-iis-website`一样简单。

当然，使用这种方法时需要考虑一些事情。我将在下一篇关于 OpenShift 上的 Windows 容器的文章中探讨这些问题。敬请关注。

## 开放式 CNV

OpenShift 容器本地虚拟化技术(CNV)建立在 T2 kube virt 技术的基础上。KubeVirt 项目允许您使用 Kubernetes 控制虚拟机(VM ),将它视为一个容器。该技术的诸多优势，加上 OpenShift，包括无需任何代码更改的即时提升和转移、由 OpenShift 提供的基于角色的访问控制(RBAC)和网络策略，等等。简而言之，您可以在保持 Windows VM 运行的同时获得 Kubernetes 和 OpenShift 的所有优势。

如果你想移动你的遗产。NET Framework 应用程序来打开 Shift 并立即从它的特性中获益*，这条路线可能是最好的。它为您评估前进的道路赢得了时间，这可能包括容器—Windows、Linux 或两者都有。*

 *## 还会有更多

本文只是对集装箱化最新技术的一个简单介绍。NET 应用程序。你个人的道路取决于你的情况、预算、时间等等。为了帮助做出更容易的决定，可以再找三篇关于这个主题的文章——每条路径一篇: [Linux 容器](/blog/2021/04/15/containerize-net-for-red-hat-openshift-linux-containers-and-net-core/)、Windows 容器和带有 Windows VM 的 OpenShift CNV。

*Last updated: April 15, 2021**