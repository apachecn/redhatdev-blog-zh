# 第 1 部分:使用 Knative 介绍无服务器

> 原文：<https://developers.redhat.com/coderland/serverless/serverless-knative-intro>

Knative 无服务器环境允许您将代码部署到 Kubernetes，但是不消耗任何资源，除非您的代码需要做一些事情。使用 Knative，您可以通过将代码打包成 Docker 映像并将其提交给系统来创建服务。您的代码只在需要时运行，Knative 自动启动和停止实例。本文向您展示了如何利用这项技术。

![coderland_1](img/b0cff628902278bdd4486a0c563dfb6a.png)

## 编译驱动程序

《编译驱动程序》是 Coderland 主题公园的一个新的惊险刺激项目。它从将骑手吊到一个巨大的塔顶开始。从那里，他们穿过空气冲向游乐设施底部的钢筋混凝土垫。不幸的是，随着汇编司机的安全记录已成为众所周知的，乘客远远低于预期。这里有一个视频解释了完整的场景:

[https://www.youtube.com/embed/R8PGrhfVWTc?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/R8PGrhfVWTc?autoplay=0&start=0&rel=0)

## **你的任务**

随着公园的收入目标处于危险之中，管理层转向你，Coderland 最有才华的开发人员，创建一个摄影亭，捕捉高兴的骑手无助地摔倒在地时的表情。您将创建一个无服务器函数，从安装在游乐设备旁边的摄像机获取原始图像数据，然后在图片上标记消息、日期和 Coderland 徽标。当然，游客可以购买一张传家宝级的照片，当他们摇摇晃晃地离开游乐设施时，在这个过程中填满公园的金库。

整体流程如下所示:

[https://www.youtube.com/embed/nH5sYr4Pgk8?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/nH5sYr4Pgk8?autoplay=0&start=0&rel=0)

你的任务是拍一张快乐的科德兰客人的照片:

![Your assignment is to take something like this picture of a happy Coderland guest](img/bc9153aba006fbd45f9af76a523bdeb0.png "Figure 1: Your assignment is to take something like this picture of a happy Coderland guest...")

...并把它变成这个无价的纪念品，在 Coderland Swag 商店售价 19.95 美元: **![](img/38b2d1d23c31d53241cb2e782ef052a0.png)**

![Add captions and a logo to the image](img/bb769a144625b52eb79e892a76701bdf.png "Figure 2: Add captions and a logo to the image.")

许多编译驱动的早期乘客要求他们的苦难的纪念照片，一些他们可以与他们的朋友，亲人和律师分享的东西。因此，我们相信 Coderland 摄影展将会取得巨大的成功。

## **简单说说无服务器**

使用无服务器计算的主要原因是经济上的，而不是技术上的。在编译司机照片亭的例子中，每当我们发现足够多的勇敢的人愿意被绑在车上时，代码只需要运行一两秒钟。这意味着即使在繁忙的一天，代码运行的总时间也不会超过几分钟。设置一个 24/7 运行的服务器来托管图像处理代码意味着我们在大多数时间里都在为没有得到利用的处理能力买单。更麻烦的是，如果我们维护我们自己的服务器，我们就要负责更新补丁和升级。

这就是无服务器计算的强大之处。我们不按分钟付费，而是按调用次数付费。因此，与大多数云计算一样，我们只为我们使用的东西付费，但我们使用的东西比以前少得多。我们将向 Knative 部署一个服务，Knative 将根据需要启动和停止该服务。如果我们在一段时间内不调用图像处理代码，Knative 就会关闭它。当你使用无服务器计算时，你会经常听到这个短语。这项服务仍然存在，只是没有运行。当我们调用服务时，Knative 会启动它。而且，正如“无服务器”这个词所暗示的，我们没有自己的服务器来管理或维护。

## **从这里到那里**

为了帮助您完成任务，我们将经历四个步骤:

1.  回顾处理实际照片的图像处理代码(在第 2 部分中讨论)

2.  看一看 React 前端，它可以让您轻松地测试图像操作代码(在第 2 部分中讨论)

3.  将图像操作代码部署到 Kubernetes 内部的 Knative 运行中(在第 3 部分中讨论)

4.  解决阻止 React 前端调用 Knative 内部的服务的问题(在第 3 部分中讨论)

一路上，我们有更多的视频，git repos，Docker 图片和其他有用的资源。祝你好运，玩得开心！

## 下一步是什么

我们希望听到您对此内容的评论和问题。你可以在 coderland@redhat.com 找到我们。

除此之外，本系列还有两篇文章:

*   [第 2 部分:构建无服务器服务](/coderland/serverless/building-a-serverless-service "Part 2: Building a Serverless Service")
*   [第 3 部分:将无服务器服务部署到 Knative](/coderland/serverless/deploying-serverless-knative "Part 3: Deploying a Serverless Service to Knative")

要知道这两篇文章是独立的。您可以运行图像处理服务，而无需将它部署到 Knative，并且可以直接跳到 Knative 文章，而无需浏览代码。

但是说真的，你应该忽略你所有的责任和义务，现在就通读这两篇文章。玩得开心！

*Last updated: April 21, 2021*