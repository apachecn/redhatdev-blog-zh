# 如何定制红帽 OpenShift 3.11 SDN

> 原文：<https://developers.redhat.com/blog/2019/11/01/how-to-customize-the-red-hat-openshift-3-11-sdn>

在本文中，我将重点介绍一个定制 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 软件定义网络(SDN)的实际案例。为了实现这一点，我将确定配置 [OpenShift SDN](https://docs.openshift.com/container-platform/3.11/architecture/networking/sdn.html#architecture-additional-concepts-sdn) 不同方面的 OpenShift-Ansible 库存参数，特别是集群、门户和 docker 网络。

## 为什么要定制 SDN？

我经常被问到的一个重要问题是:为什么需要定制 SDN？不完全是内部的吗？用户通常认为没有必要定制 SDN，因为 OpenShift 的 SDN 对 OpenShift 集群之外的网络没有影响；因此，不应担心知识产权冲突。然而，情况并非总是如此。

在大型组织中，指定的专用网络(例如 10.0.0.0/8)通常在组织自己的网络内部“公开”使用。考虑到这一点，主要的问题是 OpenShift 集群外部的服务可能正在使用与集群本身使用的 IP 相冲突的私有范围内的 IP。

当 OpenShift 集群中的应用试图与这些冲突的外部服务进行通信时，这些冲突将导致问题，因为 OpenShift SDN 将试图在其自己的网络内路由这些请求，从而导致无法从集群内访问这些服务。

## 背景

### 默认值

OpenShift 在安装时配置的默认网络是:

| **组件** | **可变库存参数** | **注释** |
| 集群网络 | osm_cluster_network_cidr and osm_host_subnet_length which by default are 10.128.0.0/14 and 9 respectively. | 每个节点从该范围中获得一个“/23”子网，总共允许 512 个子网专用于节点，每个节点上总共有 510 个 IP 用于容器。运行“oc get pods - all-namespaces -o wide”将显示已经从该范围分配了哪些 IP，并且应该都在该范围内。 |
| 服务网络 | openshift_portal_net | By default this is 172.30.0.0/16\. This means you have a total of 65534 addresses to assign to services.运行“oc get services - all-namespaces”也应该显示所有服务 IP 都在这个范围内。 |
| 码头网络 | open shift _ docker _ options = "-bip 172 . 17 . 0 . 1/16-fixed-CIDR 172 . 17 . 0 . 0/17 " | 这些值是 docker 守护进程的命令行参数，由 openshift-ansible 中的 openshift_docker_options 变量配置。 |

### 自定义这些值

通过定义集群可以处理的节点、单元和服务的数量，对这些值进行微调可以决定集群的规模。考虑到这一点，对于开发集群，您可能希望允许更多的服务，而不是 pods，来满足开发人员的特定需求。但是，在生产系统中，您可能更喜欢通过扩展节点数量来满足业务和运营需求的能力。

本文基于只有子网 192.168.0.0/16 供整个集群使用的情况。考虑到这一点，我将子网划分如下:

| **子网** | **使用** |
| 192.168.0.0/17 | 集群网络(192 . 168 . 0 . 0–192 . 168 . 127 . 254)，总共支持 127 个"/24 "子网，相当于 127 个节点，每个节点上有 254 个 IP 可用于容器。 |
| 192.168.128.0/18 | 门户/服务网络(192.168.128.0-192.168.191.254)共有 16382 个服务 IP。 |
| 192.168.192.0/24 | docker 网络，如果我用 open shift _ docker _ options = "-bip 192 . 168 . 192 . 1/24-fixed-CIDR 192 . 168 . 192 . 1/25 "，给我总共 127 个 ip。这仅对于构建具有外部依赖性的映像很重要，并且不需要很大。 |

### 您可能需要定制 OpenShift SDN 的原因

再次重申，以下是您可能希望自定义默认 OpenShift 网络的一些原因:

*   安装时创建的默认 OpenShift 和 docker 网络与可路由网络范围内运行的组织服务相冲突。
*   网络限制意味着无法将建议的单个“/14”网络专用于集群，而将两个“/16”网络专用于 OpenShift 服务和 docker。

## 最终配置

因此，以下配置反映了我们定义的较小网络。

| **存货参数** | **值** |
| osm_cluster_network_cidr | 192.168.0.0/17 |
| osm _ 主机 _ 子网 _ 长度 | eight |
| openshift_portal_net | 192.168.128.0/18 |
| openshift _ docker _ options 开启 hift _ dock _ 选项 | "- bip 192.168.192.1/18 -固定-cidr 192.168.192.1/19 " |

## 结论

在本文中，我强调了 OpenShift 3.11 清单变量，这些变量决定了在安装 OpenShift 时如何创建 OpenShift 的 SDN。我还指出了您考虑让您的集群适应未来增长的一些原因，并提供了一个定制您的 SDN 的示例案例，以降低您的路由网络和 OpenShift 的 SDN 中服务的 IP 冲突风险。

### 信用

特别感谢托马斯·斯托克威尔的同行评议。

### 参考

*   [Red Hat OpenShift 的 SDN 架构概述](https://docs.openshift.com/container-platform/3.11/architecture/networking/sdn.html)
*   [配置红帽 OpenShift 的 SDN](https://docs.openshift.com/container-platform/3.11/install_config/configuring_sdn.html)
*   [配置 docker 的默认桥接网络](https://docs.docker.com/network/bridge/#connect-a-container-to-the-default-bridge-network)
*   [红帽关于修改 docker 网络的建议](https://access.redhat.com/solutions/2942391)
*   [IP 计算器/IP 子网划分(我使用的工具)](http://www.jodies.de/ipcalc)
*   OpenShift 配置参数: [osm_cluster_network_cidr](https://github.com/openshift/openshift-ansible/blob/release-3.11/inventory/hosts.example#L763) ， [openshift_portal_net](https://github.com/openshift/openshift-ansible/blob/release-3.11/inventory/hosts.example#L764) ， [osm_host_subnet_length](https://github.com/openshift/openshift-ansible/blob/release-3.11/inventory/hosts.example#L788) ， [openshift_docker_options](https://github.com/openshift/openshift-ansible/blob/release-3.11/inventory/hosts.example#L130)

*Last updated: July 1, 2020*