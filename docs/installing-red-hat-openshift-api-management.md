# 安装 Red Hat OpenShift API 管理

> 原文：<https://developers.redhat.com/blog/2021/02/23/installing-red-hat-openshift-api-management>

[Red Hat OpenShift API 管理](https://developers.redhat.com/products/rhoam/overview)是一个新的托管应用程序服务，作为 [Red Hat OpenShift 专用](https://www.openshift.com/products/dedicated/)的附加服务。该服务在开发和交付采用 [API 优先方法](https://developers.redhat.com/blog/2019/12/02/apis-as-a-product-get-the-value-out-of-your-apis/)的[微服务](https://developers.redhat.com/topics/microservices)应用时，为开发者提供了简化的体验。

OpenShift API 管理建立在 [3scale API 管理](https://developers.redhat.com/products/3scale/overview)的成功之上，旨在让开发人员在开发应用程序时轻松共享、保护、重用和发现 API。本文向您展示了如何将 OpenShift API Management 作为附加组件安装到您的 OpenShift 专用集群中。正如您将看到的，安装、配置、管理、启动和运行 OpenShift API 管理只需不到 10 分钟的时间。查看文章末尾的视频演示。

## 第一步:让 OpenShift 专用

您将需要一个 OpenShift 专用订阅来提供一个集群以运行 Red Hat OpenShift API 管理。

从配置集群开始:

1.  在[cloud.redhat.com/openshift](https://cloud.redhat.com)上登录您的账户。
2.  打开 Red Hat OpenShift 集群管理器。
3.  通过选择 **Red Hat OpenShift Dedicated** 创建集群。
4.  选择您的云提供商。
5.  填写所有集群配置详细信息:集群名称、区域、可用性区域、计算机节点、节点实例类型、CIDR 范围(这些是可选的，但以后不能修改)等等。
6.  提交您的请求，并等待集群被调配。

## 步骤 2:选择并配置身份提供者

一旦您的集群准备就绪，您就可以配置您首选的[身份提供者](https://access.redhat.com/documentation/en-us/openshift_dedicated/4/html/authentication/configuring-identity-providers) (IDP)

首先，从下拉菜单中选择并配置您的身份提供商(IDP)。选项包括 GitHub、Google、OpenID、LDAP、GitLab 和其他支持 OpenID 流的提供商。

配置 IDP 后，您需要向 **dedicated-admins** 组添加至少一个用户。在**集群管理用户**部分点击**添加用户**，然后输入新**专用管理用户**的用户名。

每个组织的访问级别不同。任何需要完全访问权限的用户都应该被定义为管理员。作为管理员，您可以使用 3scale API 管理和 OpenShift 基于角色的访问控制(RBAC)功能来限制其他用户的访问。

## 步骤 3:安装 OpenShift API 管理

安装 OpenShift API 管理软件的过程非常简单。从“群集详细信息”页面，您可以导航到“附加组件”选项卡，并找到可供您使用的服务。设置如下:

1.  转到集群详细信息页面上的**附加组件**选项卡。
2.  选择**红帽 OpenShift API 管理**，点击**安装**。
3.  提供将接收与服务相关的[警报和通知](https://access.redhat.com/documentation/en-us/red_hat_openshift_api_management/1/topic/c26ddfc9-e5bd-4685-a403-7f84697a6e8a)的帐户的电子邮件地址。
4.  再次点击**安装**，允许操作员安装和配置服务。

安装插件后，您可以通过位于 OpenShift 专用控制台右上角的应用程序启动菜单访问该服务。应用程序启动器提供对 OpenShift API 管理和 Red Hat 单点登录(SSO)技术服务 ui 的直接访问。

## 观看视频演示

请观看以下视频，了解 OpenShift API 管理的安装步骤:

[https://www.youtube.com/embed/sd2TlBm5KHs?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/sd2TlBm5KHs?autoplay=0&start=0&rel=0)

## 使用 OpenShift API 管理

这就对了。在您的 OpenShift 专用集群上安装并运行 OpenShift API 管理就像这里描述的步骤一样简单。现在你可以开始使用这项服务了。查看以下资源，开始行动:

*   熟悉 [Red Hat OpenShift API 管理](https://developers.redhat.com/products/rhoam/overview)。
*   使用 OpenShift API 管理[部署和管理您自己的 Quarkus API](https://www.youtube.com/watch?v=NzNgc0f75pc) 。
*   从 [OpenShift 开发者视角](https://docs.openshift.com/container-platform/4.2/web_console/odc-about-developer-perspective.html)跟随快速入门，了解更多关于 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 和您的集群的信息。

*Last updated: February 22, 2021*