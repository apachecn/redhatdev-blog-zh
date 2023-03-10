# Red Hat Data Grid 8.0 带来了新的服务器架构、改进的 REST API 等等

> 原文：<https://developers.redhat.com/blog/2020/04/13/red-hat-data-grid-8-0-brings-new-server-architecture-improved-rest-api-and-more>

[Red Hat 数据网格](https://developers.redhat.com/products/datagrid/overview)帮助应用程序以内存速度访问、处理和分析数据。红帽数据网格 8.0 包含在[红帽运行时](https://www.redhat.com/en/products/runtimes)的最新更新中，提供了一个分布式内存中的 NoSQL 数据存储。这个版本包括一个用于处理复杂应用程序的新的[操作符](https://developers.redhat.com/topics/operators/)，一个减少内存消耗并增加安全性的新服务器架构，一个具有新功能的更快的 API，一个新的 CLI，以及与各种可观察性工具的兼容性。

让我们仔细看看 Data Grid 8.0，看看该工具如何帮助您将传统应用和新型微服务和功能迁移到[开放式混合云](https://www.redhat.com/en/topics/cloud-computing/what-is-hybrid-cloud)。

## 作战情报操作员

红帽数据网格 8.0 引入了一个完全受支持的运算符，它提供了操作智能，使用[Kubernetes](https://developers.redhat.com/topics/kubernetes/)API 来处理操作和管理应用程序生命周期。因此，部署和管理您作为服务使用的复杂应用程序(如分布式数据存储区)比以往任何时候都更容易。它们会自动升级，无需人工干预。

尝试以下文档来创建数据网格运营商订阅，并在 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 中运行数据网格 8.0:[数据网格运营商入门](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.0/html-single/getting_started_with_data_grid_operator/)和[为 OpenShift 运行数据网格](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.0/html-single/running_data_grid_on_openshift/)。

## 新的服务器架构

Data Grid Server 8.0 使用云原生开发最佳实践构建，可提供云就绪功能，同时显著减少占用空间。与以前的版本相比，Data Grid Server 8.0 减少了 50%的磁盘使用和初始堆大小，为您的数据留下了更多的内存。从这个版本开始，数据网格服务器与[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap)(EAP)解耦，简化了配置，减少了漏洞的攻击面。

说到安全性，Data Grid Server 8.0 集成了 Red Hat SSO，并为其他安全机制提供了比以前更强大的支持。您可以在[服务器配置指南](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.0/html/data_grid_server_guide/index)中找到更多关于设置和操作的详细信息。

参见[数据网格服务器入门](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.0/html-single/data_grid_server_guide/#start_server)在五分钟内创建一个运行中的数据网格集群。

## 更快更丰富的 REST API

Red Hat Data Grid 8.0 引入了 REST API v2，它提供了比 v1 快 50%的响应速度，以及以下新功能:

*   访问数据和操作对象，如计数器。
*   使用跨站点复制时，执行诸如正常关闭数据网格集群或将缓存状态转移到备份位置之类的操作。
*   监控集群和服务器健康状况并检索统计数据。

Data Grid 8.0 REST API 还自动在 JSON、XML、Protobuf 和纯文本等存储格式之间进行转换，以提高互操作性。Red Hat 数据网格工程团队开发并维护全面的 REST API 文档。

## 强大的 CLI

在 8.0 版中，Data Grid 提供了一个新的 CLI，其中包含用于远程访问数据和管理集群的直观命令。这个 CLI 使用熟悉的 Bash 命令进行导航，比如`cd`和`ls`。它还提供了命令历史和自动完成功能，以便于使用。此外，CLI 还提供了命令的帮助文本和手册页，并附有清晰的示例。

尝试文档:[数据网格 CLI 入门](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.0/html-single/data_grid_command_line_interface/#getting_started)。

## 增强的可观察性

Red Hat Data Grid 现在与 MicroProfile Metrics API 兼容，并为与 Prometheus 的集成提供了一个`/metrics`端点。要了解更多细节，并了解数据网格指标和直方图，[查看这里的文档](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.0)。

除了指标，Data Grid 8.0 还通过 JMX 提供了改进的统计和管理操作，以及更新的日志记录类别和对 JSON 格式日志的支持。

## 开始

准备好开始尝试数据网格 8.0 了吗？这里有更多有用的链接供您参考:

要了解更多关于 Red Hat 数据网格的信息，请访问[产品页面。](https://www.redhat.com/en/technologies/jboss-middleware/data-grid)

*Last updated: June 29, 2020*