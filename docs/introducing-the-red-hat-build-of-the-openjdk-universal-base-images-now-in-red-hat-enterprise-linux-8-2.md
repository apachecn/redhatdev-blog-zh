# 介绍 OpenJDK 通用基础映像的 Red Hat 版本——现在在 Red Hat Enterprise Linux 8.2 中

> 原文：<https://developers.redhat.com/blog/2020/06/25/introducing-the-red-hat-build-of-the-openjdk-universal-base-images-now-in-red-hat-enterprise-linux-8-2>

随着最近发布的 [Red Hat Enterprise Linux 8.2](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/8.2_release_notes/index) ，我们还添加了第一个[Red Hat build of open JDK Universal Base Images](https://developers.redhat.com/products/openjdk/)。OpenJDK 8 和 OpenJDK 11 的这些通用可用性(GA)映像为那些希望开发以安全、稳定和经过测试的方式在容器内运行的 Java 应用程序的人设置了一个新的基准。

在本文中，我们将介绍新的 OpenJDK 通用基础映像，并解释它们对 Java 开发人员的好处。在此之前，让我们快速回顾一下我们对 UBIs 的总体了解。

## 关于通用基础图像

[红帽通用基础图像(ubi)是](https://developers.redhat.com/articles/ubi-faq/#resources):

> 符合 OCI 标准的基于容器的操作系统映像，带有可自由再分发的补充运行时语言和软件包。像以前的基础映像一样，它们是从[Red Hat Enterprise Linux(RHEL)](https://developers.redhat.com/topics/linux/)的部分构建的。 [UBI 映像](https://developers.redhat.com/blog/category/ubi/)可以从 Red Hat 容器目录中获得，并且可以在任何地方构建和部署。

换句话说，UBIs 帮助应用程序开发人员到达安全、稳定和可移植的容器世界。这些图像可以使用知名的工具访问，如 Podman/Buildah 和 Docker。Red Hat Universal Base Images 还允许用户在企业级位的基础上构建和分发他们自己的应用程序，这些应用程序在 [Red Hat OpenShift](https://developers.redhat.com/openshift) 和 Red Hat Enterprise Linux 上是受支持的。

## OpenJDK 的红帽版本

OpenJDK 的[红帽构建基于上游 OpenJDK 8u 和 OpenJDK 11u 社区主导的项目。Red Hat 为这两个项目提供了重要的贡献者，并为 Red Hat 版本增加了额外的未来特性。](https://developers.redhat.com/products/openjdk/overview)

该版本包括:

*   Shenandoah 超低暂停时间垃圾收集器。
*   几个安装选项，包括 RPM，MSI 安装程序，还有一个 ZIP 版本。
*   Java Web Start 支持(仅限 Windows 对于 Linux，请使用红帽企业版 Linux RPMs)。

OpenJDK 的 Red Hat 版本至少每季度更新一次，以增强安全性和其他 bug 修复功能。Red Hat 为 OpenJDK 的主要版本提供定期支持和维护。在这种情况下，我们为版本 8 和版本 11 分别提供长期支持(LTS)版本，直到 2026 年 6 月和 2024 年 10 月。(另见: [OpenJDK 生命周期和支持政策](https://access.redhat.com/articles/1299013)。)

有关更多信息，请查看红帽生态系统目录中的新图片:

*   [OpenJDK 8 UBI8 图片](https://access.redhat.com/containers/?tab=package-list#/registry.access.redhat.com/ubi8/openjdk-8/images/1.3-2)
*   [OpenJDK 11 UBI8 图片](https://access.redhat.com/containers/?tab=package-list#/registry.access.redhat.com/ubi8/openjdk-8/images/1.3-2)

## 开始使用 OpenJDK UBI 图像

OpenJDK UBI 映像有默认的启动脚本，可以自动检测应用程序 jar 并启动 Java。可以使用环境变量自定义脚本的行为。查看容器内的`/help.md`，了解更多信息。

同时，这里有一个 Dockerfile 的简短示例，它将一个名为`testubi.jar`的应用程序添加到 OpenJDK 11 UBI8 映像中:

```
FROM registry.access.redhat.com/ubi8/openjdk-11

COPY target/testubi.jar /deployments/testubi.jar
```

## 扩展通用基础映像 EULA

当我们在 2019 年 5 月推出 Universal Base Images 时，我们包括了一个最终用户许可协议(EULA)，该协议使红帽合作伙伴能够自由使用和再分发大量 RHEL 软件包，这些软件包可以部署在红帽和非红帽平台上。根据该协议，开发者可以构建安全、可靠、可移植的基于容器的软件，并将其部署到任何地方。反馈非常积极，对此我们表示感谢！我们还了解到您需要更多，因此我们正在为客户扩展套餐。

Red Hat Partner Connect 计划中的应用程序开发人员现在可以从全套 Red Hat Enterprise Linux (RHEL)用户空间包(非内核)构建容器应用程序，并通过他们选择的容器注册表重新分发它们。与 UBI-only 相比，这一扩展几乎将可用的软件包数量增加了两倍。

**参见** : [红帽简化了红帽企业 Linux 包的容器开发和再分发](https://developers.redhat.com/blog/2020/02/26/red-hat-simplifies-container-dev-and-redistribution-rhel-packages/)。

## 结论

如果您希望在容器中开发 Java 应用程序——并且您还希望能够在安全性、可靠性和稳定性方面信任底层基础——那么 OpenJDK UBI8 映像可能是您正在寻找的解决方案。我们希望您尝试新的 OpenJDK 通用基础映像，在混合云中构建容器化的 Java 应用程序。

在接下来的几周，我们将继续更新，更详细地介绍如何使用 OpenJDK UBI，包括更多如何处理 OpenJDK UBI8 容器图像的示例。敬请期待！