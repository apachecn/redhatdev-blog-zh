# 将 Oracle 的通用连接池用于 Red Hat JBoss Enterprise Application Platform 7.3 和 Oracle RAC

> 原文：<https://developers.redhat.com/blog/2020/12/07/use-oracles-universal-connection-pool-with-red-hat-jboss-enterprise-application-platform-7-3-and-oracle-rac>

数据是一个关键的业务应用程序组件，但确保一致、可靠的数据访问可能具有挑战性。将分布式服务和高可用性添加到您的应用程序需求中会使数据访问变得更加复杂。您现在可以将 Oracle 的通用连接池(UCP)与 Oracle Real Application Clusters(RAC)和[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/overview)(JBoss EAP)7.3 一起使用。本文介绍了带有 Oracle 通用连接池的连接池，并演示了如何在 JBoss EAP 7.3 部署中将 UCP 与 Oracle RAC 数据库集成。

## 什么是连接池？

一个*连接池*是一组表示物理数据库连接的连接对象。在运行时，应用程序从池中请求一个连接。当应用程序完成连接时，它将连接释放回池中。

打开到数据库的物理连接包括建立 TCP/IP 连接、协商会话参数(来自协议)和认证用户。用户认证可能需要大量的加密密钥生成处理。更糟糕的是，这些步骤中的每一步都需要一个远程过程调用(RPC ),这需要一个到数据库的网络往返行程，并带有隐含的网络延迟。

实现无中断的最大应用正常运行时间需要停机检测、透明的计划维护以及平衡工作负载。所有这些因素都会严重影响应用程序的可用性和性能。连接池是处理这些问题的有效方法。同样重要的是，连接池让我们可以重用连接，而不是在每次发出请求时都创建一个新的连接。

连接池最大限度地减少了客户端和数据库中的资源使用。客户端只需要访问由大量客户端共享的活动连接池。在数据库中，每个活动连接都分配一组资源—内存和 CPU 中的资源—可以通过使用客户机中的池来最小化这些资源。

