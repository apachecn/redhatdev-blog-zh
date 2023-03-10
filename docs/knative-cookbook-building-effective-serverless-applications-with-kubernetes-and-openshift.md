# Knative Cookbook:使用 Kubernetes 和 OpenShift 构建有效的无服务器应用程序

> 原文：<https://developers.redhat.com/blog/2020/04/23/knative-cookbook-building-effective-serverless-applications-with-kubernetes-and-openshift>

[无服务器架构](https://en.wikipedia.org/wiki/Serverless_computing)最近在云原生应用部署中占据了中心位置:企业开始看到无服务器应用为他们带来的好处，如敏捷性、快速部署和资源成本优化。与任何其他新技术一样，有多种方法可以接近和使用无服务器技术，例如功能即服务(FaaS)和后端即服务(BaaS)，也就是说，将您的应用程序作为短暂的容器运行，能够自动伸缩。

[![Knative Cookbook front and back cover](img/850b0000b46f8b97e3ee6589e57b871a.png)](https://developers.redhat.com/books/knative-cookbook)

[Knative](https://knative.dev) 成立之初的简单目标是拥有一个 Kubernetes 本地平台来构建、部署和管理您的无服务器工作负载。Kubernetes 解决了许多云原生应用程序的问题，但也有一些复杂性，尤其是从部署的角度来看。要使用 Kubernetes 进行简单的服务部署，开发人员必须编写最少两个[YAML](https://yaml.org)(比如一个部署服务)，然后执行必要的管道工作以向外界公开服务。这种复杂性导致应用程序开发人员花费更多的时间来设计 YAMLs 和其他核心平台任务，而不是关注业务需求。

我举个例子来解释一下这个问题。假设我想部署一个 hello world 类型的应用程序并公开服务。首先，我需要创建一个部署，所以这里我创建了一个名为`myboot`的部署:

No video provider was found to handle the given URL. See [the documentation](https://www.drupal.org/node/2842927) for more information.

接下来，我需要将这个部署公开为名为`myboot`的服务(例如，应用程序即服务):

https://gist.github.com/kameshsampath/1bd96f24e25371f280863ee8522fdab9

但是等等，每次我想将应用程序部署为服务时，我需要编写这些复杂的 YAMLs 吗？不幸的是，我做到了，直到 Knative 出生。Knative 通过一个更简单的部署模型提供了所有必要的中间件原语，从而解决了这些 Kubernetes 问题。在 Knative 上，您可以部署任何现代应用程序工作负载，例如整体应用程序、微服务，甚至是微小的功能。Knative 可以在任何运行 Kubernetes 的云平台上运行，这使企业在运行其无服务器工作负载时更加敏捷和灵活，而不依赖于云供应商特定的功能。

让我向您展示一下，当您将相同的`myboot`部署为一个 Knative 服务时，您的部署是多么简单:

https://gist.github.com/kameshsampath/a28fe7848c89f655a3b7c611979dae27

虽然更简单的部署模型是 Knative 提供的一个关键特性，但是 Knative 的功能远不止于此。这就是 we——作为 Red Hat 开发者计划的一部分——认为帮助开发者快速轻松地投入 Knative 会很棒的地方，这催生了 [Knative 教程](https://github.com/redhat-developer-demos/knative-tutorial) 。本教程不仅提供了 Knative 的入门经验，也为下一个级别做好了准备。

事实上有许多方法可以实现无服务器，这导致了开发者的困惑，以下问题立即被提出:

*   我应该选择哪种实现:FaaS 还是 BaaS？
*   最快的入门方式是什么？
*   我可以应用无服务器技术的用例有哪些？
*   我如何衡量收益？
*   我应该使用什么工具来开发无服务器应用程序？

当我们开始探索无服务器技术作为撰写 Knative 教程的一部分时，我们也有同样的问题。我们在研究过程中面临的问题和挑战成为了这本烹饪书的症结所在。

随着 Knative 项目的加速发展，很明显应该有一本指南书来提供 Knative 和典型操作场景的实际应用。正是看到了这种需求，伯尔·萨特和我创作了这本*烹饪书*，它涵盖了以下主题:

*   将 Knative 安装到您的 Kubernetes 集群中。
*   自动缩放至零。
*   纵向扩展以应对请求高峰。
*   以无服务器的方式响应外部事件刺激。
*   使用阿帕奇卡夫卡与 Knative Eventing。
*   使用[Kubernetes](https://developers.redhat.com/topics/kubernetes/)[Knative](https://knative.dev)[卡夫卡](https://kafka.apache.org)和 [卡迈勒](https://github.com/apache/camel-k/) 为 [4K 云-原生计算](https://developers.redhat.com/devnation/tech-talks/4K-Kubernetes-with-Knative-Kafka-and-Kamel/watch/)。

[在这里](https://developers.redhat.com/books/knative-cookbook/?v=1)下载*烹饪书*，我们希望这本电子书能对你的烹饪之旅有所帮助！

*Last updated: June 29, 2020*