# 您可能在 Red Hat 峰会上错过的与容器相关的内容

> 原文：<https://developers.redhat.com/blog/2019/06/04/container-related-information-you-might-have-missed-at-red-hat-summit>

如果你没有足够的运气参加最近的[红帽峰会](https://www.redhat.com/en/summit/2019) *或*你去了但没能参加所有与容器相关的会议，不用担心。我们与 Red Hat 容器首席技术产品经理 Scott McCarty 合作，为您概述您错过了什么。

## 为您的应用程序选择正确的容器基础映像

[Red Hat Universal Base Image(UBI)](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)为您提供了三种选择来构建[容器](https://developers.redhat.com/topics/containers/)，并在下面提供[Red Hat Enterprise Linux](https://developers.redhat.com/rhel8)(RHEL)的全部功能。目标是创建完全支持您的应用程序的尽可能小的图像。您可以根据要打包到容器中的应用程序来选择基本映像。例如，如果您有一个 Golang 或。NET 应用程序，该应用程序的所有依赖项都是内置的。这意味着您可以使用最小映像(`ubi-minimal`)，它包含`microdnf`，一个只支持安装、更新和删除功能的包管理器。它还包括一套最少的工具。

基本映像(`ubi`)允许您运行在 Red Hat Enterprise Linux 上运行的任何应用程序。它包含全功能的`yum`包管理器以及基本的操作系统工具，如`tar`、`gzip`和`vi`。讨厌你的人，请在下面的评论区文明地讨论。)如果需要在一个容器中运行多个服务，`ubi-init`在启动时运行`systemd`。要使用它，在构建时启用您的服务，您就可以开始了。

Scott 还介绍了各种图像和主机组合的支持选项。例如，如果您有一个经过认证的应用程序(有关应用程序认证信息，请参见 [Scott 的幻灯片](http://crunchtools.com/files/2019/05/Choosing-the-right-container-base-image-for-your-application.pdf))运行在一个基于 UBI 构建的容器中，并且所有内容都托管在 Red Hat 平台上，那么您有权获得最高级别的支持。当然，其他组合的支持度可能较低。

ubi 是容器工具箱的一个很好的补充。要了解更多信息，[幻灯片](http://crunchtools.com/files/2019/05/Choosing-the-right-container-base-image-for-your-application.pdf)可以在网上找到， [Scott 关于 Red Hat Universal Base Image](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) 的文章也是一个很好的资源。

## 构建生产就绪容器

在 Scott McCarty 和 Ben Breard 的演讲中，一个很好的主题是构建容器的五条戒律:

1.  **标准化**:确保每个人尽可能使用相同的基本映像。
2.  **最小化**:将图像中的内容限制为实际服务于工作负载的内容。
3.  **代表**:维护图像图层的责任应该由拥有该技术专业知识的人来承担。例如，您的中间件专家应该负责定义中间件层的`Dockerfile`。
4.  **过程**:通过掌舵图、可行的行动手册和操作人员，尽可能地将过程自动化。
5.  **迭代**:当你发现错误时，在代码中捕获那些来之不易的知识。

[看一下幻灯片](http://crunchtools.com/files/2019/05/Summit-2019_-Building-Production-Ready-Containers.pdf),了解大量重要信息和真实体验。

## RHEL 8 集装箱工具

Scott 和 Dan Walsh 涵盖了来自[开放容器倡议](https://www.opencontainers.org)的开源项目: [podman](https://podman.io) 、 [skopeo](https://github.com/containers/skopeo/blob/master/README.md) 和 [buildah](https://buildah.io) 。 [Dan 和 Scott 的幻灯片](http://crunchtools.com/files/2019/05/Red-Hat-Enterprise-Linux-8-Container-Tools.pdf)可供使用，另外，如果您访问[红帽峰会虚拟活动](http://stats.eventcore.com/wf/click?upn=x0IXmEraw-2BSAZWl0mHPX9iDyKwSVIjiK-2FaDAYDiMIfeBgTYKkvJOtt0rgrK0P3XN06O6KQabIRHUH-2BqK-2BgZJI-2FoIoB20OAAC6H0NpWHX6KI-3D_Pj5ETFMBU1yXtgiSsKbRxZjbVfYeUF5sPcVH0t2gHPpJT4a-2BK-2B-2FRwfUNNnOHT-2FH4wDZGGzzUWOGZWBuEK9oPeB0oNzFM4luz-2FndLLXAeX09Q-2FhemZ39BAFDmoC4Nde9NhARC75IqsFlT0pFW3-2FxViNLZ12OKly9ifAhQHQjfHtLzb5UQnn2I9KItRwJzJoAL7B0a51fYIc603jA-2FyZ4qvh3vKir5thLkaH87bWOlOAU0uC9vQz4Qw457Fya8Fpei3U3sNcsgV-2FVGhRP1xOwIqCiTVWSuv-2F4I-2BSMYQGlRET98lzbBrUl5lm2NsswioOWM)，您可以在“红帽企业 Linux 8 之路”专题讲座中找到该会议的视频。我们在[我们的容器页面](https://developers.redhat.com/topics/containers/)上也有很多资源。

如果你还没看过丹解释波德曼的好处，那就暂停你的生活，现在就去做吧。

## Linux 容器内部 2.0

这个全面的会议包括一个关于注册表的部分，指出了[码头](https://quay.io)和[红帽集装箱目录](https://access.redhat.com/containers/)的特点，包括为注册表中的每个图像计算的集装箱健康指数。这个非常有用的特性让图像消费者知道给定的图像是否有安全漏洞。虽然这是一个很棒的特性，但它确实让映像所有者有责任在发现漏洞和修复程序推出时继续重建和更新映像。(例如，你的作者刚刚发现他需要重建[为](https://quay.io/repository/dougtidwell/2048-stack?tab=info)[“在 Eclipse Che 中创建自定义堆栈”视频](https://developers.redhat.com/che-custom-stacks/)创建的 2048 图像。)

Scott 讲述了许多其他重要的主题，包括容器编排、容器标准和体系结构。正如你所料，[幻灯片](http://crunchtools.com/files/2019/05/Linux-Container-Internals-2.0.pdf)可以在网上找到。你也可以带着[一个互动的、动手的 Katacoda 实验室](https://learn.openshift.com/subsystems/)快速开始，或者查看 [Scott 的样本代码](https://github.com/fatherlinux/learn-katacoda)进行深入研究。

## 摘要

我们只是提供了 Scott 和其他人在 Red Hat Summit 上展示的与容器相关的伟大内容的一个例子。同样，查看[我们的容器页面](https://developers.redhat.com/topics/containers/)获取更多资源来帮助你开始。如果你对下一步想看什么有什么想法，请在下面的评论中告诉我们。

*Last updated: September 3, 2019*