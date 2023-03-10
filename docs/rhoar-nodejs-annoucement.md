# 宣布:Node.js 在 Red Hat OpenShift 应用程序运行时中正式发布

> 原文：<https://developers.redhat.com/blog/2018/03/12/rhoar-nodejs-annoucement>

![Node.js Foundation Logo](img/1a062f73a47a3ea754e636ceffa7c5f9.png)

## 摘要

今天，Red Hat 正在通过订阅 [Red Hat OpenShift 应用运行时](http://developers.redhat.com/rhoar) (RHOAR)向 Red Hat 客户提供 Node.js。RHOAR 为应用开发者提供了多种运行在 [OpenShift 容器平台](https://www.openshift.com/)上的应用运行时。

Node.js 基于 V8 JavaScript 引擎，允许您编写服务器端 JavaScript 应用程序。Node.js 加入了现有的一组受支持的运行时，并为开发人员提供了一个事件驱动的非阻塞 I/O 模型，使其轻量级和高效，非常适合跨分布式设备运行的数据密集型实时应用程序。

## 盒子里有什么？

此版本提供了 Node.js 核心运行时版本 8.9.4、npm 5.6.0 以及相关的任务和支持工具，以支持开发人员开始使用 Node.js 和启动项目。

请注意，RHOAR Node.js 基础映像允许您使用 npm 提供的任何社区 Node.js 模块为 OpenShift 开发 Node.js 应用程序。社区 [npm](https://npmjs.org/) 模块不受 Red Hat 支持。

## 启动 OpenShift

使用[developers.redhat.com/launch](https://developers.redhat.com/launch)您可以立即创建一个 Node.js 应用程序，并将其直接部署到 [OpenShift Online](http://openshift.com/) 或您自己的本地 OpenShift 集群。它提供了一种创建示例应用程序(称为 booster)的简单方法，以及一种在 OpenShift 上构建和部署这些 booster 的简单方法。

支持者可以展示开发人员如何使用 Node.js 来构建云原生应用和服务的基本构建块，如创建 RESTful APIs、实现健康检查、外部化配置或断路器等弹性功能。

## 从 Red Hat 容器目录访问 Node.js 图像

Node.js 运行时是通过 [Red Hat 容器目录](https://access.redhat.com/containers/)以包含 Node.js 8.9.4 的容器化 OpenShift [S2I 构建器映像](https://blog.openshift.com/create-s2i-builder-image/)的形式提供的。可以从命令行(使用 oc 命令)或从 OpenShift Dashboard GUI 界面获取。下面是一个命令，您可以使用它将映像提取到本地系统，以便与 OpenShift 一起使用:

```
oc import-image nodejs:8 --from=registry.access.redhat.com/rhoar-nodejs/nodejs-8 --confirm
```

然后，可以使用以下命令构建一个示例 Node.js 应用程序并将其部署到 Red Hat OpenShift:

```
oc new-app --name nodejs-example nodejs:8~https://github.com/openshift/nodejs-ex
oc expose svc/nodejs-example
```

使用这些发行版的 Red Hat 客户将能够获得最新的更新、安全建议，了解容器更新的时间和原因，并保持最新的可用标记图像。

## 证明文件

RHOAR 团队一直在不断增加和改进 Node.js 的官方文档。这包括对[发行说明](https://access.redhat.com/documentation/en-us/red_hat_openshift_application_runtimes/1/html/red_hat_openshift_application_runtimes_release_notes/)、[入门指南、](https://access.redhat.com/documentation/en-us/red_hat_openshift_application_runtimes/1/html/getting_started_with_red_hat_openshift_application_runtimes/)和新的 [Node.js 运行时指南](https://access.redhat.com/documentation/en-us/red_hat_openshift_application_runtimes/1/html-single/node.js_runtime_guide/)的更新。

## 开发者互动学习场景

这些[自定进度场景](https://learn.openshift.com/)为您提供了一个预配置的 OpenShift 实例，无需任何下载或配置即可从您的浏览器访问。用它来[试验 Node.js](https://learn.openshift.com/middleware/rhoar-getting-started-nodejs/) ，或者了解 RHOAR 中的其他技术，看看它如何帮助解决现实世界的问题。

[![Node.js Interactive Learning Scenario Screenshot](img/561fa3b870ba583994abf77bee7471f8.png)](https://learn.openshift.com/middleware/rhoar-getting-started-nodejs/)

## 获得支持

通过订阅 Red Hat OpenShift 应用程序运行时，Red Hat 客户可以获得对 Node.js 的支持。请联系您当地的 Red Hat 代表或 [Red Hat 销售人员](https://www.redhat.com/en/about/contact/sales)了解如何享受 Red Hat 及其全球合作伙伴网络提供的世界级支持。

展望未来，根据 [Red Hat 产品更新和支持生命周期](https://access.redhat.com/support/policy/updates/jboss_notes/)，客户可以期待对 Node.js 和其他 RHOAR 运行时的支持。

## 下一步是什么？

RHOAR 团队不断从客户和更广泛的开源开发者社区获得反馈，并跟踪上游 Node.js 版本。他们正在根据反馈对 RHOAR 运行时进行更新，并考虑支持来自 Red Hat 和非常大的 Node.js 生态系统的其他模块。

## 太棒了。

该版本由 Red Hat 的 RHOAR 工程团队制作，涉及许多小时的开发、测试、编写文档、测试以及与更广泛的 Red Hat 客户、合作伙伴和 Node.js 开发人员社区合作，以整合大大小小的贡献。我们很高兴你选择使用它，并希望它达到或超过你的期望！

## 更多资源

*   [Red Hat OpenShift 应用运行时开发者主页](http://developers.redhat.com/rhoar)
*   [红帽加入 Node.js 基金会](https://developers.redhat.com/blog/2015/10/07/red-hat-joins-node-js-foundation/)
*   [RHOAR Shootout - Node.js](https://lanceball.com/slides/rhoar-shootout/)
*   [node . js on open shift for Your Enterprise](http://lanceball.com/riviera-dev-2017/)
*   [用断路器保护 Node.js REST 客户端](https://lanceball.com/words/2017/01/05/protect-your-node-js-rest-clients-with-circuit-breakers)

*Last updated: September 3, 2019*