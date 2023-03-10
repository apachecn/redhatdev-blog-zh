# 红帽通用基础图像:如何在 3 分钟或更短时间内完成

> 原文：<https://developers.redhat.com/blog/2019/07/29/red-hat-universal-base-image-how-it-works-in-3-minutes-or-less>

当我们在 5 月宣布 [Red Hat Enterprise Linux 8](https://developers.redhat.com/blog/2019/05/07/red-hat-enterprise-linux-8-now-generally-available/) 时，我们还[宣布](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)所有 RHEL 8 基本操作系统映像，以及许多新的 RHEL 7 操作系统映像，将根据新的[通用基本映像最终用户许可协议(EULA)](https://www.redhat.com/licenses/EULA_Red_Hat_Universal_Base_Image_English_20190422.pdf) 提供。如果 UBI 对你来说是新的，这篇文章总结了 UBI，解释了你为什么想要使用它，并提供了一组资源让你开始使用 UBI。如果你有问题，我们刚刚发布了一个全新的 UBI 常见问题。

## UBI 是什么？

Red Hat Universal Base Images (UBI)是符合 OCI 标准的基于容器的操作系统映像，带有补充的运行时语言和可自由再分发的软件包。像以前的 RHEL 基础映像一样，它们是从 Red Hat Enterprise Linux 的一部分构建的。UBI 映像可以从 [Red Hat 容器目录](https://access.redhat.com/containers/)中获得，并且可以在任何地方构建和部署。

而且，你不需要成为一个红帽客户来使用或重新分发它们。真的。

## 包括什么？

红帽通用基础图像包括三件事:

*   提供了一组三个基本映像(最小、标准和多服务),为各种使用情形提供最佳起点。[了解更多。](https://developers.redhat.com/products/rhel/ubi/#assembly-field-sections-18555)
*   一组语言运行时映像(PHP、Perl、Python、Ruby、Node.js)使您能够像 Red Hat 构建的容器映像一样自信地立即开始编码。
*   一组相关的 YUM 存储库/通道包括 RPM 包和更新，允许您随时添加应用程序依赖项和重建 UBI 容器映像。

## UBI 在行动:如何在短短 3 分钟内完成

有了 UBI，你可以在一个平台上封装一个应用，并在另一个平台上共享部署。在这个视频中，斯科特·麦卡蒂在短短三分钟内解释了它的工作原理。

https://www.youtube.com/watch?v=VG7Y1mjVIE0

## 为什么要用 UBI？

**TL；dr** —通过构建便携式应用程序节省开发时间。

在 UBI 之前，你必须为每个需要部署的目标打包你的容器化应用。考虑到这一点，容器并不像现在的 zip 或 gif 文件那样具有可移植性。UBI 允许您一次创建映像，并使用企业级软件包部署到任何地方。另一种方法是使用不可信的、不可靠的和/或劣质的包，这些包不能满足企业级的需求。这种方法从一开始就被打破了。

### 面向 ISV 的特别提示—Red Hat 容器认证

客户对应用程序的安全性感到紧张。Red Hat 通过提供容器认证解决了这一问题，让任何想要尝试和/或使用您的软件的人充满信心。这是一项免费服务——只需加入[红帽技术合作伙伴计划](https://developers.redhat.com/products/rhel/ubi/#assembly-field-sections-18515)。

## 开始

以下是开始的三个步骤:

1.  从[红帽容器目录](https://access.redhat.com/containers/)的 UBI7 或 UBI8 下载，按照 Scott 在上面视频中展示的那样做。
2.  接下来，用一种运行时语言做一些事情:PHP、Perl、Python、Ruby 或 Node.js。你也可以在目录中找到这些语言(RHEL 7 或 8 有单独的目录)。
3.  给我们反馈；请参阅下面的联系方式。

## 资源

*   [UBI 信息页面。](https://developers.redhat.com/products/rhel/ubi/)
*   [UBI 常见问题](https://developers.redhat.com/articles/ubi-faq/)。
*   通过电子邮件将 UBI 问题和/或反馈发送至 redhat DOT com。
*   [加入](https://www.redhat.com/mailman/listinfo/ubi)UBI 社区邮件列表，关注 UBI 讨论。

*Last updated: September 3, 2019*