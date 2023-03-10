# 云原生 Java 应用变得简单:Eclipse JKube 1.0.0 现已推出

> 原文：<https://developers.redhat.com/blog/2020/09/09/cloud-native-java-applications-made-easy-eclipse-jkube-1-0-0-now-available>

在 Eclipse Foundation 的[九个月孵化之后，](https://projects.eclipse.org/projects/ecd.jkube/governance) [Eclipse JKube](https://github.com/eclipse/jkube) 1.0.0 终于来了。这个版本标志着伟大的 [Fabric8 Maven 插件](https://github.com/fabric8io/fabric8-maven-plugin/) (FMP)项目的最终弃用。JKube 是 FMP 的完全替代品，包含了所有的主要特性。依赖 FMP 来创建 [Apache Maven](https://maven.apache.org/) Java 容器的项目应该[迁移到 Eclipse JKube](https://www.eclipse.org/jkube/docs/migration-guide/) 以充分利用本文中描述的新特性、错误修复和上游项目维护。

JKube 是一个插件集合，外加一个适合你的 Maven 项目的独立 Java 库。如果你有一个 [Java](https://developers.redhat.com/topics/enterprise-java/) 项目需要被部署到 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 或 [Red Hat OpenShift](https://developers.redhat.com/openshift) 中，这就是适合你的工具。JKube 负责与集群部署相关的一切，而作为开发人员，您可以专注于实现您的应用程序，而不用担心需要将它部署到哪里。

## Eclipse JKube 是如何工作的？

JKube 提供了具体的*目标*和一套实现这些目标的开发工具。目标包括[检查](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#jkube:log)容器日志，[观察](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#jkube:watch)项目变更，[调试](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#jkube:debug)云中的 Java 应用程序。一旦配置了这些目标，您就可以使用 JKube 通过各种构建策略来构建 Java 容器映像，包括源到映像(S2I)。只需添加我们的 [kubernetes-maven-plugin](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin) 或 [openshift-maven-plugin](https://www.eclipse.org/jkube/docs/openshift-maven-plugin) 依赖项，然后您的应用就可以为您的 kubernetes 或 openshift 集群构建并部署了。

JKube 从适用于大多数 Java 应用程序的自以为是的缺省值中推断出它的配置。如果这个设置不适合你的项目，你可以定制插件来满足你特定的项目需求。该项目还生成 Kubernetes 和 OpenShift 资源描述符和配置，并提供将它们部署到 k8s 兼容集群中的方法。

## 有什么新鲜事？

如果您是从 Fabric8 Maven 插件进入 Eclipse JKube 的，您会很高兴地看到 JKube 1.0.0 GA 中的这些新特性和更新:

*   支持我们所有发电机的 S2I 制造战略。
*   [支撑](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#_jib_java_image_builder)用于[起重臂](https://github.com/GoogleContainerTools/jib)(无码头)建造和推动。
*   Kubernetes 和 OpenShift 的独立插件，包括 OpenShift 的特定资源和构建策略。

此外，所有基础映像都基于 [Java 11](https://developers.redhat.com/topics/enterprise-java) 。

## 了解关于 Eclipse JKube 的更多信息

如果你想了解更多关于 Eclipse JKube 的信息，请访问项目[网站](https://www.eclipse.org/jkube/)，查看我们的[快速入门](https://github.com/eclipse/jkube/tree/master/quickstarts/maven)，并访问 [Katacoda 课程](https://katacoda.com/jkubeio)。你也可以通过[推特](https://gitter.im/eclipse/jkube)联系我们，别忘了在[推特](https://twitter.com/jkubeio)上关注我们！

## 向原创 Fabric8 贡献者致敬

由于 [Fabric8 Maven Plugin](https://github.com/fabric8io/fabric8-maven-plugin/graphs/contributors) 项目团队的[辛勤工作](https://github.com/eclipse/jkube/blob/master/REBRANDING.md)和原始代码的开发，Eclipse JKube 的发布成为可能。