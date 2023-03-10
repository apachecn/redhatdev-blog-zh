# Camel K 1.0:无服务器集成平台正式上市

> 原文：<https://developers.redhat.com/blog/2020/06/18/camel-k-1-0-the-serverless-integration-platform-goes-ga>

经过多月的等待，[阿帕奇骆驼 K 1.0](https://github.com/apache/camel-k/releases/tag/1.0.0) 终于来了！这个开创性的项目向开发人员介绍了云原生应用程序开发和自动化云配置，不费吹灰之力。随着 1.0 通用版(GA)的发布，Apache Camel K 比以往任何时候都更加稳定，开发人员将会对其性能改进非常满意。

## 关于骆驼 K

Apache Camel K 是一个轻量级集成平台，原生运行在 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 上。它使用[操作模式](https://developers.redhat.com/topics/kubernetes/operators)来简化应用程序开发生命周期。例如，您可以使用运营商轻松连接到 Apache Kafka 或 [Red Hat AMQ 流](https://www.redhat.com/en/resources/amq-streams-datasheet)。Camel K 还自带内置工具和许多预构建的企业模式，以便于重用。

Apache Camel K 是为性能而构建的，具有快速的构建时间、较小的内存占用和快速的启动时间。它还与 [Knative](https://developers.redhat.com/blog/2020/04/23/knative-cookbook-building-effective-serverless-applications-with-kubernetes-and-openshift/) 无缝集成，用于[无服务器](https://developers.redhat.com/topics/serverless-architecture/)工作负载，可以自动扩展和缩减。

## 了解有关 Camel K 1.0 GA 版本的更多信息

在最新的 Camel K 1.0 版本中，现在内置了监视和跟踪功能，以及透明的日志记录来改进调试。还可以使用 Camel K 定义云上的`cron`作业。

用于 Camel K 的工具现在也可用于 VS 代码 IDE，为集成文件添加了自动完成和错误突出显示功能，并且[“Kamel”CLI](https://github.com/apache/camel-k/releases/tag/1.0.0)包含许多旨在改善整体用户体验的功能。

要了解关于 Apache Camel K 的更多信息，并获得 1.0 GA 版本中新增和改进的详细技术概述，请参见 Apache Camel 博客上我的[发布公告](https://camel.apache.org/blog/2020/06/camel-k-release-1.0.0/)。我在那里的帖子将引导您了解 1.0 GA 版本中的新的很酷的特性，并帮助您开始使用 Apache Camel K。

*Last updated: June 25, 2020*