当使用 [Oracle 通用连接池](https://docs.oracle.com/cd/E11882_01/java.112/e12265/intro.htm#BABHFGCA) (UCP)时， [Java 开发人员](https://developers.redhat.com/topics/enterprise-java)只在逻辑层打开和关闭连接。连接在池中保持活动状态，并根据需要借用和返回池中。

在接下来的部分中，我们将演示在 JBoss EAP 部署中将 UCP 与 Oracle RAC 数据库集成的几种方法。

## 将通用连接池与 Oracle RAC 数据库集成

除了连接池的固有优势之外，通过 Oracle RAC 数据库实施 UCP 的开发人员还可以利用连接池中的其他功能。这些功能解决了高可用性、可扩展性和性能问题，如下所示:

*   **运行时连接负载平衡**允许 UCP 根据集群中节点的负载获取连接。
*   **快速连接故障转移**支持计划外停机，并快速替换池中的死连接。故障转移还支持计划内停机，并有助于确保连接在其工作完成之前不会中断。
*   **事务关联性**允许正在进行的事务附着到集群中的特定节点，从而提高性能。
*   **对数据库驻留连接池和应用程序连续性的内置支持**提供与服务器端池的无缝集成，并屏蔽中断以支持动态事务恢复。
*   **显式请求边界**始于从 UCP 借用连接，止于连接返回连接池。JDBC 驱动程序提供了明确的请求边界声明 API`beginRequest`和`endRequest`。

Oracle UCP 自动配置为在 RAC 实例中监听 Oracle notification service 事件。监听器事件包括:

*   服务开启/关闭
*   节点向上/向下
*   负载平衡和相似性建议

## 使用 JBoss EAP 配置 Oracle UCP

使用 JBoss EAP 配置 Oracle 通用连接池有两种可能的方法。我们将两者都考虑。

### 选项 1:用 web 应用程序描述符(`web.xml`)配置 UCP

这种方法允许您使用一个`web.xml`文件和`<context-param>`名称-值对的标准 web 应用程序描述符来配置连接池。每个名称都使用前缀“`ucp.`”来标识 UCP 参数。下面的`web.xml`设置`jndiName`、`url`、`connectionFactoryClassName`和`dataSourceName`，并标识池中连接的用户:

```
<web-app>

...

<context-param>

<param-name>ucp.jndiName</param-name>

<param-value>java:/datasources/mypool_usingwl</param-value>

</context-param>

<context-param>

<param-name>ucp.url</param-name>

<param-value>jdbc:oracle:thin:@myhost:5521/myservice</param-value>

</context-param>

<context-param>

<param-name>ucp.connectionFactoryClassName</param-name>

<param-value>oracle.jdbc.replay.OracleDataSourceImpl</param-value>

</context-param>

<context-param>

<param-name>ucp.dataSourceName</param-name>

<param-value>myDataSource</param-value>

</context-param>

<context-param>

<param-name>ucp.user</param-name>

<param-value>scott</param-value>

</context-param>

</web-app>

```

注意，`<param-name>`中的属性名应该由前缀`ucp.`和不带前缀`set`的设置器名组成。例如，`ucp.initialPoolSize`映射到`setInitialPoolSize(int)`。

**注意**:关于通用连接池配置值的完整列表，请访问 [Oracle UCP 文档](https://docs.oracle.com/en/database/oracle/oracle-database/19/jjuar/index.html)。

### 选项 2:用 XML 配置文件配置 UCP

要使用这个配置选项，您必须首先将 JVM 系统属性设置为`oracle.ucp.jdbc.xmlConfigFile`。在`web.xml`中，确保`ucp.dataSourceName`参数与来自`/ucp-properties/connection-pool/data-source/data-source-name`的`data-source-attribute`值相匹配。

对于这个`web.xml`:

```
-Doracle.ucp.jdbc.xmlConfigFile=file:/Users/scott/conf/ucp_config.xml

```

传递以下值:

```
<web-app>

...

<context-param>

<param-name>ucp.dataSourceName</param-name>

<param-value>myDataSourceInXml</param-value>

</context-param>

Then /Users/scott/conf/ucp_config.xml

<ucp-properties>

    <connection-pool

    connection-factory-class-name="oracle.jdbc.replay.OracleDataSourceImpl"

    connection-pool-name="pool1"

    initial-pool-size="10"

    max-connections-per-service="15"

    max-pool-size="30"

    min-pool-size="2"

    password="*****"

    url="jdbc:oracle:thin:@myhost:5521/myservice”

    user="scott">

    <connection-property name="autoCommit" value="false"></connection-property>

    <connection-property name="oracle.net.OUTBOUND_CONNECT_TIMEOUT" value="2000">

    </connection-property>

    <data-source data-source-name="myDataSourceInXml" description="pdb1" service="ac">

    </data-source>

    </connection-pool>

</ucp-properties>

```

## 在 JBoss EAP 部署中使用 Oracle UCP

当应用服务器启动或部署时，UCP 会创建一个连接池。当此事件发生时，该对象读取配置或描述文件，并使用配置的值创建 UCP 数据源。然后，它使用 Java 命名和目录接口(JNDI)将数据源绑定到配置的地址。它还将数据源绑定到应用程序范围的对象，该对象是使用上下文和依赖注入(CDI)注入的。包含的对象可以使用 JNDI 地址和应用程序范围的对象来检索数据源及其连接。应用程序范围的对象和 JNDI 检索的数据源在同一个池中，因此这种机制不会产生重复。

在 JBoss EAP 部署中使用 UCP 有两种可能的方法。

### 选项 1:使用上下文和依赖注入(CDI)

首先，这里有一个为此目的使用 CDI 的示例:

```
@Inject

@UCPResource

private DataSource ds;

```

在这种情况下，数据源就可以使用了。不需要额外的代码。

### 选项 2:使用 Java 命名和目录接口(JNDI)

第二种选择是使用 Java 命名和目录接口。要使用 JNDI，您需要执行额外的步骤来检索池并将其关联到数据源:

```
public class OracleUcp extends HttpServlet { // sample usage in a servlet

  private DataSource ds = null;

  // Retrieve Datasource reference using JNDI

  @Override

  public void init() throws ServletException { // association must occur after init

    Context initContext;

    try {

      initContext = new InitialContext();

      ds = (DataSource) initContext.lookup("java:/datasources/mypool_usingwl");

```

## 结论

我们希望本文中的讨论和示例能够帮助您理解如何在 JBoss EAP 部署中使用 Oracle UCP 和 Oracle RAC。结合使用这些技术是开发弹性的、高度可用的应用程序的好方法。我们推荐以下额外的资源来深入了解 JBoss EAP:

*   访问[红帽开发者 JBoss EAP 主页](https://developers.redhat.com/products/eap/overview)了解 JBoss 企业应用平台概况。
*   请参见 *[在 JBoss 7.0 EAP](https://blogs.oracle.com/dev2dev/using-universal-connection-pool-ucp-as-a-pool-datasource-in-jboss-70-eap)* 中使用通用连接池(UCP)作为池数据源(Pablo Silberkasten，2017 年 3 月)，了解有关在 JBoss EAP 7.0 中使用 UCP 的更多信息。
*   参见*[Red Hat JBoss EAP](https://developers.redhat.com/blog/2020/04/10/jboss-eap-7-3-brings-new-packaging-capabilities/)*的新打包功能(James Falkner，2020 年 4 月)了解 JBoss EAP 7.3 中的新打包功能。
*   *[用红帽 JBoss EAP 开发 Microprofile 应用](https://developers.redhat.com/blog/2020/07/01/develop-eclipse-microprofile-applications-on-red-hat-jboss-enterprise-application-platform-expansion-pack-1-0-with-red-hat-codeready-workspaces/)* (Emmanuel Hugonnet，2020 年 7 月)提供了更多用 JBoss EAP 开发应用的例子。

*Last updated: December 2, 2020*