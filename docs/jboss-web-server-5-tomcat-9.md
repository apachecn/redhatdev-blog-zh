# 等待结束了:带有 Tomcat 9 的 JBoss Web Server 5 终于来了！

> 原文：<https://developers.redhat.com/blog/2018/07/19/jboss-web-server-5-tomcat-9>

我们很高兴地宣布面向 Red Hat Enterprise Linux (RHEL)的 [Red Hat JBoss Web Server 5.0](https://developers.redhat.com/products/webserver/overview/) 正式发布。其他平台也将很快发布。该版本包括通过 Narayana 进行交易处理的技术预览。JBoss Web Server 5 可以从 JBoss Web Server 5.0 Maven 存储库和容器目录中以 ZIP 或 RPM 格式获得。

JBoss Web Server 将市场领先的开源技术与企业功能相结合，为大型网站和轻量级 Web 应用程序提供单一解决方案。它结合了世界上部署最多的 web 服务器(Apache)和顶级的 servlet 引擎(Tomcat)以及中间件中最好的支持(来自 Red Hat)。

### 主要组件

*   #### Apache Tomcat 9

    *   符合 Java servlet 规范的 Servlet 容器。JBoss Web 服务器包含 Apache Tomcat 9
*   #### 阿帕奇雄猫本地库

    *   Tomcat 库，它改进了 Tomcat 的可伸缩性、性能以及与本机服务器技术的集成
*   #### Tomcat Vault

    *   JBoss Web 服务器的扩展，用于安全存储 JBoss Web 服务器使用的密码和其他敏感信息
*   #### 模 _ 簇库

    *   一个允许 Apache Tomcat 和 Apache HTTP 服务器的 mod_proxy_cluster 模块之间通信的库。这允许 Apache HTTP 服务器用作 JBoss Web 服务器的负载平衡器

### **新特性和增强功能**

#### JBoss Web Server 5 基于全新的 Tomcat 9.0.7。特色亮点:

- HTTP/2 支持
- Servlet 4.0 规范
-带 JSSE 连接器(NIO 和 NIO2)的 TLS OpenSSL
-NIO 连接器，安装`tomcat-native`时默认为 HTTP/1.1
-TLS 虚拟主机(SNI)支持

#### 它还具有以下增强功能:

-支持嵌入式发行版(fat JAR 部署)。
-提供异步 NIO2 支持。
-通过 Narayana 和 DBCP2(技术预览)提供交易处理。
-为从安装的 Red Hat Enterprise Linux 用户提供系统守护程序集成脚本。zip 存档。
-`tomcat-vault`安装过程已改进。
-`tomcat-vault`的`vault.properties`文件可以存储在`JWS_HOME`之外。
-管理器和主机管理器 web 应用程序行为发生了变化。
—`mod_cluster`1.4 有变化:需要指定连接器。
-可以配置多个属性文件。
-不推荐使用`log4j`登录 JBoss Web 服务器。
-嵌入式 Tomcat 服务器包含在 JBoss Web Server 5.0 Maven 存储库中。

### 介绍 Narayana(技术预览)

JBoss Web 服务器上的 Narayana 目前处于技术预览状态，目前还没有官方附带文档。

Narayana 事务工具包为使用以下基于标准的事务协议开发的应用程序提供支持:

*   JTA
*   JTS
*   网络服务交易
*   剩余交易记录
*   扫描隧行显微镜
*   XATMI/TX

JBoss Web Server 上的 Narayana 正在开发中，用于与 Red Hat Process Automation Manager 集成。JBoss Web Server 上的 Narayana 还使用 Tomcat JDBC 连接池提供与 PostgreSQL 服务器的连接。

更多信息请查看 JBoss Web 服务器文档网站

*Last updated: November 15, 2018*