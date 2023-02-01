# 使用 jmxtrans 代理监控 Red Hat AMQ 7

> 原文：<https://developers.redhat.com/blog/2018/06/06/monitoring-red-hat-amq-7-with-the-jmxtrans-agent>

## 监控红帽 AMQ 7

红帽 AMQ 7 包括一些监控红帽 AMQ 经纪人的工具。这些工具允许您获得关于代理及其资源的性能和行为的指标。度量对于测量性能和识别导致低性能的问题非常重要。

以下组件用于监控红帽 AMQ 7 代理:

*   基于 Hawtio 的管理 web 控制台:这个控制台包括一些透视图和仪表板，用于监控代理的最重要的组件。
*   一个类似 Jolokia REST 的 API:它通过 HTTP 请求提供对 JMX bean 的完全访问。
*   Red Hat JBoss Operation Network:这是一个基于 Java 的企业管理平台，用于开发、测试、部署和监控 Red Hat JBoss 中间件应用程序。

这些工具是不可思议的，并与原始产品完全集成。然而，在某些情况下，Red Hat AMQ 7 部署在使用其他工具监控代理的环境中，例如`jmxtrans`。

[jmxtrans](http://www.jmxtrans.org/) is a popular monitoring tool for Java-based applications. It connects to a JVM via JMX, collects metrics, and sends the data to the back end of your choice. Included in `jmxtrans` is an agent to be deployed inside the application that is to be monitored. This blog shows how to integrate the [jmxtrans agent](https://github.com/jmxtrans/jmxtrans-agent) with Red Hat AMQ 7.

## 设置红帽 AMQ 7

首先，你需要获得最新稳定版本的`jmxtrans`代理。可以从以下地方获得:

*   [从 GitHub 库发布](https://github.com/jmxtrans/jmxtrans-agent/releases)
*   [Maven 中央储存库](http://central.maven.org/maven2/org/jmxtrans/agent/jmxtrans-agent/)

将`jmxtrans-agent` JAR 文件复制到`$ARTEMIS_INSTANCE/lib`文件夹中。

为了在引导时加载代理，您需要修改`$ARTEMIS_INSTANCE/bin`文件夹中的`artemis`文件。它定义了一个环境变量(`AMQ_JMSTRANS_AGENT`)来更新代理的启动命令行。该环境变量允许您配置何时使用`jmxtrans`代理:

```
# Add jmxtrans agent
if [ "$AMQ_JMXTRANX_AGENT" = "true" ]; then
  echo "Using jmxtrans agent to collect metrics. Configuration loaded from $ARTEMIS_INSTANCE/etc/jmxtrans-agent.xml"
  JMXTRANX_OPTS="-Xbootclasspath/p:$ARTEMIS_INSTANCE/etc -javaagent:$ARTEMIS_INSTANCE/lib/jmxtrans-agent.jar=$ARTEMIS_INSTANCE/etc/jmxtrans-agent.xml"
fi
```

接下来，您应该使用在`artemis`文件中定义的启动命令行来加载这个配置，如下所示:

```
exec "$JAVACMD" \
 $JAVA_ARGS \
 $JMXTRANX_OPTS -Xbootclasspath/a:"$LOG_MANAGER" \
 -Djava.security.auth.login.config="$ARTEMIS_INSTANCE/etc/login.config" \
 $ARTEMIS_CLUSTER_PROPS \
 -classpath "$CLASSPATH" \
 -Dartemis.home="$ARTEMIS_HOME" \
 -Dartemis.instance="$ARTEMIS_INSTANCE" \
 -Djava.library.path="$ARTEMIS_HOME/bin/lib/linux-$(uname -m)" \
 -Djava.io.tmpdir="$ARTEMIS_INSTANCE/tmp" \
 -Ddata.dir="$ARTEMIS_DATA_DIR" \
 -Djava.util.logging.manager="$ARTEMIS_LOG_MANAGER" \
 -Dlogging.configuration="$ARTEMIS_LOGGING_CONF" \
 $DEBUG_ARGS \
 org.apache.activemq.artemis.boot.Artemis "$@"
```

### 定义查询和指标

`jmstrans`代理允许您定义不同的查询来从 JMX bean 收集指标。这些查询在位于`$ARTEMIS_INSTANCE/etc`文件夹中的`jmxtrans-agent.xml`文件中定义。这些查询将定义要收集的 JMX bean 和属性。下面显示了一个从地址和队列获取指标的示例。

```
<queries>
  <!-- Addresses -->
  <query objectName="org.apache.activemq.artemis:broker=*,component=addresses,address=*"
         attributes="MessageCount,NumberOfMessages,NumberOfPages,QueueNames"
         resultAlias="service=amq7,host=#hostname#,broker=%broker%,address=%address%,attribute.#attribute#"/>

  <!-- Queues -->
  <query objectName="org.apache.activemq.artemis:broker=*,component=addresses,address=*,subcomponent=queues,routing-type=*,queue=*"
         attributes="ConsumerCount,MaxConsumers,DeliveringCount,MessageCount,MessagesAcknowledged,MessagesAdded,MessagesExpired,MessagesKilled"
         resultAlias="service=amq7,host=#hostname#,broker=%broker%,address=%address%,queue=%queue%,artemis.addresses.#attribute#"/>
</queries>
```

在 JMX 树中获取其他指标的其他查询也是有效的。

## 从控制台获取指标

`jmxtrans`包括几个输出编写器，用于将收集到的指标发送给外部服务，例如，作为 Graphite、CSV 文件和 InfluxDB。输出写入器在`jmxtrans-agent.xml`文件中定义如下:

```
 <!-- Output Writers -->
 <outputWriter class="org.jmxtrans.agent.ConsoleOutputWriter" />
```

这个块定义了控制台输出编写器，因此度量将在服务器控制台日志中显示为消息。

如果您现在启动红帽 AMQ 7，输出将类似于此:

```
$ export AMQ_JMXTRANX_AGENT=true
$ ./bin/artemis run
Using jmxtrans agent to collect metrics. Configuration loaded from /opt/amq/brokers/broker01-master/etc/jmxtrans-agent.xml
2018-06-04 17:14:39.103 INFO [main] org.jmxtrans.agent.JmxTransAgent - Starting 'JMX metrics exporter agent: 1.2.8' with configuration '/opt/amq/brokers/broker01-master/etc/jmxtrans-agent.xml'...
2018-06-04 17:14:39.113 INFO [main] org.jmxtrans.agent.JmxTransAgent - PropertiesLoader: Empty Properties Loader
2018-06-04 17:14:39.445 INFO [main] org.jmxtrans.agent.JmxTransExporter - Configuration reload interval: 10secs
2018-06-04 17:14:39.445 INFO [main] org.jmxtrans.agent.JmxTransAgent - JmxTransAgent started with configuration '/opt/amq/brokers/broker01-master/etc/jmxtrans-agent.xml'
 __ __ ____ ____ _
 /\ | \/ |/ __ \ | _ \ | |
 / \ | \ / | | | | | |_) |_ __ ___ | | _____ _ __
 / /\ \ | |\/| | | | | | _ <| '__/ _ \| |/ / _ \ '__|
 / ____ \| | | | |__| | | |_) | | | (_) | < __/ |
 /_/ \_\_| |_|\___\_\ |____/|_| \___/|_|\_\___|_|

Red Hat JBoss AMQ 7.1.0.GA
2018-06-04 17:14:40,370 INFO [org.apache.activemq.artemis.integration.bootstrap] AMQ101000: Starting ActiveMQ Artemis Server
...
```

几秒钟后，您可以看到`jmxtrans`代理正在收集指标并将它们导出到控制台(使用`ConsoleOutputWriter`):

```
2018-06-04 17:14:43,740 INFO [org.apache.activemq.artemis] AMQ241001: HTTP Server started at http://localhost:8161
2018-06-04 17:14:43,740 INFO [org.apache.activemq.artemis] AMQ241002: Artemis Jolokia REST API available at http://localhost:8161/console/jolokia
2018-06-04 17:14:43,741 INFO [org.apache.activemq.artemis] AMQ241004: Artemis Console available at http://localhost:8161/console
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,attribute.MessageCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,attribute.NumberOfMessages 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,attribute.NumberOfPages 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,attribute.QueueNames DLQ 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.ConsumerCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.MaxConsumers -1 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.DeliveringCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.MessageCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.MessagesAcknowledged 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.MessagesAdded 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.MessagesExpired 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=ExpiryQueue,queue=ExpiryQueue,artemis.addresses.MessagesKilled 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.ConsumerCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.MaxConsumers -1 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.DeliveringCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.MessageCount 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.MessagesAcknowledged 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.MessagesAdded 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.MessagesExpired 0 1528125286
service=amq7,host=localhost,broker=0_0_0_0,address=DLQ,queue=DLQ,artemis.addresses.MessagesKilled 0 1528125286
```

### jmxtrans-agent.xml 文件示例

下面显示了用于红帽 AMQ 7 的`jmxtrans`代理的完整样本文件:

```
<jmxtrans-agent>
  <queries>
    <!-- ActiveMQ Artemis -->
    <!-- Addresses -->
    <query objectName="org.apache.activemq.artemis:broker=*,component=addresses,address=*"
           attributes="MessageCount,NumberOfMessages,NumberOfPages,QueueNames"
           resultAlias="service=amq7,host=#hostname#,broker=%broker%,address=%address%,attribute.#attribute#"/>

    <!-- Queues -->
    <query objectName="org.apache.activemq.artemis:broker=*,component=addresses,address=*,subcomponent=queues,routing-type=*,queue=*"
           attributes="ConsumerCount,MaxConsumers,DeliveringCount,MessageCount,MessagesAcknowledged,MessagesAdded,MessagesExpired,MessagesKilled"
           resultAlias="service=amq7,host=#hostname#,broker=%broker%,address=%address%,queue=%queue%,artemis.addresses.#attribute#"/>

  </queries>

  <!-- Output Writers -->
  <outputWriter class="org.jmxtrans.agent.ConsoleOutputWriter" />

  <!-- Other Properties -->
  <collectIntervalInSeconds>5</collectIntervalInSeconds>
  <reloadConfigurationCheckIntervalInSeconds>60</reloadConfigurationCheckIntervalInSeconds>
</jmxtrans-agent>
```

## 参考

关于如何使用`jmxtrans`监控红帽 AMQ 7 的更多细节可在此获得:

*   [监控您的 AMQ 部署](https://access.redhat.com/documentation/en-us/red_hat_amq/7.1/html-single/using_amq_console/#monitoring_your_amq_deployment)
*   [使用 JMX 管理经纪人](https://access.redhat.com/documentation/en-us/red_hat_amq/7.1/html-single/using_amq_broker/index#managing_the_broker_using_jmx)
*   [利用乔恩和 AMQ 经纪人](https://access.redhat.com/documentation/en-us/red_hat_amq/7.1/html-single/using_jon_with_amq_broker/)
*   [jmxtrans Wiki](https://github.com/jmxtrans/jmxtrans/wiki)
*   [jmxtrans 代理 Wiki](https://github.com/jmxtrans/jmxtrans-agent/wiki)

*Last updated: June 5, 2018*