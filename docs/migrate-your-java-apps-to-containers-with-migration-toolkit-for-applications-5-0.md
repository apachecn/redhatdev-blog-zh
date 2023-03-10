# 使用应用迁移工具包 5.0 将您的 Java 应用迁移到容器中

> 原文：<https://developers.redhat.com/blog/2020/09/04/migrate-your-java-apps-to-containers-with-migration-toolkit-for-applications-5-0>

作为一名开发人员，你可能已经尝试过使用 Kubernetes。也有可能你已经在 Kubernetes 平台上运行了几个 [Java 应用](https://developers.redhat.com/topics/enterprise-java)，也许是 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 。这些最初的[容器化的](https://developers.redhat.com/topics/containers)应用程序是绿地项目，在那里你可以享受到平台的好处，提供模板化的部署、简单的回滚、资源可用性、默认的安全性以及发布服务的可管理方式。

现在，您可能会想，“我如何在现有的 Java 应用程序中享受所有这些好处？”当今生产中的大多数 Java 应用程序都运行在虚拟机(VM)上，很可能是在一个对容器不友好的应用程序平台上。那么，如何将它们从当前平台迁移到 Kubernetes 上的容器中呢？

这不是一项简单的任务，但这是一个我们多年来一直在努力解决的问题。Red Hat 的 Migration Toolkit for Applications(MTA)5.0 是最新的迭代:一个工具集合，您可以使用它来分析现有的应用程序，并发现更新它们需要什么。请继续阅读，了解 MTA 5.0 的功能和迁移路径。

## 使您的应用产品组合现代化

Red Hat 工程师、架构师和顾问花费了数年时间，从大量应用程序的平台迁移中吸取经验教训。我们将从这些 Java 迁移挑战中学到的东西转化为迁移规则，然后开发了一个工具包(从[开始，一个名为 Windup](https://github.com/windup/) 的开源项目，我们后来发布了 Red Hat Application Migration Toolkit)来帮助自动化分析应用迁移的过程，并使它们可重复。有了这个工具包，您甚至可以在开始迁移之前就发现问题。

现在，我们已经进入了企业应用程序迁移发展的新阶段。我们专注于容器化的工作负载，同时保留我们迄今为止学到的所有宝贵经验。Red Hat 的 Migration Toolkit for Applications(MTA)5.0 是下一个迭代:一个工具的集合，您可以使用它来分析现有的应用程序，并发现使它们现代化需要什么。

潜在的 MTA 移民途径包括:

*   在 Windows 上对运行在 Oracle JDK 上的 Oracle WebLogic 进行现代化:将这些应用迁移到[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/videos/vimeo/95462201)(EAP)上，并在 [OpenJDK](https://developers.redhat.com/products/openjdk/download) 和[Red Hat Enterprise Linux(RHEL)](https://developers.redhat.com/topics/linux)上运行——所有这些都在虚拟机中进行。
*   使 Tomcat 在 Ubuntu 上的 Oracle JDK 上可支持:将您的应用程序移动到 RHEL [通用基础映像](https://developers.redhat.com/videos/youtube/VG7Y1mjVIE0) (UBI)上的[Red Hat JBoss Web Server](https://developers.redhat.com/products/webserver/overview)(Red Hat 的 Tomcat 版本)和 OpenShift 上运行在容器中的 OpenJDK。
*   将 OpenJDK 和 Windows stack 上的 JBoss EAP 迁移到 OpenJDK 上的 JBoss EAP 和 OpenShift 上的容器中运行的 RHEL UBI。

## 应用程序迁移工具包 5.0

migration Toolkit for Applications 5.0 现已面向所有需要更简单的方法来更新和迁移 Java 应用程序的开发人员开放。MTA 是免费的，有五种不同的使用方式:

*   **WebUI** :可以在笔记本电脑上运行或者部署在 OpenShift 3.11 或 4.x 上的用户界面。
*   **命令行界面(CLI)** :最适用于分析大量应用，以及将 MTA 嵌入到当前的 CI/CD 管道中。
*   **IDE 插件**:
    *   使用 Eclipse desktop IDEs、 [CodeReady Studio](https://developers.redhat.com/products/codeready-studio/overview) 和[Visual Studio Code(VS Code)(tech preview)](https://marketplace.visualstudio.com/items?itemName=redhat.mta-vscode-extension)进行本地开发。
    *   使用 Eclipse Che 和 [CodeReady 工作区](https://developers.redhat.com/products/codeready-workspaces/overview)进行集中的浏览器内开发。
*   **Maven 插件**:使用 [Maven 插件](https://access.redhat.com/documentation/en-us/migration_toolkit_for_applications/5.0/html-single/maven_plugin_guide/index)在构建期间分析应用程序。

图 1 显示了标有支持应用程序的 MTA 5.0 控制台。

[![](img/a3b4a2d3c81b9ced5397bec63b2784a6.png "img_5f43a0f6e2588")](/sites/default/files/blog/2020/08/img_5f43a0f6e2588.png)

Figure 1: The MTA 5.0 console showing supported applications.

**注意**:关于开发者和架构师如何使用各种组件的更多信息，请参见[MTA 5.0 产品页面](https://developers.redhat.com/products/mta/download)。

## 许多迁移路径

migration Toolkit for Applications 5.0 支持一组广泛的转换路径，并针对以下情况提供了规则:

*   Apache Camel 2 到 Camel 3 : MTA 5.0 包含新的规则，帮助开发人员使用最新版本的 Apache Camel 3 更新他们的应用程序，有 13 个规则集和 147 个规则。图 2 显示了特定于 Camel 2 到 Camel 3 迁移的规则。(特别感谢 Matej Melko 和 John Poth 对这些规则的贡献。)

[![](img/e4f1177ccd760eb44b71e6e2603eb7e9.png "img_5f3ea15f3080a")](/sites/default/files/blog/2020/08/img_5f3ea15f3080a.png)

Figure 2: A rule specific to the Camel 2 to Camel 3 migration.

*   **Spring Boot 到夸尔库斯**:我们已经植入了从 [Spring Boot](https://developers.redhat.com/topics/spring-boot) 迁移到[夸尔库斯](https://developers.redhat.com/products/quarkus/getting-started)的三条新规则。对于即将到来的 MTA 5.1 版本，我们计划添加一组 28 条规则。已经制定了 14 条规则，2 条正在制定中，12 条正在酝酿中。图 3 显示了用适当的 Quarkus PicoCli 接口替换 Spring Shell 依赖项的规则。

[![](img/bd358937979cb9cf529c064ad5ae01a5.png "img_5f3ea14373426")](/sites/default/files/blog/2020/08/img_5f3ea14373426.png)

Figure 3: A Spring Boot-to-Quarkus migration rule.

*   JBoss EAP :我们增加了一套全面的规则来确保迁移的应用符合 Java/Jakarta Enterprise Edition(JEE)标准，这是在 JBoss EAP 上工作的一个要求。如图 4 所示，在使用 WebLogic 特定的记录器时出现了一个问题，它提供了使用标准 Java 记录器的建议。

[![](img/134f277cba4f2cd5a5812e89a5896a25.png "img_5f43a548490ed")](/sites/default/files/blog/2020/08/img_5f43a548490ed.png)

Figure 4: Choose the correct logger for your migration.

*   **容器**:我们为容器化应用添加了基于[十二要素应用方法](https://12factor.net/)的规则。图 5 显示了禁止使用 Java RMI 的规则。

[![](img/501f94b4739030a4c50ae0b54d696fa4.png "img_5f43a56a46333")](/sites/default/files/blog/2020/08/img_5f43a56a46333.png)

Figure 5: Java RMI should be avoided in cloud environments.

*   Linux:我们增加了一些规则来帮助你避免在 Windows 上使用 Java 时出现的问题。如图 6 所示，其中一个规则要求用 Linux 风格的路径替换 Windows 文件系统路径。

[![](img/ee5e07a5a3a21517ce5cc94e5385584a.png "img_5f3ea2d09e314")](/sites/default/files/blog/2020/08/img_5f3ea2d09e314.png)

Figure 7: Consider replacing Java 2D with the OpenJDK Java 2D library.

*   **OpenJDK** :这个规则帮助您避免使用不在 JDK 中的特定专有库所带来的问题，比如 Java 2D 库(如图 7 所示)。

[![](img/ee5e07a5a3a21517ce5cc94e5385584a.png "img_5f3ea2d09e314")](/sites/default/files/blog/2020/08/img_5f3ea2d09e314.png)

Figure 7: Consider replacing Java 2D with the OpenJDK Java 2D library.

**注意**:参见[转换和组合的完整矩阵](https://developers.redhat.com/products/mta/use-cases)，MTA 5.0 使其变得更加简单。

## 开始使用 MTA 5.0

点击此处[下载应用迁移工具包](https://developers.redhat.com/products/mta)。[入门指南](https://developers.redhat.com/products/mta/getting-started)让你在五分钟内启动并运行 MTA 5.0。

如果您想了解更多关于使用 MTA 来更新和迁移您的 Java 应用程序的真实体验，请观看这个视频:[架构师和开发人员如何使用 MTA](https://www.youtube.com/watch?v=t3VkRpFlkn0) 。有关更深入的介绍，请参见[应用迁移工具包视频演示](https://youtu.be/mRCz6Jl0Ds8)。

你可能还想看一看 [MTA 5.0 文档](https://access.redhat.com/documentation/en-us/migration_toolkit_for_applications/)以了解如何充分利用 MTA。例如，您可以使用文档来学习如何[编写自己的迁移规则](https://access.redhat.com/documentation/en-us/migration_toolkit_for_applications/5.0/html/rules_development_guide/index)。

关注我们 [@MTAbyRedHat](https://twitter.com/MTAbyRedHat) 了解最新动态并直接参与我们的团队。我们期待看到您的应用更上一层楼！

*Last updated: October 28, 2020*