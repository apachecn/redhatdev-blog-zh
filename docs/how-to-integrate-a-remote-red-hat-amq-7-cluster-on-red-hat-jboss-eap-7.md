# 如何在红帽 JBoss EAP 7 上集成远程红帽 AMQ 7 集群

> 原文：<https://developers.redhat.com/blog/2018/12/06/how-to-integrate-a-remote-red-hat-amq-7-cluster-on-red-hat-jboss-eap-7>

在集成环境中，使用消息系统连接不同的组件是很常见的，比如红帽 AMQ 7(RHAMQ 7)。在这种情况下，通常有 JEE 应用服务器，如[Red Hat JBoss Enterprise Application Platform 7](https://developers.redhat.com/products/eap/overview/)(JBoss EAP 7)，用于部署和运行连接到消息传递系统的应用程序。

本文详细描述了如何在 JBoss EAP 7 服务器上集成远程 RHAMQ 7 集群，并详细介绍了不同的配置和组件，以及一些改进消息驱动 bean(MDB)应用程序的技巧。

## 概观

JBoss EAP 7 中嵌入的消息传递代理是 ActiveMQ Artemis，这是一个由 HornetQ 和 Apache ActiveMQ 项目联合创建的社区项目，它是 RHAMQ 7 的基础。这种结合的结果是一个适用于两个平台的高性能消息传递系统，具有两个项目的最佳特性和一些智能的新特性。

JBoss EAP 7 以一个嵌入式 Apache ActiveMQ Artemis 服务器作为其 JMS 代理来提供 Java EE 消息传递功能，它在`messaging-activemq`子系统中进行配置。这个子系统定义了这个嵌入式代理的操作和功能。

然而，将 RHAMQ 7 部署为外部集群消息传递平台以用于不同类型的应用程序是非常常见的。JBoss EAP 7 使用 Java 连接器架构(JCA)资源适配器连接到任何消息传递提供者。JBoss EAP 7 包括一个集成的 Artemis 资源适配器。

集成的 Artemis 资源适配器可以配置为连接到 RHAMQ 7 的远程安装，然后成为 JBoss EAP 7 应用程序的 JMS 提供者。这允许 JBoss EAP 7 成为远程 RHAMQ 7 服务器的客户机。

与 JBoss EAP 7 集成的 Artemis 资源适配器有以下限制:

*   **队列和主题的动态创建:**它不*不*支持在 RHAMQ 7 代理中动态创建队列和主题。您必须直接在远程 RHAMQ 7 代理上配置所有队列和主题目的地。
*   **连接工厂的创建:** RHAMQ 7 允许使用`pooled-connection-factory`和`external-context`来配置连接工厂；每个连接工厂的创建方式都有所不同。只有`pooled-connection-factory`可以用来在 JBoss EAP 7 中创建连接工厂。`external-context`只能用于将已经在远程 RHAMQ 7 代理上配置好的 JMS 目的地注册到 JBoss EAP 7 服务器的 JNDI 树中，以便本地部署可以查找或注入它们。当连接到远程 RHAMQ 7 代理时，只支持使用通过配置`pooled-connection-factory`元素创建的连接工厂。

JBoss EAP 7 和 RHAMQ 7 之间的通信需要您设置:

*   RHAMQ 7 经纪人
*   JBoss EAP 7 服务器

本文假设 RHAMQ 7 HA 集群至少部署了三个主代理。描述如何部署高可用性(HA) RHAMQ 7 集群不在本文的讨论范围之内；然而，如果你想了解更多，自动化 AMQ 7 高可用性部署文章可以帮助你。

## RHAMQ 7 配置

JBoss EAP 7.1 中包含的 Artemis 资源适配器使用 ActiveMQ Artemis JMS Client 1.5.5。该客户端要求在地址前加上`anycastPrefix`和`multicastPrefix`。它还希望队列名与地址名相同。

```
<acceptors>
     <acceptor name="netty-acceptor">tcp://localhost:61616?anycastPrefix=jms.queue.;multicastPrefix=jms.topic.</acceptor>
</acceptors>

```

## JBoss EAP 7 配置

JBoss EAP 7 包括一个默认配置，用于带有`full`或`full-ha`配置的`messaging-activemq`子系统。默认配置不包括如何连接到远程服务器。因此，要设置它，请使用以下步骤:

*   定义远程套接字绑定，以连接到部署在其集群中的每个 RHAMQ 7 代理:

```
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=messaging-remote-broker01:add(host=BROKER01,port=61616)
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=messaging-remote-broker02:add(host=BROKER02,port=61616)
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=messaging-remote-broker03:add(host=BROKER03,port=61616)
```

*   在`messaging-activemq`子系统中定义新的远程连接器:

```
/subsystem=messaging-activemq/server=default/remote-connector=messaging-remote-broker01-connector:add(socket-binding=messaging-remote-broker01)
/subsystem=messaging-activemq/server=default/remote-connector=messaging-remote-broker02-connector:add(socket-binding=messaging-remote-broker02)
/subsystem=messaging-activemq/server=default/remote-connector=messaging-remote-broker03-connector:add(socket-binding=messaging-remote-broker03)
```

*   定义一个名为`activemq-rar.rar`的新的池连接工厂。请注意，我们使用的是连接器列表，并且我们激活了 HA 属性:

```
/subsystem=messaging-activemq/server=default/pooled-connection-factory=activemq-rar.rar:add(entries=["java:/RemoteJmsXA", "java:jboss/RemoteJmsXA"],connectors=["messaging-remote-broker01-connector", "messaging-remote-broker02-connector", "messaging-remote-broker03-connector"],ha=true)
```

*   定义一些额外的属性，例如，每个连接器之间的`user`、`password`和`rebalance-connections`(这样可以避免只连接到集群的一个成员):

```
/subsystem=messaging-activemq/server=default/pooled-connection-factory=activemq-rar.rar:write-attribute(name=user,value=user)
/subsystem=messaging-activemq/server=default/pooled-connection-factory=activemq-rar.rar:write-attribute(name=password,value=s3cr3t)
/subsystem=messaging-activemq/server=default/pooled-connection-factory=activemq-rar.rar:write-attribute(name=rebalance-connections,value=true)
```

*   定义一个新的外部上下文来声明 RHAMQ 7 集群中的队列和主题。此步骤将定义一个本地 JNDI 条目来连接到远程资源:

```
/subsystem=naming/binding=java\:global\/remoteContext:add(binding-type=external-context, class=javax.naming.InitialContext, module=org.apache.activemq.artemis, environment=[java.naming.factory.initial=org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory, queue.SampleQueue=SampleQueue, topic.SampleTopic=SampleTopic])
/subsystem=naming/binding=java\:\/queue\/SampleQueue:add(lookup=java:global/remoteContext/SampleQueue,binding-type=lookup)
/subsystem=naming/binding=java\:\/topic\/SampleTopic:add(lookup=java:global/remoteContext/SampleTopic,binding-type=lookup)
```

*   作为一个可选步骤，您可以将 EJB 3 子系统中定义的默认资源适配器从默认的(`activemq-ra`)修改为新定义的 *(* `activemq-rar.rar` *)。*通过这一更改，您的所有 MDB 实例都将连接到使用它的代理:

```
<mdb>
    <resource-adapter-ref resource-adapter-name="${ejb.resource-adapter-name:activemq-rar.rar}"/>
    <bean-instance-pool-ref pool-name="mdb-strict-max-pool"/>
</mdb>
```

*   新的池连接工厂禁用了统计信息；如果您想要监视它在运行时的工作方式，请执行以下操作来激活它们:

```
/subsystem=messaging-activemq/server=default/pooled-connection-factory=activemq-rar.rar:write-attribute(name=statistics-enabled,value=true)
```

*   执行以下操作查看统计数据:

```
/subsystem=messaging-activemq/server=default/pooled-connection-factory=activemq-rar.rar:read-resource(include-runtime=true)
```

## 改进 MDB 应用程序的技巧

一些文献描述了在最新的应用程序设计中使用 MDB 作为消费消息的反模式；但是，MDB 经常在 Java EE 应用程序中使用。在 JBoss EAP 7 环境中可以改进 MDB 的使用，并且——使用 RHAMQ 7 这样的高性能代理——您可以通过遵循一些提示获得高水平的吞吐量。

MDB 是一种特殊的无状态会话 beans。它们实现了一个名为`onMessage`的方法，当 MDB 监听的 JMS 目的地接收到一条消息时，就会触发这个方法。也就是说，MDB 是在收到来自 JMS 提供者(资源适配器)的消息时触发的，这与无状态会话 beanss 不同，在无状态会话 bean 中，方法通常由 EJB 客户端调用。MDB 异步处理消息。

但是，实例化的 MDB 的数量可以由以下内容定义:

*   资源适配器定义
*   Bean 实例池定义
*   资源适配器的池定义
*   资源适配器特定的属性

这些特性的组合将帮助您定义 JBoss EAP 7 将管理多少个 MDB 实例，从而提高您的消息传递系统的吞吐量。

### 资源适配器定义

在某些情况下，我们需要定义一些不同的池连接工厂。在这种情况下，我们可以通过使用`@org.jboss.ejb3.annotation.ResourceAdapter`注释在 MDB 中定义使用哪个资源适配器:

```
@ResourceAdapter("activemq-rar.rar")
@MessageDriven(
    name = "SampleMDB",
    mappedName = "queue/SampleQueue",
    activationConfig = {
        @ActivationConfigProperty(propertyName = "destination", propertyValue = "SampleQueue"),
        @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "javax.jms.Queue"),
        @ActivationConfigProperty(propertyName = "acknowledgeMode", propertyValue = "Auto-acknowledge")
    }
)
public class SampleMDB implements MessageListener {
```

这个注释是由 Maven `jboss-ejb3-ext-api`工件提供的，可以添加到您的 Maven 项目中，如下所示:

```
<!-- EJB3 Extension -->
<dependency>
    <groupId>org.jboss.ejb3</groupId>
    <artifactId>jboss-ejb3-ext-api</artifactId>
    <version>2.2.0.Final-redhat-1</version>
    <scope>provided</scope>
</dependency>
```

### Bean 实例池定义

JBoss EAP 7 可以在 bean 实例池中缓存 EJB 实例，以节省初始化时间。MDB 实例位于名为`mdb-strict-max-pool`的默认池定义中。如果您有大量的 MDB 定义，或者您想将它们分成不同的池，您可以创建不同的 bean 实例池，如下所示:

```
/subsystem=ejb3/strict-max-bean-instance-pool=mdb-sample-pool:add(max-pool-size=100,timeout=10,timeout-unit=MILLISECONDS)
```

### 资源适配器的池定义

您可以通过使用`@org.jboss.ejb3.annotation.Pool`注释:来设置特定 bean 将使用的特定实例池

```
@Pool("mdb-sample-pool") 
@MessageDriven(
    name = "SampleMDB",
    mappedName = "queue/SampleQueue",
    activationConfig = {
        @ActivationConfigProperty(propertyName = "destination", propertyValue = "SampleQueue"),
        @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "javax.jms.Queue"),
        @ActivationConfigProperty(propertyName = "acknowledgeMode", propertyValue = "Auto-acknowledge")
    }
)
public class SampleMDB implements MessageListener {
```

### 资源适配器特定的属性

默认情况下，在 JBoss EAP 7 中，每个 MDB 最多可以有 16 个会话，每个会话处理一条消息。您可以通过使用特定于资源适配器的属性来更改该值，以符合应用程序的要求。

`maxSession`是定义由 MDB 管理的不同数量的会话的属性。当增加 MDB 上的`maxSession`属性的大小时，确保该值不大于 MDB 池大小中的`max-pool-size`值是很重要的。如果是，将会有空闲会话，因为没有足够的 MDB 来服务它们。建议两个值相等。

MDB 的定义应该如下所示:

```
@Pool("mdb-sample-pool") 
@MessageDriven(
    name = "SampleMDB",
    mappedName = "queue/SampleQueue",
    activationConfig = {
        @ActivationConfigProperty(propertyName = "destination", propertyValue = "SampleQueue"),
        @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "javax.jms.Queue"),
        @ActivationConfigProperty(propertyName = "acknowledgeMode", propertyValue = "Auto-acknowledge"),
        @ActivationConfigProperty(propertyName = "maxSession", propertyValue = "100")
    }
)
public class SampleMDB implements MessageListener {
```

### 监控您的 MDB

要确认 MDB 的状态，此 CLI 命令将帮助您:

```
/deployment=SampleMDB.jar/subsystem=ejb3/message-driven-bean=SampleMDB:read-resource(include-runtime=true)

```

## 摘要

本文详细描述了 JBoss EAP 7 与 RHAMQ 7 集群代理和子系统配置的集成，并提供了一些关于如何改进 MDB 应用程序的额外技巧。

如果您想了解更多详情，请参考以下链接:

*   [配置消息:配置 Artemis 资源适配器以连接到红帽 AMQ 7](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/configuring_messaging/resource_adapters#using_jboss_amq_for_remote_jms_communication)
*   [配置消息传递:JMS 调优](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/configuring_messaging/#tuning_jms)
*   [JBoss EAP 7 性能调优指南:EJB 子系统调优](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/performance_tuning_guide/ejb_subsystem_tuning)

*Last updated: September 3, 2019*