# 为什么我一直需要从命令行登录？

> 原文：<https://developers.redhat.com/openshift/keep-logining-in>

当您以用户身份从命令行登录时，会创建一个新会话。与会话对应的令牌将缓存在您的本地帐户中。当您运行`oc`命令行工具时，该令牌将随对 OpenShift 的每个请求一起提供。

在典型的 OpenShift 环境中，这些会话令牌将在一天后过期。如果 OpenShift 环境的管理员覆盖了所使用的默认值，则到期时间会有所不同。

一旦会话令牌过期，要使用`oc`命令行工具，您需要再次登录。

因为登录会话将在没有通知的情况下过期，所以您应该避免将会话令牌用于普通用户帐户和使用 OpenShift REST API 的脚本。对于脚本访问，最好创建一个服务帐户。服务帐户有一个关联的访问令牌，您可以使用它，它不会过期。

### 相关文章

如何为脚本访问创建服务帐户？

[在 OpenShift 上开发应用](https://developers.redhat.com/openshift)

**红帽 OpenShift 集装箱平台**

【OpenShift 和 Kubernetes 有什么区别？

[有哪些关于 OpenShift 的书籍？](https://developers.redhat.com/openshift/openshift-books/)

在哪里可以试用 OpenShift，看看它是什么样的？

[如何在自己的电脑上运行 OpenShift 进行开发？](https://developers.redhat.com/openshift/local-openshift/)

[有哪些使用 OpenShift 的托管服务？](https://developers.redhat.com/openshift/hosting-openshift/)

*Last updated: November 19, 2020*