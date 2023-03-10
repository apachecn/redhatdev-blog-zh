# 什么是红帽万能底座形象？

> 原文：<https://developers.redhat.com/blog/2019/10/09/what-is-red-hat-universal-base-image>

早在 5 月，我们[发布了 Red Hat Universal Base Image(UBI)](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)，目标是开发人员为云构建容器化的应用程序。从那时起，我们已经发布了一个[广泛的 FAQ](https://developers.redhat.com/articles/ubi-faq/#resources) ，涵盖了从 UBI 多久更新一次到最终用户许可协议(EULA)如何允许你重新发布基于 UBI 的应用程序等话题。这些都是很重要的基本话题，但是人们似乎仍然有很多关于 UBI 是什么和不是什么的问题。

如果你是一名开发人员，你正试图弄清楚 UBI 是否适合你，首先解释它不适合你可能会更容易。红帽万能底座的形象是:

1.  **不免费的红帽企业版 Linux (RHEL)。**你不应该首先在 RHEL 的基础映像上构建你的应用程序，然后试着把它转移到 UBI 并期望它能工作。这是一种反模式。
2.  不是软呢帽的替代品。 UBI 不是一个推动新操作系统开发的地方，也不是一个为 RHEL 开发新软件包的平台。
3.  **不能代替 CentOS。** UBI 是 RHEL。这不是下游重建。当它运行在 RHEL 或 CoreOS(在[红帽 OpenShift](https://developers.redhat.com/openshift/) )上时，它被支持为 RHEL。

现在，我们来谈谈什么是 UBI:

1.  **用于制造和交付认证集装箱和操作人员的车辆。**
2.  **基于用例。** UBI 面向高级应用程序开发人员，使用类似的语言进行编程。NET、Golang、Node.js、Perl、PHP、Python 和 Ruby。
3.  **高质量。**拥有与红帽企业版 Linux 相同的质量保证。UBI 是释放和修补 RHEL 的时间表与 RHEL。它由与 RHEL 相同的性能、安全性和质量团队进行测试。
4.  **根据不同于传统 RHEL 的最终用户许可协议(EULA)进行再分发。**传统的 RHEL 包装(rpm)和容器图像受到 RHEL EULA 的限制。

## 用于制造和交付认证集装箱和操作器的车辆

UBI 使 ISV 可以轻松地为 RHEL 构建和交付认证的容器映像，为 OpenShift 构建和交付认证的[操作员](https://www.openshift.com/learn/topics/operators)。对于希望同时获得软件供应商和容器平台供应商支持的客户来说，这是一个关键的价值主张。

UBI 使开发人员能够在他们的笔记本电脑、台式机和 CI/CD 系统上使用高质量的容器映像，即使他们不运行 RHEL 或 OpenShift。这也允许 ISV 和 Red Hat 合作伙伴在 UBI 上重新分发他们的应用程序。当应用程序登陆到一个支持的环境(RHEL 或 OpenShift)，红帽将支持它。如果您是 Red Hat ISV 合作伙伴，并且正在寻找向我们的共同客户提供更高质量支持的方法，请查看 [Partner Connect 计划](https://connect.redhat.com/)；它是免费的。

## 基于用例:UBI 适用于容器化的云原生应用

如前所述，UBI 不应该被认为是免费的红帽企业 Linux。UBI 针对特定的使用案例，而不是 RHEL 可以解决的各种各样的使用案例。今天，UBI 用例包括开发人员希望开发基于流行语言的容器化的云原生应用程序，如。NET、Golang、Node.js、Perl、PHP、Python 和 Ruby。

我们正在认真考虑扩展到其他几个用例，包括 C/C++开发、容器映像构建器和 RPM 包构建器，但目前，它仅设计用于云原生应用程序开发，不应被视为免费的 RHEL 或 CentOS 的替代品。

## 高质量:RHEL 的安全和操作优势

如果您想更快地投入生产，请构建高质量的基础映像。在 UBI 上构建就像在 RHEL 上构建一样，因此您将花费更少的时间来说服安全团队和运营团队您从互联网上选择的随机映像是安全的。参见:[我应该在 RHEL 和 OpenShift](https://developers.redhat.com/blog/2016/05/18/3-reasons-i-should-build-my-containerized-applications-on-rhel-and-openshift/) 上构建容器化应用程序的 3 个理由。

由于 UBI 是 Red Hat Enterprise Linux，它遵循与 RHEL 相同的企业内容可用性时间表。这意味着你可以在 UBI 上构建你的容器化应用，用基于 CI/CD 的构建来管理它，并且在和 RHEL 相同的生命周期内继续接收 UBI 中的包的更新。换句话说，您可以专注于您的应用程序，而不是修复底层容器映像中的包。更少的 CI/CD 测试错误意味着有更多的时间关注新项目。

## 可再分配

Red Hat Enterprise Linux 一直有一个最终用户许可协议，防止签署该协议的用户重新分发 RHEL。这延伸到了 RHEL 的每分钟转数和集装箱图像。这个 EULA 对于管理红帽和它的顾客之间的关系是必要的。这确保了客户可以订阅每台活动的 RHEL 服务器，从而使 Red Hat support 能够在客户需要时快速为他们提供帮助。这在传统的服务器环境中运行良好，公司通常通过安装的服务器来衡量支持。

UBI 的最终用户许可协议(EULA)与 RHEL 不同。这意味着用户可以把它拆下来，在上面构建一个应用程序，开源的或者私有的，并且在任何地方以任何方式重新发布这个应用程序。当 UBI 图像登陆 RHEL 或 OpenShift 时，它就像 RHEL 一样得到支持。UBI 的授权来自主机，而不是容器映像。这在基于云的环境和开发人员环境中工作得很好，在这些环境中，容器并不总是运行在受支持的容器主机上(例如 CI/CD 服务等)。

## 查看红帽通用基础图片

如果您正在构建新的应用程序，您应该查看 Red Hat Universal Base Image。它可以被认为是基于用例的、可再发行的 RHEL 版本。这为您提供了 RHEL 基础映像的所有优势，而没有 EULA 对再分发的限制。

查看 Red Hat Container 目录的 [UBI 产品页面](https://access.redhat.com/containers/#/product/5c180b28bed8bd75a2c29a63)上所有基于 UBI 的图片。从可信来源下载可信映像。

*Last updated: July 1, 2020*