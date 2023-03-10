# 保守 Kubernetes 的秘密

> 原文：<https://developers.redhat.com/blog/2020/09/07/keeping-kubernetes-secrets-secret>

开发技术讲座由创造我们产品的红帽技术专家主持。这些会议包括真正的解决方案加上代码和示例项目，以帮助您开始。在这次演讲中，你将从[亚历克斯·索托·布埃诺](https://developers.redhat.com/authors/alex-soto)和[伯尔·萨特](https://developers.redhat.com/blog/author/burrsutter/)那里学习如何管理[库伯内特](https://developers.redhat.com/topics/kubernetes)的秘密。

每个人都在谈论[微服务](https://developers.redhat.com/topics/microservices/)和[无服务器架构](https://developers.redhat.com/topics/serverless-architecture/)，以及如何使用像 Kubernetes 这样的集群管理器来部署它们。但是，那些秘密(比如证书、密码、SSH 和 API 密钥)呢？当前的趋势增加了运行我们的服务所需的秘密的数量。这一事实对我们的安全团队提出了新的维护要求。

在实例自动启动的动态场景中，或者出于可伸缩性的原因，同一服务有多个实例的动态场景中，我们如何共享和管理服务的这些秘密呢？你跟上了吗？

观看这段录制的视频，了解如何在 Kubernetes 中使用 [Vault](https://github.com/hashicorp/vault-k8s) 管理您的秘密，以及为什么 Kubernetes 的秘密本身可能还不够。开始让安全性成为您开发过程中的一等公民。

观看整个演讲:

[https://www.youtube.com/embed/pCgYqnGwXBM?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/pCgYqnGwXBM?autoplay=0&start=0&rel=0)

## 了解更多信息

加入我们即将到来的开发者大会，看看我们收集的[过去的开发技术演讲](https://developers.redhat.com/devnation/?page=0)。

*Last updated: September 3, 2020*