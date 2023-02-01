# 如何在 Red Hat JBoss EAP-7 容器中以主/从方式启动多个 Artemis 代理

> 原文：<https://developers.redhat.com/blog/2017/10/19/start-multiple-artemis-brokers-inside-red-hat-jboss-eap-7-container-masterslave-fashion>

为了尽可能简单，我们将遍历一个独立的用例。

通常，当我们需要在我们的**独立环境**中拥有**消息传递特性**时，我们为 EAP 容器使用**完整概要文件**。

如果我们对**集群功能**有要求，那么我们更喜欢使用**高可用性配置文件**，但是如果同时需要**集群和消息传递，那么我们会选择全高可用性配置文件**。

默认情况下，使用完整/完整 h a 配置文件，EAP-7 容器为我们提供了嵌入式 Artemis 代理的默认配置。但是，在某些场景中，人们可能需要在同一个 EAP-7 容器中添加一个代理。在这种情况下，我**建议为额外的 Artemis Broker** 提供单独的连接器映射。

首先，在 socket-binding-group 中为另一个 Artemis 代理创建 socket-binding 条目:

> ```
> <socket-binding name="http-2" port="${jboss.http.port:8180}" />
> ```

在这里，我创建了 **http-2 作为额外 Artemis** 的套接字绑定。默认情况下，Artemis 使用 http 端口 8080 进行通信。然而，当我们启动 EAP-7 服务器时，它将被占用，因此为了避免冲突，我为容器中的 Artemis 的第二个实例添加了一个额外的套接字。

下一步是**重命名现有的默认 Artemis 服务器/代理，以在多个实例**之间进行识别。我正在尝试设置主从拓扑，因此我将服务器名称设置为“主服务器”。

> ```
> <server name="master">
> ```

然后我**复制同一个服务器，用一个名字粘贴在消息子系统里面作为备份**。

> ```
> <server name="backup">
> 
> <security-setting name="#">
> 
> <role name="guest" send="true" consume="true" create-non-durable-queue="true" delete-non-durable-queue="true"/>
> 
> </security-setting>
> 
> <address-setting name="#" dead-letter-address="jms.queue.DLQ" expiry-address="jms.queue.ExpiryQueue" max-size-bytes="10485760" page-size-bytes="2097152" message-counter-history-day-limit="10"/>
> 
> <http-connector name="http-connector-2" socket-binding="http-2" endpoint="http-acceptor-2"/>
> 
> <http-connector name="http-connector-throughput-2" socket-binding="http" endpoint="http-acceptor-throughput-2">
> 
> <param name="batch-delay" value="50"/>
> 
> </http-connector>
> 
> <in-vm-connector name="in-vm" server-id="1"/>
> 
> <http-acceptor name="http-acceptor-2" http-listener="default"/>
> 
> <http-acceptor name="http-acceptor-throughput-2" http-listener="default">
> 
> <param name="batch-delay" value="50"/>
> 
> <param name="direct-deliver" value="false"/>
> 
> </http-acceptor>
> 
> <in-vm-acceptor name="in-vm" server-id="1"/>
> 
> <jms-queue name="ExpiryQueue-2" entries="java:/jms/queue/ExpiryQueue-2"/>
> 
> <jms-queue name="DLQ-2" entries="java:/jms/queue/DLQ-2"/>
> 
> <connection-factory name="InVmConnectionFactory-2" connectors="in-vm" entries="java:/ConnectionFactory-2"/>
> 
> <connection-factory name="RemoteConnectionFactory-2" connectors="http-connector-2" entries="java:jboss/exported/jms/RemoteConnectionFactory-2"/>
> 
> <pooled-connection-factory name="activemq-ra-2" transaction="xa" connectors="in-vm" entries="java:/JmsXA-2 java:jboss/DefaultJMSConnectionFactory-2"/>
> 
> </server>
> ```

在这一步之后，我认为应该有一个主代理，并执行下面的**命令，使两个代理都处于主从模式**:

> ```
> $ /subsystem=messaging-activemq/server=master/ha-policy=shared-store-master:add(failover-on-server-shutdown=true)
> 
> $ /subsystem=messaging-activemq/server=backup/ha-policy=shared-store-slave:add(allow-failback=true,restart-backup=true,failover-on-server-shutdown=true)
> ```

**消息子系统的整体配置如下:**

