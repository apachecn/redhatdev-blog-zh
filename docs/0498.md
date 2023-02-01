# 针对 Red Hat 运行时的 Node.js 更新改进了对本机模块、诊断报告等的支持

> 原文：<https://developers.redhat.com/blog/2020/04/09/node-js-update-for-red-hat-runtimes-brings-improved-support-for-native-modules-diagnostic-reporting-and-more>

在像 [Red Hat OpenShift](https://www.openshift.com/) 这样的 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 发行版上，或者在 Red Hat Enterprise Linux (RHEL)上，或者通过使用我们的[通用基础映像](https://developers.redhat.com/blog/category/ubi/)开发应用程序，使用 Red Hat 的 Node.js 构建更加容易。Red Hat 运行时的最新更新现在包括 Node.js 12.4.1，它为 LTS 版本提供了支持的运行时。与 Node.js 10 相比，Node.js 的这个新的 Red Hat 版本以及 Red Hat Enterprise Linux 8.1 的发布提供了许多新的功能和增强。

本文主要关注这些新特性和增强功能。

## 新增和变更的功能

通过最新的 RHEL 8.1 和 Node.js 12 的 Red Hat 版本，我们现在提供了许多超越版本 10 的新功能和增强功能。值得注意的变化包括:

*   将 V8 引擎升级到 7.4 版。
*   添加了一个新的默认 HTTP 解析器，`llhttp`(不再是实验性的)。
*   集成了生成堆转储的功能。
*   添加了对 ECMAScript 2015 (ES6)模块的支持。
*   改进了对本机模块的支持。
*   移除了工作线程必须有标志的要求。
*   增加了新的实验诊断报告功能。
*   改进的性能。

有关 Node.js 12.14.1 的详细变更，请参见[上游版本说明](https://nodejs.org/en/blog/release/v12.14.1/)和[上游文档](https://nodejs.org/dist/latest-v12.x/docs/api/)。

## 在 OpenShift 上部署新版本

Nodeshift 是一个固执己见的命令行应用程序和可编程 API，它简化了 NodeJS 应用程序到 OpenShift 的部署。为了帮助这个过程，Red Hat 为 Node.js 创建并维护[源到图像(S2I)容器图像。博客](https://cloud.docker.com/u/nodeshift/repository/docker/nodeshift/centos7-s2i-nodejs/tags)*[使用 Node.js 12 on Red Hat OpenShift today](https://developers.redhat.com/blog/2019/04/29/use-node-js-12-on-red-hat-openshift-today/)*解释了如何使用 Nodeshift 将 node . js 项目部署到 open shift。

## 证明文件

Runtimes 团队不断增加和改进 Red Hat 的 Node.js build 的官方文档。这项工作包括对[发行说明](https://access.redhat.com/documentation/en-us/red_hat_build_of_node.js/12/html-single/release_notes_for_node.js_12/index)和 [Node.js 运行时指南](https://access.redhat.com/documentation/en-us/red_hat_build_of_node.js/12/html-single/node.js_runtime_guide/index)的更新。

## 开发者互动学习场景

这些自定进度的场景(如图 1 所示)为您提供了一个预配置的 OpenShift 实例，无需任何下载或配置即可从浏览器访问该实例。使用该工具[试验 Node.js](https://learn.openshift.com/middleware/rhoar-getting-started-nodejs/) 或了解运行时内的其他技术，并查看 OpenShift 上的 Node.js 如何帮助解决现实世界的问题。

[![](img/31be319763bf9ff78536dcb14bf161a7.png "katacoda-node")](/sites/default/files/blog/2018/03/katacoda-node.png)

Figure 1: The Node.js interactive learning scenario.

## 结论

我们的目标一直是提供上游 Node.js 核心项目的快速发布。例如，这样做可以让我们提供应用和工具，让开发人员快速启动和运行，Node.js 容器映像，以及与 Red Hat 的云原生堆栈的其他组件的集成。如果您需要，Red Hat 可以通过 Red Hat OpenShift、Red Hat Enterprise Linux 和 [Universal Base Images](https://access.redhat.com/solutions/4309231) 为受支持的配置提供生产和开发支持。

## 更多资源

查看以下资源:

*   文章:[用 Node.js 在 RHEL 上的容器中开发](https://developers.redhat.com/blog/2019/09/13/develop-with-node-js-in-a-container-on-red-hat-enterprise-linux/)
*   背景:[红帽加入 Node.js 基金会](https://developers.redhat.com/blog/2015/10/07/red-hat-joins-node-js-foundation/)
*   下载:[node . js 的 Red Hat 构建的容器图像](https://catalog.redhat.com/software/containers/search?q=node.js&p=1)
*   如何:[用断路器保护 Node.js REST 客户端](https://lanceball.com/words/2017/01/05/protect-your-node-js-rest-clients-with-circuit-breakers)
*   Node.js 12: [今天在红帽 OpenShift 上使用 node . js 12](https://developers.redhat.com/blog/2019/04/29/use-node-js-12-on-red-hat-openshift-today/)
*   演示:[node . js on open shift for Your Enterprise](http://lanceball.com/riviera-dev-2017/)
*   产品页面:[红帽旗下](https://access.redhat.com/products/nodejs/) [Node.js](https://access.redhat.com/products/nodejs/) [build](https://access.redhat.com/products/nodejs/)

*Last updated: June 29, 2020*