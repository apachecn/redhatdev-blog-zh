# Red Hat 简化了 Red Hat Enterprise Linux 包的容器开发和再分发

> 原文：<https://developers.redhat.com/blog/2020/02/26/red-hat-simplifies-container-dev-and-redistribution-rhel-packages>

[Red Hat Partner Connect](https://developers.redhat.com/techpartner/) 计划中的应用程序开发人员现在可以构建他们的容器应用程序，并从全套 Red Hat Enterprise Linux (RHEL)用户空间包(非内核)中重新分发它们。这使得 UBI 上的包数增加了近三倍。

当我们 [在 2019 年 5 月推出](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) 红帽通用基础映像(UBI)时，我们为红帽合作伙伴提供了自由使用和再分发大量 RHEL 包的能力，这些包可以部署在红帽和非红帽平台上。这使得开发人员能够构建安全、可靠、可移植的基于容器的软件，然后可以部署到任何地方。对此，我们的反馈非常积极，对此我们表示感谢，但我们了解到您还需要更多，因此我们与 Red Hat Partner Connect 成员分享这份高级预览，以帮助您制定计划。

## **Red Hat 技术合作伙伴的扩展和独家再分发权利**

我们很高兴[宣布](http://redhat.com/en/about/press-releases/red-hat-extends-partner-offerings-drive-open-hybrid-cloud-innovation)扩展合作伙伴 条款和条件 授予 Red Hat 技术合作伙伴在您构建基于 UBI 的映像 时免费使用和再分发所有 Red Hat Enterprise Linux 用户空间包的权利。现在有了三倍多的 RHEL 软件包，您可以简化您的容器和操作符开发，并通过 Red Hat 和非 Red Hat 注册表自由地重新分发您的基于容器的软件。这仅适用于参与并完成[红帽容器认证](https://connect.redhat.com/partner-with-us/red-hat-container-certification)的红帽合作伙伴。

## **开始使用**

要使用和重新分发所有 RHEL 用户空间包，注册参加红帽容器认证计划的[红帽合作伙伴连接成员需要接受更新的容器附录并同意通过该计划认证他们的软件。为了访问所有适用的 RHEL 软件包，需要在拥有有效订阅的 RHEL 容器主机上构建容器。](https://connect.redhat.com/partner-with-us/red-hat-container-certification)

## **常见问题解答**

1.  **合作伙伴需要满足哪些具体要求才能利用 Red Hat Container 认证的扩展范围？**
    1.  合作伙伴容器映像必须使用 RHEL 7 UBI 或 RHEL 8 UBI 作为基础映像。
    2.  合作伙伴必须接受 [Red Hat 技术合作伙伴](https://developers.redhat.com/techpartner/) Connect 协议。当前协议没有变化，因此如果已经签署，就没有必要重新签署。
    3.  合作伙伴必须参与 Red Hat 容器认证，这意味着接受更新的容器附录条款作为认证工作流程的一部分，并完成容器认证。这个更新的容器附录允许使用除内核之外的所有 RHEL 软件包，只要产生的作品不构成商业 Red Hat 产品的实质性复制。
2.  这个扩展的范围与 Red Hat Universal Base Images (UBI)有什么不同？
    任何人都可以使用和重新发布 UBI 软件包。无论他们是否订阅了 Red Hat Enterprise Linux。基于 RHEL 的 UBI 是 RHEL 用户空间包的一个子集。这个公告让红帽技术合作伙伴使用和重新分发所有 RHEL 用户空间包，包括 UBI。
3.  **认证工作流程有何不同？**
    您提交软件进行认证的方式没有变化。
4.  编码实践会发生什么变化？
    无一。像今天一样继续使用 UBI 基本图像。从各种 RHEL 回购中添加所需的任何用户空间包。
5.  UBI 是否已更改为包含 RHEL 内核？
    不，UBI 是一样的，任何人都可以使用和自由再分发基于 UBI 的图片，无论他们是否有红帽订阅。这份更新的合作伙伴协议允许技术合作伙伴使用和重新分发任何由基于 UBI 的映像构建的 RHEL 用户空间包。

## **资源**

*   [红帽集装箱认证](https://connect.redhat.com/partner-with-us/red-hat-container-certification)
*   [红帽集装箱认证数据表](https://rhc4tp-cms-prod-vpc-76857813.s3.amazonaws.com/s3fs-public/RH-Container-Cert-Datasheet-US%20%281%29.pdf)
*   [红帽集装箱认证合作伙伴指南](https://redhat-connect.gitbook.io/partner-guide-for-red-hat-openshift-and-container/)
*   [红帽通用基础图片 UBI 页面](https://developers.redhat.com/products/rhel/ubi/) 和 [UBI 常见问题](https://developers.redhat.com/articles/ubi-faq/)
*   [了解如何成为 Red Hat 合作伙伴](https://developers.redhat.com/techpartner/)

## **了解更多信息**

如果您想了解更多信息， [请加入](https://www.brighttalk.com/webcast/14777/384281?utm_source=Red+Hat&utm_medium=brighttalk&utm_campaign=384281) 我们，参加我们 3 月 18 日的网络研讨会“RHEL 简化的容器开发和再分发”，我们将在会上讨论这些扩展的再分发功能如何增强您的软件开发和业务。 [今天登记](https://www.brighttalk.com/webcast/14777/384281?utm_source=Red+Hat&utm_medium=brighttalk&utm_campaign=384281) 。

*Last updated: June 3, 2022*