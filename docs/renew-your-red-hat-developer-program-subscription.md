# 如何为个人续订 Red Hat Developer 订阅

> 原文：<https://developers.redhat.com/articles/renew-your-red-hat-developer-program-subscription>

[红帽开发者计划](https://developers.redhat.com/about)提供了许多会员福利，包括免费的红帽开发者个人订阅。该订阅通过[Red Hat software access for developers](https://developers.redhat.com/products)页面提供对 Red Hat 产品组合的访问。[红帽企业版 Linux](/products/rhel/overview) (RHEL)就是其中包含的产品之一。

面向个人的红帽开发者订阅为期一年。到期后，您必须重新注册，才能继续获得与订购相关的所有支持和权益。这种支持不会改变，开发者仍然可以免费获得。本文将指导您完成重新注册的过程，并回答有关该过程的常见问题。

**注:**参见[如何激活免费的 Red Hat Enterprise Linux 订阅](/blog/2021/02/10/how-to-activate-your-no-cost-red-hat-enterprise-linux-subscription)了解如何设置您的 RHEL 订阅。

## 您不能续订，但可以重新注册

首先要明白的是，在第一年之后，您不能为个人续订免费的 Red Hat Developer 订阅。与付费订阅不同，面向开发者的免费版的有效期为一年。

那么，开发者该怎么做呢？幸运的是，这很简单:*你可以重新注册*。是的，就这么简单。一旦您的开发人员订阅到期，只需重新注册并获得一个新的免费订阅。请注意，您必须等到当前套餐到期后才能续订。

## 为什么我们要求您每年重新注册一次？

Red Hat 开发者订阅对开发者是免费的，但是它提供的服务和支持对 Red Hat 不是免费的。所以，我们想鼓励下载 RHEL 的开发者实际使用它。一种方法是要求开发者每年重新注册一次他们的订阅。重新注册确认订阅仍然有效，这是为支持、安全、更新等付出的小小代价。

## 如何重新注册您的 Red Hat 开发者订阅

Red Hat 首席客户支持专家 Daniel Marshburn 提供了以下关于在一年订阅期满后重新注册个人 Red Hat Developer 订阅的指导:

1.  在 Chrome 的匿名窗口、Firefox 的私人窗口或 Edge 的 InPrivate 窗口中打开[developers.redhat.com](http://developers.redhat.com/)。
2.  使用您的 Red Hat 登录 ID 登录网站。
3.  确认所提供的条款和条件。
4.  注销所有 Red Hat 站点并关闭浏览器。
5.  等待 15 到 20 分钟，然后登录[access.redhat.com/management](http://access.redhat.com/management)。
6.  您现在应该可以在您的帐户上看到一个新的 Red Hat Developer for Individuals 订阅。

## 解决纷争

在接受新的条款和条件后，您可能需要删除并重新附加您的许可证，以便您的系统能够识别续订:

```
sudo subscription-manager remove --all
sudo subscription-manager unregister
sudo subscription-manager clean
sudo subscription-manager register
sudo subscription-manager refresh
sudo subscription-manager attach --auto
```

## 常见问题

以下是对常见问题的回答，这些问题涉及为期一年的 Red Hat Developer 个人订阅到期后会发生什么。

### 1.不马上重新注册怎么办？我会失去所有的软件和工具吗？

总之，*没有*。如果您在订阅到期后没有立即重新注册，将不会从您的 RHEL 实例中删除任何内容。不会删除任何内容。在您重新注册之前，您将没有资格获得更新和访问支持文档以及您可能需要的其他权益。但与此同时，你可以继续开发软件。

### 2.我创建的容器图像还能运行吗？

如果您已经使用 Red Hat 的通用基础映像(UBI)基础映像创建了容器映像，那么一切都准备好了。这些图像将继续运行，运行你编写的无 bug 软件，没有任何问题。

## 如果我有更多的问题怎么办？

我们是来帮忙的！如果你看完这篇文章后有任何问题，欢迎在下面留下你的评论。您也可以查看[免费的 Red Hat Enterprise Linux 个人开发者订阅常见问题解答](/articles/faqs-no-cost-red-hat-enterprise-linux)了解更多信息。

感谢您成为 Red Hat 开发者计划的成员。

*Last updated: January 11, 2023*