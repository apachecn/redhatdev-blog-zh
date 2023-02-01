# Open Liberty Java runtime 现已对 Red Hat Runtimes 订户开放

> 原文：<https://developers.redhat.com/blog/2019/11/14/open-liberty-java-runtime-now-available-to-red-hat-runtimes-subscribers>

Open Liberty 是一款轻量级、生产就绪的 [Java](https://developers.redhat.com/topics/enterprise-java/) 运行时，用于将微服务封装和部署到云，现在可以作为[红帽运行时](https://www.redhat.com/en/products/runtimes)订阅的一部分。如果你是 Red Hat Runtimes 的订阅者，你可以在 Open Liberty 上编写你的 [Eclipse MicroProfile](https://microprofile.io/) 和 [Jakarta EE](https://jakarta.ee/) 应用程序，然后在 Red Hat 和 IBM 的商业支持下，在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 上的容器中运行它们。

## 开发云原生 Java 微服务

Open Liberty 旨在提供流畅的开发者体验，启动时间[一秒](https://openliberty.io/blog/2019/10/30/faster-startup-open-liberty.html)，内存占用低，以及我们新的[开发模式](https://openliberty.io/blog/2019/10/22/liberty-dev-mode.html):

[![Tweet about Open Liberty Dev Mode.](img/a42697b24a12b664a669e4783347ab20.png)](https://twitter.com/javahippie/status/1187986394117001216)

Open Liberty 提供了 MicroProfile 3 和 [Jakarta EE 8](https://developers.redhat.com/blog/2019/09/12/jakarta-ee-8-the-new-era-of-java-ee-explained/) 的完整实现。 [MicroProfile](https://developers.redhat.com/videos/youtube/fbVYQENPa4s/) 是多个厂商(包括 Red Hat 和 IBM)和 Java 社区之间的一个[合作项目](https://microprofile.io/contributors/)，旨在为编写微服务优化企业 Java。Liberty 有一个为期四周的发布时间表，通常在规范发布后不久就会有最新的 MicroProfile 发布。

此外，Open Liberty 在常见的开发工具中也得到支持，包括 VS Code、Eclipse、Maven 和 Gradle。服务器配置(例如，添加或删除应用程序的功能或“特性”)是通过 XML 文件完成的。Open Liberty 的零迁移政策意味着您可以专注于重要的事情(编写您的应用程序！)而不必担心 API 在您的管理下发生变化。

## 在容器中部署到任何云

当您准备好部署您的应用程序时，您可以将其打包并部署到 OpenShift。零迁移原则意味着新版本的 Open Liberty 功能不会破坏你的应用程序，你可以控制你的应用程序使用哪个版本的功能。

通过 MicroProfile [指标](https://www.openliberty.io/guides/microprofile-metrics.html)、[健康](https://www.openliberty.io/guides/kubernetes-microprofile-health.html)和 [OpenTracing](https://www.openliberty.io/guides/microprofile-opentracing.html) 来监控实时微服务，这为您的应用添加了可观察性。您的应用程序和 Open Liberty 运行时发出的指标可以使用 Prometheus 进行整合，并显示在 Grafana 中。

## 了解开放的 Liberty 开发人员指南

我们的 [Open Liberty 开发者指南](https://www.openliberty.io/guides/)提供可运行代码和解释，帮助你学习如何用 [MicroProfile](https://openliberty.io/guides/?search=microprofile&key=tag) 和 [Jakarta EE](https://openliberty.io/guides/?search=jakarta%20ee) 编写微服务，然后将它们部署到 Red Hat OpenShift。

## 开始

要开始使用 Open Liberty，请尝试[打包和部署应用指南](https://openliberty.io/guides/getting-started.html)和[部署微服务到 OpenShift 指南](https://openliberty.io/guides/cloud-openshift.html)。

*Last updated: July 1, 2020*