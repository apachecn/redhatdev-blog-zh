# Red Hat JBoss 数据虚拟化的新版本。

> 原文：<https://developers.redhat.com/blog/2018/02/19/red-hat-jboss-data-virtualization-6-4>

Red Hat 自豪地宣布 JBoss 数据虚拟化(JDV) 6.4 的发布

### 概观

JBoss 数据虚拟化是一个数据集成解决方案，它位于多个数据源的前面，允许将它们视为一个单一的数据源，在正确的时间以所需的形式向任何应用程序和/或用户提供正确的数据。

### JDV 6.4 功能

JBoss 数据虚拟化 6.4 版本侧重于支持新的和更新现有的云、大数据和内存数据源。

### JDV 6.4 中增加了以下新功能和数据源集成:

OSISoft PI 是一个实时数据历史应用程序，具有高效的时间序列数据库。客户使用 PI 系统来记录、分析和监控实时信息，例如从原材料到最终产品的制造流程和产品谱系。OSIsoft PI 是作为 JDV 6.3 的技术预览版发布的，JDV 6.4 完全支持它。

Couchbase Server 是一个开源的、分布式的面向 NoSQL 文档的数据库，它针对可能服务于许多并发用户的交互式应用程序进行了优化。Couchbase 在 OpenShift 上作为开发者预览版提供。

**亚马逊关系数据库服务(RDS)** 使得在云中建立、操作和扩展关系数据库变得容易。它提供可调整的容量，并自动执行管理任务，包括数据库设置、修补和备份。JDV 6.4 支持以下 Amazon RDS 数据库引擎:PostgreSQL、MySQL、MariaDB、Oracle 和 MS SQL Server。

**亚马逊 S3** (简单存储服务)是亚马逊 web 服务(AWS)提供的 web 服务，通过 Web 服务接口(REST、SOAP 和 BitTorrent)提供存储。JDV 公开了存储过程，通过利用 JDV 的 Web 服务资源适配器来利用 S3 对象资源。这通常用于访问存储在亚马逊 S3 上的 CSV 或 XML 格式的数据或其他对象文件。

**Presto** 是一个开源的分布式 SQL 查询引擎，用于针对从千兆字节到千兆字节的所有大小的数据源运行交互式分析查询。Presto 是为交互式分析而设计的，接近商业数据仓库的速度，同时可以扩展到脸书、Airbnb 和网飞等组织的规模。

**JDG 集成**改进:JBoss 数据网格(JDG)是为弹性可伸缩性和快速访问大量数据而设计的。在 JDV 6.3 中，我们引入了对 JDG 作为物化目标的支持(除了以前可用的 JDG 作为数据源)。JDG 为 JDV 中的外部物化视图提供了可伸缩、快速和一致的性能。在 JDV 6.4 中，我们显著简化了 JDG 作为物化目标和数据源的配置，使得这个重要的集成更加容易。

### 工具改进

JDV 6.4 与 JBDSIS 11.2.0.GA 和 JBDS 11.2.0.GA 一起发布(Eclipse Oxygen)

对于 JDV 6.4，我们在几个方面改进了 Teiid Designer 的可用性，包括:

*   将数据源连接管理整合到一个视图中
*   VDB 编辑简化
*   对 JDG 7.1 集成的用户界面支持

**数据服务构建器** (DSB，技术预览版)是一个基于网络的用户界面，允许用户轻松地创建新的数据服务，并在同一工具中测试它们。该工具旨在为新用户和开发人员提供一种工具，直观地创建简单的数据服务并对其进行查询。该工具将是 JDV 6.4 的技术预览版。注意，除了 DSB 之外，基于 Eclipse 的 Teiid 设计器仍然完全受支持。

### 即将推出…

**OpenShift**–JDV 6.4 正在 open shift 上获得支持，将于 2018 年 3 月向客户提供。OpenShift 上的 JDV 将允许客户通过 RESTful OData API 轻松地将本地和云中的多个异构数据源作为数据服务公开。请注意，数据服务构建器 WebUI 将在 OpenShift 上作为 JDV 6.4 的开发人员预览版提供，因此开发人员可以轻松创建数据服务并使用相同的工具进行测试。此次在 OpenShift 上发布的 JDV 6.4 也将包括 JDG 的简化配置，作为 JDV 的物化目标。

### 额外资源

**JDV 6.4 GA 可从**下载

-[-](https://developers.redhat.com/products/datavirt/hello-world/)

- [客户门户](https://access.redhat.com/products/red-hat-jboss-data-virtualization/)

[**JDV 6.4 文档**](https://access.redhat.com/documentation/en-us/red_hat_jboss_data_virtualization/)

[**JDV 6.4 支持的配置**](https://access.redhat.com/articles/703663)

[**Redhat.com 上的 JDV**上的 ](https://www.redhat.com/en/technologies/jboss-middleware/data-virtualization)

*Last updated: November 15, 2018*