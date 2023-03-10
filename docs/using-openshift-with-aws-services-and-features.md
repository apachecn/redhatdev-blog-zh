# 将 OpenShift 与 AWS 服务和功能结合使用

> 原文：<https://developers.redhat.com/blog/2017/05/04/using-openshift-with-aws-services-and-features>

AWS 合作伙伴解决方案架构师 Mandus Momberg 展示了将 OpenShift 与 AWS 原生功能相集成的机制。在 AWS 上部署 OpenShift Container Platform 3.5 的 Red Hat [参考架构](http://red.ht/2qvCH6s)中涵盖了许多概念。

首先，如果你在 AWS 上运行或考虑 RHEL，查看一下[云访问程序](https://www.redhat.com/en/technologies/cloud-computing/cloud-access)。这允许您以 1:2 的比例将标准 RHEL 订阅转换为云访问许可，每个标准许可证对应 2 个云虚拟机。

这些年来，在扩展 Ansible 以全面支持 AWS 方面已经做了很多工作。事实上，Red Hat IT 在内部使用 Ansible 来提供所有 AWS 资源。在 AWS 上部署 OpenShift 时，OpenShift 安装程序实际上利用了这个功能，以及动态清单特性。当然，这些可以扩展以满足您的任何需求。

OpenShift 的 AWS 集成中更令人兴奋的新特性之一是服务目录。OpenShift 服务目录提供了一个简单的一键式 AWS 本地服务供应，它建立在 AWS Open Service Broker API 基础之上，这是一个用于与 AWS 集成的标准化 API。该目录有效地允许您的用户使用一站式界面来部署本地服务，如 RDS，以及基于 OpenShift 的应用程序。用于部署与 AWS 集成的 OpenShift 的 Ansible 脚本位于[这里](https://github.com/fusor/catasb)。

OpenShift 的另一个新特性是原子支持。Atomic 是一个基于 RHEL 核心组件的轻量级操作系统，可以作为 OpenShift 的计算引擎。它在一个易于管理的包中提供了底层容器技术。AWS 和 Red Hat 在 Atomic 上紧密合作，以便针对云工作负载进行优化。虽然普通 RHEL 在云环境中运行良好，但 Atomic 是专门为在云中运行容器而构建的——允许您挤出那些额外的 cpu 周期。

最后，在 AWS 上运行 OpenShift 有几种认可的架构模式，使用其中一种经验证的模式将使您的 AWS OpenShift 部署更加轻松和成功。

#### 单节点部署

这些实际上是为“开发者游乐场”准备的，将单个 OpenShift VM (all-in-one node)与 bastion 主机和负载平衡器结合在一起。CDK 是另一种提供类似开发环境的机制。

#### 特定工作负载 VPC

对于大多数应用程序，考虑特定于工作负载的 VPC 的首选模式，其中在 VPC 中有一个用于站点组件的专用 OpenShift 集群。例如，您可以部署这些 VPC:

*   支付处理
*   面向用户的服务
*   后端数据服务
*   共同事务

这是亚马逊 AWS 推荐的方法。

#### 高度可用

一个高度可用的 OpenShift AWS 模式包括设置冗余的 OpenShift 集群，每个集群都位于自己的 AWS 可用性区域内。您的应用程序在单个 VPC 内的每个 AZ 中运行。Route53 用于基于 DNS 的流量管理，将流量定向到适当的 AZ。

简而言之，在云环境中运行 OpenShift 有很多优势，对于主要的云提供商来说也有特殊的好处。亚马逊 AWS 和 Red Hat 多年来一直合作完善 OpenShift AWS 集成，为您提供取得成功所需的工具。