# 用 Quarkus 1.7 的红帽版本引领 Java 的未来

> 原文：<https://developers.redhat.com/blog/2020/11/10/leading-the-future-of-java-with-the-red-hat-build-of-quarkus-1-7>

![](img/cabb6bdef3aceef7d08d66c8c142b8fd.png)

Quarkus 的最新支持版本 [Red Hat build 继续为](https://developers.redhat.com/products/quarkus/getting-started) [Kubernetes](https://developers.redhat.com/topics/kubernetes) 本地和[无服务器](https://www.redhat.com/en/topics/cloud-native-apps/what-is-serverless)应用驱动 [Java 开发](https://developers.redhat.com/topics/enterprise-java)的未来。本文介绍了一些技术，这些技术使得使用 Quarkus 1.7 的 Red Hat 版本为基于[容器的](https://developers.redhat.com/topics/containers)和无服务器环境创建快速、轻量级的 Java 应用程序变得前所未有的容易。

## 本机代码编译

使用 Quarkus 的 Red Hat 版本的开发人员现在可以根据应用程序的需要选择部署本地编译的代码还是基于 JVM 的代码。原生编译的 Quarkus 应用程序速度极快且内存高效，这使得 Quarkus 成为无服务器和高密度云部署的绝佳选择。Quarkus 1.7 对原生可执行文件的支持由基于 OpenJDK 11 的 GraalVM 下游发行版 [Mandrel](https://developers.redhat.com/blog/2020/06/05/mandrel-a-community-distribution-of-graalvm-for-the-red-hat-build-of-quarkus/) 提供。

**注**:参见[最近发布的 *Quarkus IDC 实验室验证报告*的摘要](https://www.redhat.com/en/blog/key-findings-idc-red-hat-quarkus-lab-validation)，了解更多关于 Quarkus 与传统 Java 框架相比的表现。

## 附加特性和功能

Quarkus 1.7 的 Red Hat 版本包括以下附加特性和功能:

*   [红帽 OpenShift 4.5](https://developers.redhat.com/blog/2020/07/16/improved-navigation-in-the-openshift-4-5-developer-perspective#) 认证[支持的配置](https://access.redhat.com/articles/4966181)。
*   支持[红帽 OpenShift 无服务器](https://developers.redhat.com/topics/serverless-architecture) ( [主动发球](https://github.com/knative/serving))。
*   与红帽数据网格 8 ( [Infinispan 客户端](https://quarkus.io/guides/infinispan-client))和红帽单点登录技术( [Keycloak](https://quarkus.io/guides/security-keycloak-authorization) )的集成。
*   针对[缓存](https://quarkus.io/guides/spring-cache)、配置和调度的 Spring 兼容性增强。
*   用于远程过程调用的 [gRPC](https://quarkus.io/guides/grpc-getting-started) 扩展。
*   支持[兵变](https://quarkus.io/guides/getting-started-reactive#mutiny)反应框架。

### gRPC 扩展和远程开发

Quarkus 的创始原则之一是[带给 Java 开发人员快乐](https://quarkus.io/vision/developer-joy)。Quarkus 通过为开发人员提供工具和功能，如实时编码、统一配置、IDE 插件等，实现了这一承诺。Quarkus 的 Red Hat build 还支持一个[庞大的扩展生态系统](https://code.quarkus.io/)，用于轻松配置、集成和编译其他框架和技术。一种这样的技术是 [gRPC 扩展](https://quarkus.io/blog/quarkus-grpc/)，它允许开发者公开和使用带有传输层安全性(TLS)加密和认证的远程过程调用。

Quarkus 还提供了一个[远程开发模式](https://quarkus.io/guides/maven-tooling#remote-development-mode)，让开发人员在 OpenShift 等容器环境中运行 Quarkus。在远程开发模式下，对本地文件的更改立即可见。

### OpenShift 和无服务器部署

其较小的内存占用和快速的启动时间使 Quarkus 成为无服务器应用程序的理想运行时。Quarkus 的 Red Hat 版本针对容器和 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview)进行了优化。因此，开发人员可以轻松部署 Kubernetes-native 和无服务器应用程序，而无需担心底层基础设施。 [Quarkus OpenShift 扩展](https://quarkus.io/guides/deploying-to-openshift)允许您在单个构建命令中使用 Apache Maven 或源到映像(S2I)方法在 OpenShift 上部署 Quarkus 应用程序和 Kubernetes 资源。Quarkus OpenShift 扩展还支持将 Quarkus 应用程序部署到安装了 Knative Serving 的 OpenShift。Knative Serving 根据负载大小调整应用服务。

**注意**:参见 [Red Hat Build of Quarkus 1.7](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.7/) 文档，了解更多关于[在 OpenShift](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.7/html-single/deploying_your_quarkus_applications_on_red_hat_openshift_container_platform/index) 和[上部署 Quarkus 应用作为 OpenShift 无服务器服务](https://cloud.redhat.com/learn/topics/serverless)的信息。

### 开发人员的更多工具

除了与 OpenShift 的优化和集成，Quarkus 的 Red Hat build 还与 Red Hat Data Grid 8 和 Red Hat 的单点登录技术紧密集成。

Red Hat Data Grid 8 是一款基于 Infinispan 的内存分布式 NoSQL 数据存储解决方案。使用 Quarkus [Infinispan 扩展](https://quarkus.io/guides/infinispan-client)，开发人员可以连接到运行在应用程序进程之外的数据网格服务器，并创建本地可执行文件。参见红帽 OpenShift 上的 *[安全连接 Quarkus 和红帽数据网格，了解更多关于该技术的信息。](https://developers.redhat.com/blog/2020/10/15/securely-connect-quarkus-and-red-hat-data-grid-on-red-hat-openshift/)*

Red Hat 的单点登录技术为保护 web 应用程序提供了支持。 [Keycloak 扩展](https://quarkus.io/guides/security)提供了架构、认证和授权机制以及其他工具，用于为您的应用程序创建生产质量的安全性。参见 DevNation Tech Talk， *[使用 Keycloak](https://developers.redhat.com/videos/youtube/JvPBWPDQ940)* 轻松保护您的云原生微服务，了解有关使用 Red Hat 单点登录技术保护 Quarkus 微服务的更多信息。

## 夸库斯的下一步是什么？

Quarkus 社区正在快速创新和发布更新。我们将继续效仿这一创新，支持 Java 开发人员使用 Quarkus 创建云原生应用。Quarkus 的 Red Hat build 的未来版本将增加新的特性和功能，以提高开发人员的工作效率。我们也将继续寻找方法来支持开发者在 OpenShift 之外创建无服务器应用。

## 开始使用 Quarkus 1.7 的 Red Hat 版本

对于对入门感兴趣的开发者来说， [Quarkus 初始化器](http://code.quarkus.redhat.com/)是引导你的 Quarkus 应用程序和发现它的扩展生态系统的强大方法。

*Last updated: November 3, 2022*