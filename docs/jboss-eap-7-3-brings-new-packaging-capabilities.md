# JBoss EAP 7.3 带来了新的打包功能

> 原文：<https://developers.redhat.com/blog/2020/04/10/jboss-eap-7-3-brings-new-packaging-capabilities>

除了大量的[新特性和改进](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html-single/7.3.0_release_notes/index#new_features_and_enhancements)之外，[发布的 Red Hat JBoss Enterprise Application Platform 7.3](https://www.redhat.com/en/blog/announcing-availability-red-hat-jboss-enterprise-application-platform-73)提供了创新的打包功能。在本文中，我将重点介绍其中的两个新功能，并展示它们的好处:将映像分为构建和运行时，以及使用 [Galleon](https://docs.wildfly.org/galleon/) 进行配置调整。

## 分离构建映像和运行时映像

JBoss EAP 映像的传统容器化包括运行一个 [S2I 构建](https://docs.openshift.com/container-platform/latest/builds/understanding-image-builds.html#build-strategy-s2i_understanding-image-builds)，它将一个基本 JBoss EAP 映像(包含 EAP 中的所有内容)与应用程序构建相结合。这种旧的做事方式导致最终的映像中有许多不必要的东西，比如 Maven 和 S2I 工具这样的构建工具。

在 JBoss EAP 7.3 中，有新的精简的运行时映像，可以在管道中作为链式构建进行构建，这样最终的映像就小得多，并且不包含构建时所需的所有工具。这种行为提供了较小的图像大小，并节省了网络带宽和存储成本。

## 用帆船修整

现在，您可以定制主 JBoss EAP for OpenShift 映像配置，使其只包含您需要的*功能，从而减少内存占用和启动时间。供应工具 [Galleon](https://docs.wildfly.org/galleon/) 提供了几个层，您可以选择这些层来控制 JBoss EAP 服务器中的功能。该服务器包含几个[支持的预定义 Galleon 层](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/getting_started_with_jboss_eap_for_openshift_container_platform/capability-trimming-eap-foropenshift_default)，开发者和运营团队可以创建自定义层，用于以可重复的方式添加小部分功能。*

这两个功能提供了许多好处，演示视频中对此进行了概述。来看看吧，一定要试试 JBoss EAP 7.3，发现许多其他令人惊叹的新特性和改进！

[https://www.youtube.com/embed/UvLh5G8prT0?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/UvLh5G8prT0?autoplay=0&start=0&rel=0)

*Last updated: June 29, 2020*