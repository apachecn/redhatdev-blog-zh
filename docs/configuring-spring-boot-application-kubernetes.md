# 在 Kubernetes 上配置 Spring Boot 应用程序

> 原文：<https://developers.redhat.com/blog/2017/10/02/configuring-spring-boot-application-kubernetes>

当开发人员计划在 Kubernetes 上部署[Spring Boot](https://developers.redhat.com/blog/2017/10/02/configuring-spring-boot-application-kubernetes/)应用程序时，Spring 开发人员想到的第一个问题是“我能使用 Spring Config 服务器吗？”[Spring Config serve](http://cloud.spring.io/spring-cloud-config)r 是一种事实上的分布式应用程序集中配置方式。**是的，**我们可以使用[Spring Config server](http://cloud.spring.io/spring-cloud-config)，但是让我们想到一些**约束**使得[Spring Config server](http://cloud.spring.io/spring-cloud-config)在一个典型的企业部署中可以有:

*   生产过程中无法访问互联网。
    *   这意味着如果客户使用其他 VCS，我就不能使用默认的 Git backed。
*   另一种可能是在本地运行自己的 ConfigServer。
    *   但是，您需要在每个环境中进行部署和设置，然后应用策略、规则等。
*   限制对配置服务器属性和文件的访问。
    *   通常是让*-prod 对其他环境用户或文件权限不可见。
*   我还可以使用哪些后端？
    *   如果我需要添加一个新的后端，我需要如何对我的配置服务器进行定制？
*   我无法将 [Spring Config server](http://cloud.spring.io/spring-cloud-config) 托管的属性挂载为卷/文件。

虽然可能会有解决这些限制的方法，但作为一名开发人员，我更喜欢使用我的平台本身提供的东西，而不是自己定制。使用本机方式的最大好处是清晰的 **关注点分离** ，我是一名开发人员，我不需要担心配置对我来说是如何可用的，只需确保我将拥有它！在 Kubernetes 内部有没有更好的方法来做同样的事情？答案是肯定的:)。那好吧，

1.  它们是什么？
2.  我如何使用它们？
3.  我可以用它的什么后端？
4.  安全吗？
5.  Spring Boot 能使用它们吗？
6.  那些类型的属性可以热重装吗？

为了回答所有这些问题，我决定写这个博客系列“**在 Kubernetes** 上配置 Spring Boot 应用程序”。

在我们进入细节之前，让我回答上面提出的一些问题。

## 它们是什么？

在 Kubernetes 中，我们可以使用 [ConfigMaps](https://developers.redhat.com/blog/2017/10/03/configuring-spring-boot-kubernetes-configmap/) 和 [Secrets](https://developers.redhat.com/blog/2017/10/04/configuring-spring-boot-kubernetes-secrets/) [来处理所有的应用配置和属性。](https://kubernetes.io/docs/concepts/configuration/secret/) 使用 ConfigMaps 和 Secrets ，应用自然满足[Config](https://12factor.net/config)原理 [十二因素 App](https://12factor.net/) 。

## 我如何使用它们？

有很多方法可以使用它们，但是我们将集中讨论使用主题的两种简单方法，即

*   为**环境变量**为
*   为**文件**内的容器

## 他们支持什么样的后端？

何时 [ConfigMaps](https://developers.redhat.com/blog/2017/10/03/configuring-spring-boot-kubernetes-configmap/) 和 [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) 可以作为文件从各种后端加载，如 S3、CephFS、GlusterFS、NFS 等等。更多详情请见[【https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/)。

## 安全吗？

使用 配置图 和 机密 作为文件是使 配置图 和 机密 安全的最简单方法。我们可以申请文件/目录权限。我们还可以根据 [名称空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) 与环境和 [服务帐户](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) 同义来限制访问。

回答问题"**Spring Boot 能用它们吗？** ， 我把博客系列分成了两部分，分别献给 ConfigMaps 和 秘密 。

博客文章提供了详细的演示源。源 repos 上的**自述**帮助你自己尝试东西(DIY)。

## Spring Boot 和库伯内特系列

[如何在 Kubernetes 上配置 Spring Boot 应用](https://developers.redhat.com/blog/2017/10/02/configuring-spring-boot-application-kubernetes/)

第一部分:[使用 ConfigMap 在 Kubernetes 上配置 Spring Boot](https://developers.redhat.com/blog/2017/10/03/configuring-spring-boot-kubernetes-configmap/)

第二部分:[配置 Spring Boot·库伯内特的秘密](https://developers.redhat.com/blog/2017/10/04/configuring-spring-boot-kubernetes-secrets/)

*Last updated: January 29, 2019*