> ```
> <subsystem >
>  <server name="master">
>  <shared-store-master failover-on-server-shutdown="true"/>
>  <security-setting name="#">
>  <role name="guest" delete-non-durable-queue="true" create-non-durable-queue="true" consume="true" send="true"/>
>  </security-setting>
>  <address-setting name="#" message-counter-history-day-limit="10" page-size-bytes="2097152" max-size-bytes="10485760" expiry-address="jms.queue.ExpiryQueue" dead-letter-address="jms.queue.DLQ"/>
>  <http-connector name="http-connector" endpoint="http-acceptor" socket-binding="http"/>
>  <http-connector name="http-connector-throughput" endpoint="http-acceptor-throughput" socket-binding="http">
>  <param name="batch-delay" value="50"/>
>  </http-connector>
>  <in-vm-connector name="in-vm" server-id="0"/>
>  <http-acceptor name="http-acceptor" http-listener="default"/>
>  <http-acceptor name="http-acceptor-throughput" http-listener="default">
>  <param name="batch-delay" value="50"/>
>  <param name="direct-deliver" value="false"/>
>  </http-acceptor>
>  <in-vm-acceptor name="in-vm" server-id="0"/>
>  <jms-queue name="ExpiryQueue" entries="java:/jms/queue/ExpiryQueue"/>
>  <jms-queue name="myQueue" entries="java:/jms/queue/myQueue"/>
>  <jms-queue name="DLQ" entries="java:/jms/queue/DLQ"/>
>  <connection-factory name="InVmConnectionFactory" entries="java:/ConnectionFactory" connectors="in-vm"/>
>  <connection-factory name="RemoteConnectionFactory" entries="java:jboss/exported/jms/RemoteConnectionFactory" connectors="http-connector"/>
>  <pooled-connection-factory name="activemq-ra" transaction="xa" entries="java:/JmsXA java:jboss/DefaultJMSConnectionFactory" connectors="in-vm"/>
>  </server>
>  <server name="backup">
>  <shared-store-slave failover-on-server-shutdown="true"/>
>  <security-setting name="#">
>  <role name="guest" delete-non-durable-queue="true" create-non-durable-queue="true" consume="true" send="true"/>
>  </security-setting>
>  <address-setting name="#" message-counter-history-day-limit="10" page-size-bytes="2097152" max-size-bytes="10485760" expiry-address="jms.queue.ExpiryQueue" dead-letter-address="jms.queue.DLQ"/>
>  <http-connector name="http-connector-2" endpoint="http-acceptor-2" socket-binding="http-2"/>
>  <http-connector name="http-connector-throughput-2" endpoint="http-acceptor-throughput-2" socket-binding="http-2">
>  <param name="batch-delay" value="50"/>
>  </http-connector>
>  <in-vm-connector name="in-vm" server-id="1"/>
>  <http-acceptor name="http-acceptor-2" http-listener="default"/>
>  <http-acceptor name="http-acceptor-throughput-2" http-listener="default">
>  <param name="batch-delay" value="50"/>
>  <param name="direct-deliver" value="false"/>
>  </http-acceptor>
>  <in-vm-acceptor name="in-vm" server-id="1"/>
>  <jms-queue name="ExpiryQueue-2" entries="java:/jms/queue/ExpiryQueue-2"/>
>  <jms-queue name="myQueue" entries="java:/jms/queue/myQueue2"/>
>  <jms-queue name="DLQ-2" entries="java:/jms/queue/DLQ-2"/>
>  <connection-factory name="InVmConnectionFactory-2" entries="java:/ConnectionFactory-2" connectors="in-vm"/>
>  <connection-factory name="RemoteConnectionFactory-2" entries="java:jboss/exported/jms/RemoteConnectionFactory-2" connectors="http-connector-2"/>
>  <pooled-connection-factory name="activemq-ra-2" transaction="xa" entries="java:/JmsXA-2 java:jboss/DefaultJMSConnectionFactory-2" connectors="in-vm"/>
>  </server>
>  </subsystem>
> ```

**保存**配置文件**启动红帽 JBoss EAP-7 serve** r

* * *

**点击这里快速上手 JBoss**[**EAP**](https://developers.redhat.com/products/eap/download/?intcmp=7016000000124dvAAA)**下载。**

*Last updated: October 17, 2017*