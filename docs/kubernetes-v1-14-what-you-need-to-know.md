# Kubernetes v1.14 版:你需要知道的

> 原文：<https://developers.redhat.com/blog/2019/03/25/kubernetes-v1-14-what-you-need-to-know>

[Kubernetes v1.14](https://kubernetes.io) 今天发布。该版本的主题是可扩展性和支持更多的工作负载。它包括 31 个增强功能，创纪录地有 10 个功能从测试版升级到稳定版。以下是亮点:

## 持久本地存储

这项功能以前是测试版，现在被归类为稳定版。持久本地存储的主要用例是数据库和分布式文件系统。显然，本地存储的性能优于远程磁盘，无论该存储是由云提供商提供的本地 SSD 还是连接到裸机系统的磁盘。这从 Kubernetes 1.5 版就开始了，所以它的升级是一个重要的里程碑。

## 对 Windows 节点的生产级支持

如果你是一家只购买 Red Hat Enterprise Linux 的商店，这个特性不会给你带来任何好处，但是绝大多数企业都有包含 Windows 工作负载的异构环境。随着这一特性进入稳定状态，您不必寻找不同的 orchestrator 来管理这些 Windows 容器。Kubernetes v1.14 提供了对 pods、服务类型、工作负载控制器和指标/配额的改进支持，与 Linux 的这些功能相当。

## PID 限制现在处于测试阶段

Kubernetes 管理员关心的一个问题是 Linux 主机上进程 id(PID)的分配。在过去，没有什么可以阻止容器内的进程创建 PID。如果 PIDs 的系统供应耗尽，即使其他资源不受限制，也会导致主机不稳定。管理员现在可以为每个 pod 的 PID 数量设置默认值(beta ),并为一个 pod 内的每个节点保留一定数量的可分配 PID(alpha)。

## Pod 优先级和抢占支持

您现在可以为 pods 分配优先级，以便 Kubernetes 控制器可以在集群资源不足时做出更好的决策。不太重要的豆荚可以被删除，为更重要的豆荚腾出空间。

## 从 RBAC 基础架构中删除的发现 API

默认情况下，有一组 API 允许未经身份验证的访问。在 1.14 版中，发现 API 已从该集中删除，以提高隐私性和安全性。

## `kustomize`与`kubectl`整合

[`kustomize`](https://kustomize.io) 允许您以声明方式配置资源。在 1.14 版中，这些功能可以通过`-k`标志或`kustomize`子命令通过`kubectl`获得。详见`kubectl`文档的[App 定制部分。](https://kubectl.docs.kubernetes.io/pages/app_customization/introduction.html)

## `kubectl`外挂机制现已稳定

`kubectl`插件是向`kubectl`添加子命令的独立二进制文件。这些子命令可以提供基本`kubectl`发行版中没有的新特性和定制特性。[Kubernetes 文档的用插件扩展`kubectl`部分](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/)解释了它是如何工作的。

## 更新的`kubectl`文档。和一个标志。还有一个吉祥物。

`kubectl`的文档已经从头开始重写。你可以在[kube CTL . docs . kubernetes . io](https://kubectl.docs.kubernetes.io)找到。它以一系列章节的形式发布，带您了解使用`kubectl`可以做的一切，包括如何管理、定制和部署应用程序；获取有关资源的信息；调试容器等等。

这些文档包括新的`kubectl`徽标。它以海底为主题，新的[乌贼](https://en.wikipedia.org/wiki/Cuttlefish)吉祥物游过 Kubernetes 标志。吉祥物的官方发音是*kubee-cudding*，很可能是任何关于命令结尾应该发音为“控制”的争论的丧钟

## 香肠是如何制作的

有关上述一些特性的更多背景信息，请参见 GitHub 上的问题及其相关讨论:

*   [持久(非共享)本地存储管理(第 121 期)](https://github.com/kubernetes/enhancements/issues/121)
*   [强化默认的 RBAC 发现集群角色绑定(问题 789)](https://github.com/kubernetes/enhancements/issues/789)
*   [PID 限制(第 757 期)](https://github.com/kubernetes/enhancements/issues/757)
*   Pod 优先级和抢占支持(问题 268 和 [564](https://github.com/kubernetes/enhancements/issues/564)
*   [支持 K8s 的 Windows 服务器容器(第 116 期)](https://github.com/kubernetes/enhancements/issues/116)

*Last updated: September 3, 2019*