# Red Hat Runtimes 将 Vert.x 和 Dekorate 引入 Spring Boot 2.2.6

> 原文：<https://developers.redhat.com/blog/2020/06/17/red-hat-runtimes-brings-vert-x-and-dekorate-to-spring-boot-2-2-6>

对[红帽运行时](https://developers.redhat.com/middleware/)的最新更新支持 [Spring Boot 2.2.6](https://access.redhat.com/documentation/en-us/red_hat_support_for_spring_boot/2.2/html/release_notes_for_spring_boot_2.2/) ，以及[德考特项目](https://github.com/dekorateio/dekorate)和[弹簧无功](https://spring.io/reactive)。总的来说，这些技术对于开发人员在 [Red Hat OpenShift 容器平台](https://developers.redhat.com/openshift/)上构建基于 Spring 的应用程序是一种推动。在本文中，我将介绍此次更新的亮点。

**关于 Red Hat 运行时** : [Red Hat 运行时](https://developers.redhat.com/middleware/)为有云原生应用开发需求的开发者、架构师和 IT 领导提供了一套全面的框架、运行时和编程语言。开发人员使用 Red Hat 运行时来访问 [Red Hat OpenShift 容器平台](https://developers.redhat.com/openshift/)上的各种应用程序运行时。

## 装饰

Dekorate 是一个广泛的框架，允许开发人员抽象编辑的细节，避免编写 XML、YAML 和 JSON 文件的冗长工作。相反，de Korea 只是在编译时生成这些清单[。](https://developers.redhat.com/blog/2019/08/15/how-to-use-dekorate-to-create-kubernetes-manifests/)

## 弹簧反作用

最新发布的 Red Hat Runtimes 为 OpenShift 和独立的 Red Hat Enterprise Linux (RHEL)部署带来了 [Project Reactor 和 Spring WebFlux](https://developers.redhat.com/blog/2019/08/30/extending-support-for-spring-boot-2-1-6-and-spring-reactive/) 的优势。除了 Spring Boot 版本的更新之外，这个版本还包括一组用于 Spring Boot 运行时的 Eclipse Vert.x 扩展。Vert.x 扩展中有一个异步 I/O API，用于被动处理应用服务之间的网络通信。这些增加扩展了 Spring WebFlux 的反应能力，同时保留了 Spring Boot 的抽象和快速原型制作能力。

## 红帽支持 Spring Boot

Red Hat Runtimes 团队不断更新和改进在 OpenShift 和 RHEL 上构建 Spring Boot 应用程序的文档。请参见 [Spring Boot 2.2.6 发行说明](https://access.redhat.com/documentation/en-us/red_hat_support_for_spring_boot/2.2/html/release_notes_for_spring_boot_2.2/)和 [Spring Boot 运行时指南](https://access.redhat.com/documentation/en-us/red_hat_support_for_spring_boot/2.2/html/spring_boot_runtime_guide/)了解更多关于红帽支持 Spring Boot 的细节。

### 开发者互动学习场景

Red Hat 提供了多种[自定进度的场景](https://learn.openshift.com/middleware/)来帮助您学习如何使用 Red Hat 运行时解决现实世界中的问题。每个场景都提供了一个预配置的 Red Hat OpenShift 实例，您可以从浏览器访问它，无需任何下载或配置。您可以使用 OpenShift 实例来[试验 Spring Boot](https://learn.openshift.com/middleware/courses/middleware-spring-boot/) 或者了解其他运行时和技术。图 1 显示了各种可用的场景。

[![A screenshot showing nine interactive learning modules on the Red Hat Interactive Courses homepage.](img/305117d4bdf78718224202caa46c9595.png "Screen Shot 2019-08-14 at 3.54.45 PM")](/sites/default/files/blog/2019/08/Screen-Shot-2019-08-14-at-3.54.45-PM.png)

Figure 1\. Self-paced interactive scenarios from Red Hat.

## 红帽支持 Spring Boot

红帽客户可以通过订阅[红帽运行时](https://developers.redhat.com/products/rhoar/overview/)来获得对 [Spring Boot](https://developers.redhat.com/topics/spring-boot/) 的支持。请联系您当地的 Red Hat 代表或 [Red Hat 销售部](https://www.redhat.com/en/about/contact/sales)，了解如何享受 Red Hat 和我们全球合作伙伴网络的世界级支持。

**注意**:展望未来，客户可以期待 Spring Boot 和其他运行时的更新遵循 Red Hat [产品更新和支持生命周期](https://access.redhat.com/support/policy/updates/jboss_notes/)。

### Spring Boot 在红帽时代的下一步是什么

Red Hat Runtimes Spring Boot 团队不断从客户和更广泛的开源开发者社区获得反馈，同时跟踪上游 Spring Boot 版本的变化。该团队将基于这些因素继续更新 Red Hat 对 Spring Boot 的运行时支持，并考虑来自 Red Hat 和更大的 [Java](https://developers.redhat.com/topics/enterprise-java/) 和 Spring 社区的额外模块。

## 结论

最新的 Red Hat 运行时更新是由 Red Hat 的应用程序运行时产品和工程团队与 [Snowdrop](https://snowdrop.me/) 上游社区一起开发的。这些团队合作了许多小时的开发、测试、编写文档、更多的测试，并整合了来自更广泛的客户、合作伙伴和 Spring 开发人员的 Red Hat 社区的贡献。我们希望它满足或超过您的期望！

*Last updated: June 25, 2020*