# Red Hat 软件集合 3.3 测试版:新增和更新的组件

> 原文：<https://developers.redhat.com/blog/2019/04/17/red-hat-software-collections-3-3-beta-new-and-updated-components>

[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)每年通过两个发布系列为 Red Hat Enterprise Linux 提供最新的稳定版本的开发工具。我们很高兴在此版本中引入三个新组件和两个更新组件，Red Hat Software Collections 3.3 Beta。

新组件包括:

*   Ruby 2.6
*   MariaDB 10.3 具有新的 Maria db Java 连接器
*   Redis 5.0

更新的项目包括:

*   Apache httpd 的两个更新
*   HAProxy 的一个更新

组件详情见下文。

## 新组件

**Ruby 2.6**

此版本引入了这一实验性功能以及许多较小的功能和性能增强:

*   RubyVM::AbstractSyntaxTree 模块。这个模块有一个 parse 方法，它将给定的字符串解析为 Ruby 代码，并返回代码的 AST(抽象语法树)节点。parse_file 方法将以 Ruby 代码的形式打开并解析给定的文件，并返回 AST 节点。

包装名称:rh-ruby26

容器图像:rhscl-beta/ruby-26-rhel7

系统支持:适用于 x86_64、s390x、aarch64、ppc64le 的 RHEL 7

**MariaDB 10.3**

有几十个增强功能，但这里有几个:

*   一个新的*RH-Maria db 103-Maria db-java-client*包，为 MariaDB 和 MySQL 数据库服务器提供 Java 数据库连接(JDBC)连接器。
*   Oracle 兼容性(数据类型、序列和 PL/SQL 语法)使得迁移更加容易。
*   系统版本化的表和时态语法(例如，截至),用于跟踪对数据库每一行的修订。可以为每个表打开此功能，并且可以删除历史记录来管理表的大小。
*   专门构建的存储引擎包括对 MyRocks(针对高写入工作负载)和 Spider(专为扩展而设计)的支持。
*   数据混淆和全部/部分数据屏蔽。
*   即时添加列、隐藏列和压缩列。

包装名称:rh-mariadb103

容器图像:rhscl-beta/mariadb-103-rhel7

系统支持:适用于 x86_64、s390x、aarch64、ppc64le 的 RHEL 7

**第五季**

Redis 是一个开源的内存数据结构存储，用作数据库、缓存和/或消息代理。主要增加的是 Streams——一种新的类似日志的数据结构，用于存储多个字段和带有自动排序的字符串值。

程序包名称:HR-redis 5

容器映像:rhscl beta/redis-5-rhel 7

系统支持:适用于 x86_64、s390x、aarch64、ppc64le 的 RHEL 7

## 更新的组件

**Apache httpd 2.4 更新**

*   Mod_security 是 Apache httpd 的一个模块，它提供了一个开源的 web 应用防火墙。它保护 web 应用程序免受一系列常见的攻击和漏洞，并且是对 web 服务器的一个有价值的补充。
*   mod _ auth _ Mellon rebase—一个带有简单 SAML 2.0 服务提供者的 n Apache 模块。

包装名称:httpd24

容器图像:rhscl-beta/ruby-26-rhel7

系统支持:x86_64、s390x、aarch64、ppc64le 的 RHEL 6 和 7

**HAProxy 1.8 更新**

HAProxy 1.8 版本(现在为 1.8.17)主要关注以下两个方面:

*   性能和应用加速。HAProxy 一直以其可靠性和性能而闻名，但在 HAProxy 1.8 中，我们设法进行了进一步的改进，并包括了一些加速应用程序的功能(如 HTTP/2 支持)。
*   云和微服务。内核设置现在可以在运行时改变，我们已经支持了几种不同的方式(运行时 API，DNS)。

包装名称:rh-haproxy18

系统支持:适用于 x86_64 的 RHEL 7

## 关于软件集合

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)(脚本语言、开源数据库、web 工具等。)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

## 资源

*   [红帽软件集合 3.3 测试版发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3-beta/html-single/3.3_release_notes/index)
*   [截至 3.3 测试版的完整软件集合列表](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3-beta/html-single/3.3_release_notes/index#tabl-RHSCL-Components)

*Last updated: May 1, 2019*