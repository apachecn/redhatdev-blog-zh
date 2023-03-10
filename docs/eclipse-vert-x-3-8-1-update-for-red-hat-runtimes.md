# 针对 Red Hat 运行时的 Eclipse Vert.x 3.8.1 更新

> 原文：<https://developers.redhat.com/blog/2019/10/21/eclipse-vert-x-3-8-1-update-for-red-hat-runtimes>

Red Hat Runtimes 的最新更新已经发布，现在支持 Eclipse Vert.x 3.8.1。

Red Hat Runtimes 为应用程序开发人员提供了各种应用程序运行时，使他们能够在 [Red Hat OpenShift 容器平台](https://developers.redhat.com/openshift/)上运行。

## 有什么新鲜事？

在此次更新中，一些新增内容和更新包括:

*   **Vert.x Web API 合同**扩展 Vert.x Web 以支持 [OpenAPI 3](https://www.openapis.org/) ，为您带来一个简单的界面来构建您的 Vert.x 路由器并添加安全和验证处理程序。现在，您可以用 OpenAPI 定义您的 API，并直接在 Vert.x 代码中使用它们。这不仅对于 API 优先驱动的开发来说是一个强大的特性，而且对于利用 Vert.x 非阻塞 API 来构建健壮和高性能的后端来说也是如此。
*   绿色. x SQL 客户端
    *   该客户端是传说中的*反应式 PostgreSQL 客户端*的发展，并提供
        *   反应式 PostgreSQL 客户端
        *   反应式 MySQL 客户端
    *   这些实现提供了对 PostgreSQL 和 MySQL 的高性能无阻塞访问。
*   绿色. x 邮件客户端
*   客户端支持一些额外的身份验证方法，如 DIGEST-MD5、TLS 和 SSL，并且是完全异步的。客户端还允许连接池保持连接的开放和重用。
*   **Openshift 容器平台**:此外，这一最新版本在 OCP 4.1 上得到验证，对 OCP 4.2 的支持将在它可用后很快包括进来。

## 证明文件

有关更多详细信息，请查看支持的配置和组件详细信息

*   Red Hat 运行时 Eclipse Vert.x 支持的[配置](https://access.redhat.com/articles/3985941)
*   Red Hat 运行时 Eclipse Vert.x 3.8.1 [组件详细信息](https://access.redhat.com/articles/4486451)

如果您是 Eclipse Vert.x 的新手，并且想要了解更多，请访问我们的实时学习门户网站，获取有指导的[教程](https://learn.openshift.com/middleware/courses/middleware-vertx/)或[文档](https://docs.redhat.com/)以获取详细信息。

## 开发者互动学习场景

这些[自定进度的场景](https://learn.openshift.com/middleware/courses/middleware-vertx/)为您提供了一个预配置的 Red Hat OpenShift 实例，无需任何下载或配置即可从您的浏览器访问。使用它来学习和试验 Vert.x 或了解 Red Hat 运行时中的其他技术，并了解它如何帮助解决现实世界中的问题。

## 获得对 Eclipse Vert.x 的支持

Red Hat 客户可以通过订阅 [Red Hat Runtimes](https://www.redhat.com/en/products/runtimes) 来获得对 Eclipse Vert.x 的支持。请联系您当地的 Red Hat 代表或 [Red Hat 销售部](https://www.redhat.com/en/about/contact/sales)，了解如何享受 Red Hat 及其全球合作伙伴网络提供的世界级支持。

展望未来，根据 [Red Hat 产品更新和支持生命周期](https://access.redhat.com/support/policy/updates/jboss_notes/)，客户可以期待对 Eclipse Vert.x 和其他运行时的支持。

### Red Hat 运行时和 Eclipse Vert.x 支持背后的人

这个产品是由 Red Hat 的运行时产品和工程团队与 [Eclipse Vert.x](https://vertx.io/) 上游社区一起开发的，涉及许多小时的开发、测试、文档编写、测试，并与更广泛的 Red Hat 客户、合作伙伴和 Vert.x 开发人员社区合作，以整合大大小小的贡献。我们很高兴你选择使用它，并希望它达到或超过你的期望！

### 关于 Red Hat 运行时的更多信息

[Red Hat runtime](https://www.redhat.com/en/products/runtimes)为有云原生应用开发需求的开发人员、架构师和 IT 领导提供了一套全面的框架、运行时和编程语言。它旨在加速业务解决方案的开发和交付。以下是使用 Red Hat 运行时创建微服务的不同运行时和使能器。更多详情请点击此[链接](https://www.redhat.com/en/products/runtimes)。

![](img/100bb87f9f8e4f00b81bc72cca97abe8.png)

*Last updated: July 1, 2020*