# 红帽开源数据库内测:增加了 PostgreSQL 10，MongoDB 3.6 更新 MySQL 5.7

> 原文：<https://developers.redhat.com/blog/2018/04/06/red-hat-open-source-data-bases-beta-adds-postgresql-10-mongodb-3-6-updates-mysql-5-7>

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)(脚本语言、开源数据库、web 工具等。)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

**Red Hat Software Collections 3.1 beta 带来了以下新的/更新的开源数据库:**

## 新增: **PostgreSQL 10**

PostgreSQL 是一个强大的开源对象关系数据库系统，拥有超过 15 年的积极开发和成熟的架构，在可靠性、数据完整性和正确性方面赢得了良好的声誉。以下是 PostgreSQL 10 中的新增功能:

*   逻辑复制——用于分发数据的发布/订阅框架
*   声明式表分区——方便划分数据
*   改进的查询并行性——快速征服您的分析
*   同步复制的仲裁提交-放心分发数据
*   SCRAM-SHA-256 认证-保护您的数据访问

此版本还标志着 PostgreSQL 的版本化方案变为“x.y”格式。这意味着 PostgreSQL 的下一个次要版本将是 10.1，下一个主要版本将是 11。

有哪些版本，在哪里有？

*   RHSCL 包括 PostgreSQL 9.2、9.4、9.5 和 9.6
*   RHEL 6 包括 PostgreSQL 8.4
*   RHEL7 包括 PostgreSQL 9.2

RHEL 开发人员和用户拥有 PostgreSQL 的最新稳定版本，用于需要数据库的应用程序开发。

PostgreSQL 10 版本包括显著的增强功能，可有效地实现跨多个节点分布数据的能力，从而实现更快的访问、管理和分析，包括本机逻辑复制、声明性表分区和改进的查询并行性。

仅用于 RHEL7

包:rh-postgresql10

Linux 容器映像:rhscl-beta/postgresql-10-rhel7

## 新增内容:MongoDB3.6

MongoDB 3.6 是面向现代应用程序的领先数据库的最新版本，是本机数据库功能和增强的顶点，将允许您轻松地发展您的解决方案来解决新出现的挑战和用例。

允许开发人员持久化丰富的嵌套数据而不使其扁平化是 MongoDB 的优势之一。文档可以建模任何类型的数据:键值、图形和关系数据集在文档中就像异构的嵌套结构一样常见。MongoDB Server 3.6 使查询语言更加强大，新的数组更新操作符允许您在任何嵌套深度指定对特定的匹配数组项的就地更新。对$lookup 聚合阶段的扩展现在允许不相关的子查询和多个匹配条件，因此可以在数据库中处理复杂组合中的引用和连接文档。

现代应用程序需要即时响应变化，向用户和实时更新的界面提供通知。为了实现这一点，MongoDB 3.6 引入了变更流，应用程序可以使用它来获得更新的实时通知，以收集数据。

健壮系统的一个关键特征是它们可以优雅地处理网络中断，但是处理这些中断的防御性编码可能会给开发人员带来很大的负担。MongoDB 3.6 通过可重试的写操作减轻了这一负担，这一新特性确保写操作只执行一次，即使在停机时也是如此。

从这个版本开始，MongoDB 服务器本身将默认拒绝所有连接，除非它们来自白名单中的 IP。
文档的灵活性与数据验证完全兼容，MongoDB 3.6 通过引入 JSON Schema 改进了之前的功能。使用 JSON Schema，您可以(在 JSON 中)在每个集合的基础上，准确地指定什么符合有效文档的条件，例如字段可以具有的类型，它是否是必需的，以及文档是否允许规范中没有列出的字段。在 MongoDB 3.6 中，模式不是一件紧身衣，而是一个验证框架，您可以根据自己的需要进行调整。

去年推出的 BI 连接器已经针对 MongoDB 3.6 进行了完全重写，使其运行速度更快，更易于管理。BI connector 2.0 在将 SQL 查询翻译成 MongoDB 的原生聚合框架方面做得更好，因此它可以将更多的工作直接推送到 MongoDB，而不必在内存中自己完成。

有哪些版本，在哪里？

*   RHSCL 包括 MongoDB 2.4、2.6、3.2、3.4 和现在的 3.6。
*   MongoDB 不包含在任何版本的 RHEL 中。

仅用于 RHEL7

包装名称:rh-mongodb36

Linux 容器映像:rhscl-beta/mongodb-36-rhel7

## **更新:MySQL 5.7 适用于 ppc64le、s390x 和 aarch64**

MySQL 5.7 是世界上最流行的开源数据库的最新版本。新版本提供了更高的性能、可伸缩性和可管理性，并通过 JSON 支持和 MySQL 路由器增强了 NoSQL 功能，从而可以轻松地将应用程序连接到多个 MySQL 数据库。这个版本的 Red Hat 软件集合使得 MySQL 5.7 集合可用于 ppc64le、s390x 和 aarch64 架构。此前，MySQL 5.7 仅适用于英特尔 x86_64。

有哪些版本，在哪里？

*   RHSCL 包括 MySQL 5.5、5.6 和 5.7
*   RHEL6 包括 MySQL 5.1
*   RHEL7 不包含 MySQL。

仅用于 RHEL7

包装名称:rh-mysql57

## 参考资料:

*   参见 [Hello World](https://developers.redhat.com/products/softwarecollections/hello-world/) 快速安装软件集合。
*   [RHSCL 3.1 beta 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3-beta/html/3.1_release_notes/)
*   [使用容器映像的 RHSCL 3.1 beta】](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3-beta/html/using_red_hat_software_collections_container_images/)
*   [红帽集装箱目录](https://access.redhat.com/containers/)

*Last updated: November 15, 2018*