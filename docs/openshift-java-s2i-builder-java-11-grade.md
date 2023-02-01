# 为 OpenShift 构建 Java 11 和 Gradle 容器

> 原文：<https://developers.redhat.com/blog/2018/12/18/openshift-java-s2i-builder-java-11-grade>

你如何让你的 [Java](https://developers.redhat.com/blog/category/java/) 应用在云中运行？

首先，你从天空抓取一个云，例如，(1) [从 Red Hat OpenShift Online](https://www.openshift.com/learn/get-started/) 上的免费帐户开始，或者(2)在本地的笔记本电脑上使用[Red Hat Container Development Kit(CDK)](https://developers.redhat.com/products/cdk/overview/)或 [upstream Minishift](https://docs.okd.io/latest/minishift/getting-started/preparing-to-install.html) 在 Windows、macOS 和 Linux 上，或者(3)使用`oc cluster up`(仅在 Linux 上)，或者(4)从在公共云或内部云上运行 [Red Hat OpenShift](http://openshift.com/) 的人那里获得登录。然后，您[下载 oc CLI 客户端工具](https://www.okd.io/download.html)可能是针对 Windows 的(并将其放在您的 PATH 中)。然后，在 OpenShift 控制台的 UI 中，从您名字下的右上角菜单中选择*复制登录命令*，并使用例如`oc status`命令。

太好了——现在你只需要[容器化](https://developers.redhat.com/blog/category/containers/)你的 Java 应用程序。当然，您可以开始编写自己的 workers 文件，选择一个合适的容器基础映像(并与您的同事讨论 [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/) 对 CentOS 对 Fedora 对 Ubuntu 对 Debian 对 Alpine 并且，特别是如果您在企业环境中，弄清楚如何在生产中支持它)，为容器弄清楚合适的 JVM 启动参数，添加监控，等等。

但也许你今天真正想做的是...好吧，让你的 Java 应用在云中运行吧！

请继续阅读，寻找更简单的方法。

## 使用 Maven

为此，我喜欢使用 OpenShift 的源到图像(S2I)构建器，因为我发现这是从源代码到运行容器最快最简单的方法。您的应用程序的构建可能已经产生了一个自包含的“胖罐子”，因为您可能正在使用像 Spring Boot、Thorntail.io(以前称为 WildFly Swarm)或 Vert.x do 这样的现代框架。为了使本文简单，让我们使用[这个“Hello World”服务器](https://github.com/vorburger/s2i-java-example/blob/master/src/main/java/ch/vorburger/openshift/s2i/example/Server.java)和这个[最简单的 Maven POM](https://github.com/vorburger/s2i-java-example/blob/master/pom.xml) 。字面上的*和下面的命令一样简单，只需一分钟就可以编译、打包、启动并运行:*

```
$ oc new-app https://github.com/vorburger/s2i-java-example
```

如果您的 OpenShift 配置没有任何注册的 Java builder，或者它不知道为您选择哪一个(*错误:多个图像或模板匹配“JEE”*)，[，那么您可以指定它应该使用上游 Fabric8 社区提供的这个](https://github.com/fabric8io-images/s2i/tree/master/java/images/centos):

```
$ oc new-app fabric8/s2i-java~https://github.com/vorburger/s2i-java-example
```

然后，您可以通过公开`new-app`作为路由创建的服务来访问演示:

```
$ oc expose svc/s2i-java-example; oc get route s2i-java-example
```

## Gradle, too!

这个`fabric8/s2i-java` S2I 构建器实际上**不仅适用于 Maven，也适用于基于 Gradle 构建的项目**(我[增加了对 Gradle](https://github.com/fabric8io-images/s2i/issues/118) 的支持；参见说明如何使用此的[示例项目)，如下所示:](https://github.com/fabric8io-images/s2i/tree/master/java/examples/gradle)

```
$ oc new-app fabric8/s2i-java~https://github.com/fabric8io-images/s2i --context-dir=java/examples/gradle --name s2i-gradle-example
```

顺便说一下，如果您的构建在另一个位置生成了一个 JAR 文件，或者使用了 S2I Java 映像期望找到的另一个名称，那么您可以[将一个. s2i/environment 文件放到您的项目中来覆盖](https://github.com/fabric8io-images/s2i/tree/master/java/images/centos#running-a-java-application) `ARTIFACT_COPY_ARGS`或`JAVA_APP_JAR`或者设置您的应用程序可能需要的任何定制的固定环境变量。

## 还有 Java 11！

`fabric8/s2i-java` S2I 镜像现在具有在 Java 11 : 下构建和运行 **[的功能](https://github.com/fabric8io-images/s2i/issues/160)**

```
$ oc new-app fabric8/s2i-java:latest-java11~https://github.com/vorburger/s2i-java-example
```

Java 11 特性是全新的，很可能会发展；欢迎社区反馈关于[任何需要的 JVM 启动参数调整](https://github.com/fabric8io-images/s2i/issues/191)，与使用 jlink 的 Java 11 模块性的[更小的 JDK 基础映像相关的工作，以获得更快的启动和更少的 JRE 内存消耗(并且，希望](https://github.com/fabric8io-images/s2i/issues/181)[会更小](https://bugzilla.redhat.com/show_bug.cgi?id=1652177)，使用更[的最小容器基础映像](https://github.com/fabric8io-images/s2i/issues/192)，或者[可能是 32 位变体](https://github.com/fabric8io-images/s2i/issues/202)。非常欢迎投稿！

注意:红帽不支持 Fabric8 社区提供的图像。

*Last updated: January 13, 2022*