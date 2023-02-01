# 在 OpenShift 容器平台 4 上部署红帽 AMQ 流和 Fuse

> 原文：<https://developers.redhat.com/blog/2019/10/03/deploy-red-hat-amq-streams-and-fuse-on-openshift-container-platform-4>

在下面的视频中，我演示了如何在 [OpenShift 4](https://developers.redhat.com/openshift/) 上部署[红帽 AMQ 流](https://developers.redhat.com/blog/2019/07/04/announcing-red-hat-amq-streams-1-2-with-apache-kafka-2-2-support/)(基于上游阿帕奇卡夫卡)。

我还将通过使用[红帽保险丝](https://developers.redhat.com/products/fuse/overview)演示如何使用 AMQ 流的基本方法。有一个 Camel route 在`/goodbye`暴露了一个 REST 端点，当点击它时，它向主题发送一个“再见世界”消息。还有一个定时器定期向主题发送“Hello World”消息。一个单独的 Camel route 从主题中提取内容，并记录消息以提高我们的可见性。

看看在 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 上启动和运行 AMQ 流有多简单！

[https://www.youtube.com/embed/S1PmT01FJ80?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/S1PmT01FJ80?autoplay=0&start=0&rel=0)

*Last updated: July 1, 2020*