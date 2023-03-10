# 开放自由 20.0.0.5 带来了 EJB 持久计时器协调和跨成员故障转移的更新

> 原文：<https://developers.redhat.com/blog/2020/05/13/open-liberty-20-0-0-5-brings-updates-to-ejb-persistent-timers-coordination-and-failover-across-members>

在[Open Liberty](https://openliberty.io/about)20.0.0.5 中，您现在可以为 Enterprise JavaBeans (EJB)持久计时器配置故障转移，直接从资源适配器加载 Java 身份验证和授权服务(JAAS)类，将您的日志格式化为 JSON 或 dev，并指定哪些 JSON 字段不包含在您的日志中。在本文中，我们将讨论这些特性以及如何实现它们。

查看 [Open Liberty 的 GitHub 问题列表](https://github.com/OpenLiberty/open-liberty/issues?q=label%3Arelease%3A20005+label%3A%22release+bug%22+)中的修复错误列表。

如果你对 Open Liberty 即将推出的内容感兴趣，看看我们当前的开发版本，其中包括带有 Open Liberty 的 GraphQL。

## 使用 20.0.0.5 运行您的应用

如果你使用的是 [Maven](https://openliberty.io//guides/maven-intro.html) ，这里是运行你的应用程序的坐标:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.5</version>
    <type>zip</type>
</dependency>
```

或者对于 [Gradle](https://openliberty.io//guides/gradle-intro.html) :

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.5,)'
}
```

或者，如果您使用 Docker:

```
FROM open-liberty
```

你也可以看看我们的下载页面或者[问一个关于栈溢出的问题](https://stackoverflow.com/tags/open-liberty)。

### 跨成员的 EJB 持久计时器协调和故障转移

通过开放的自由 20.0.0.5，开发者可以为 EJB 持久定时器特性添加一个可配置的属性。新属性设置了在另一个服务器接管并运行计时器之前，持久计时器允许完成的最长时间。

在此功能之前，跨多个 Open Liberty 服务器的自动 EJB 持久计时器的协调仅限于通过配置每个服务器上的 EJB 计时器服务将计时器持久保存到同一个数据库来确保在所有服务器上只创建一个计时器实例。这种设置导致在其中一个服务器上创建了一个计时器实例，但是如果原始服务器停止或崩溃，则无法转到另一个服务器。为了启用故障转移，该特性添加了一个新的可配置属性`missedTaskThreshold`，它指定了在允许另一个服务器接管并运行持久计时器之前，您希望允许持久计时器执行完成的最长时间。

启用 EJB 持久计时器特性——或另一个隐式启用它的特性，如`ejb-3.2`——并将其配置为使用数据源。在这个例子中，我们配置这个特性来使用 Java EE 或 Jakarta EE 默认数据源。无论您是否希望启用故障转移，都需要这么多配置。

将特征添加到`server.xml`:

```
<server>
  <featureManager>
    <feature>ejbPersistentTimer-3.2</feature>
    <feature>jdbc-4.2</feature>
    ... other features
  </featureManager>

  <dataSource id="DefaultDataSource">
    <jdbcDriver libraryRef="OraLib"/>
    <properties.oracle URL="jdbc:oracle:thin:@//localhost:1521/EXAMPLEDB"/>
    <containerAuthData user="dbuser" password="dbpwd"/>
  </dataSource>
  <library id="OraLib">
    <file name="${shared.resource.dir}/jdbc/ojdbc8.jar" />
  </library>

  <!-- The following enables failover for persistent timers -->
  <persistentExecutor id="defaultEJBPersistentTimerExecutor" missedTaskThreshold="5m"/>

  ...
</server>
```

## 从资源适配器加载 JAAS 登录模块

Open Liberty 支持 JAAS 登录模块，用于配置对 J2EE 连接器架构(JCA)管理的资源的访问。一般来说，JAAS 登录模块被打包成一个共享库，并针对 Open Liberty 进行配置。有时，JAAS 登录模块被打包为 JCA 资源适配器的一部分。在过去，要使用这些 JAAS 登录模块，必须从 ResourceAdapter 中提取类并将其配置为共享库。有了这个新特性，`jaasLoginModule`现在可以直接从资源适配器加载类，简化了配置。以前，必须将 JAAS 自定义登录模块打包(或重新打包)到一个单独的共享库中，以配置 Open Liberty 来使用它们。

启用 AppSecurity 和 JCA 功能，并配置资源适配器。使用`jaasLoginModule`元素上的`classProviderRef`属性来引用资源适配器的`id`:

```
<server>
  <featureManager>
    <feature>appSecurity-2.0</feature>
    <feature>jca-1.7</feature>
    ... other features
  </featureManager>

  <resourceAdapter id="eciResourceAdapter" location="${shared.resource.dir}/cicseci.rar"/>

  <!-- classProviderRef indicates that the login module class is found in the resource adapter -->
  <jaasLoginModule id="identityProp" controlFlag="REQUIRED"
      className="com.ibm.ctg.security.idprop.LoginModule"
      classProviderRef="eciResourceAdapter">
    <options propIdentity="Caller"/>
  </jaasLoginModule>

  <jaasLoginContextEntry id="CTGEntry" loginModuleRef="identityProp" name="CTGEntry"/>

  <connectionFactory id="cf1" jndiName="eis/cf1" jaasLoginContextEntryRef="CTGEntry">
    <properties.eciResourceAdapter ConnectionUrl="tcp://localhost" portNumber="2006" serverName="MYSERVER"/>
  </connectionFactory>

  ...
</server>
```

同样的方法也可以用于打包在应用程序中的 JAAS 自定义登录模块。将`classProviderRef`设置为指向包含登录模块类的`application`、`webApplication`或`enterpriseApplication`元素的`id`。将 JAAS 自定义登录模块打包到应用程序中时，请将登录模块包含在以下位置之一:

*   在企业应用程序的顶层 JAR 中。
*   在企业应用程序的资源适配器模块中。
*   在企业应用程序的 web 模块中。
*   在企业应用程序的 EJB 模块中。
*   在 web 应用程序中。

应该注意的是，JAAS 自定义登录模块需要使用带有容器管理的身份验证的资源引用。

你可以找到更多关于[为 Liberty](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_sec_jaas.html) 配置 JAAS 自定义登录模块的信息。

### 打开 Liberty 控制台日志

Open Liberty 控制台日志现在能够用日期、时间戳和其他相关信息格式化日志。通过使用服务器日志配置中的`consoleFormat`日志属性，用户可以将不同的格式(如 JSON 或 dev)应用于出现在他们的`console.log`文件中的服务器日志。dev 格式是默认格式，以基本格式显示消息，没有时间戳或任何其他相关信息。它只显示消息日志级别和消息本身。

例如:

```
consoleFormat=dev (default)
[AUDIT ] CWWKE0001I: The server server1 has been launched.
```

这个特性为`consoleFormat`日志服务器配置属性引入了一个名为`simple`的新选项。这个新选项将 Open Liberty 配置为以与`message.log`文件相同的简单格式将日志输出到`console.log`文件或控制台(`console.log/standard-out`)。

例如:

```
consoleFormat=simple
[25/11/19 10:02:30:080 EST] 00000001 com.ibm.ws.kernel.launch.internal.FrameworkManager A CWWKE0001I: The server server1 has been launched.
```

要配置 Liberty 日志以新的简单控制台格式输出日志，您只需在`server.env`、`bootstrap.properties`或`server.xml`中设置以下日志服务器配置:

#### server.env

`WLP_LOGGING_CONSOLE_FORMAT=simple`

#### bootstrap .属性

`com.ibm.ws.logging.console.format=simple`

#### server.xml

`WLP_LOGGING_CONSOLE_FORMAT=simple`

### 从 JSON 日志记录输出中忽略指定的字段

在 Open Liberty 中，用户可以将他们的服务器日志格式化为 JSON 格式。当日志是 JSON 格式时，用户必须指定他们想要发送到`messages.log`或`console.log/standard-out`的源(消息、跟踪、访问日志、ffdc、审计)。

用户现在可以指定想要省略的 JSON 字段。这个特性为用户添加了一个选项，可以在 JSON 日志记录过程中省略 JSON 字段。在 Open Liberty 中省略 JSON 字段名称的选项非常有用，因为用户可能不希望 Open Liberty 在其 JSON 输出中提供某些默认字段。不需要的字段增加了记录的大小，这在记录传输期间浪费了网络 I/O，并且浪费了下游日志聚合工具中的空间。现在，用户可以选择只发送他们需要的字段，这样他们就可以发送到下游的日志聚合工具，而不会使用不必要的空间和 I/O。例如，在 Docker 容器中运行 Open Liberty(每个容器中只有一个服务器)的人可能不希望包含表示服务器名称和用户目录的 JSON 字段。

该属性最初仅用于重命名字段名称。要重命名 JSON 字段名，格式被指定为`source:defaultFieldName:newFieldName`或`defaultFieldName:newFieldName`。要省略`defaultFieldName`，请将`newFieldName`留空。例如，要省略所有源的字段，请使用`defaultFieldName:`格式。要省略特定源的字段，请使用`source:defaultFieldName:`格式，其中`source`是您想要指定的源，例如消息、跟踪、访问日志、ffdc 或审计。

通过在`bootstrap.properties` :
`com.ibm.ws.logging.json.field.mappings=trace:ibm_userDir: ,ibm_datetime:`中添加以下内容来省略 JSON 字段的示例。

您可以通过查看 IBM 知识中心的[日志和跟踪](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html)或 [Open Liberty 日志文档](https://openliberty.io/docs/ref/config/#logging.html)找到更多信息。

## 现在尝试在红帽运行时打开自由 20.0.0.5

Open Liberty 是 Red Hat Runtimes 产品的一部分。如果你是 Red Hat Runtimes 的用户，你现在可以尝试 Open Liberty。

*Last updated: May 30, 2022*