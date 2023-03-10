# Red Hat Fuse 简化了消息代理集成

> 原文：<https://developers.redhat.com/blog/2021/01/08/message-broker-integration-made-simple-with-red-hat-fuse>

本文展示了[红帽 AMQ 7](https://developers.redhat.com/products/amq/overview) 和 [IBM MQ](https://www.ibm.com/products/mq) 之间的示例集成，使用[红帽 Fuse 7](https://developers.redhat.com/products/fuse/download) 进行集成。传统上，开发人员使用资源适配器与外部系统进行消息桥接。*资源适配器*是一个系统库，提供到企业信息系统(EIS)的连接。类似于 Java 数据库连接(JDBC)驱动程序如何提供到数据库管理系统的连接，资源适配器插入到应用服务器，例如[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/download)(JBoss EAP)。然后，它连接应用服务器、企业信息系统和企业应用程序。

资源适配器工作得很好，但是配置可能会让人不知所措，特别是对于需要在应用服务器上添加大量模块的场景，或者资源适配器契约需要大量管理资源的场景。有些场景需要配置多个资源适配器，然后安排它们之间的消息交换。

Red Hat Fuse 为组件之间的消息路由提供了一种更简单、更灵活的模式。Fuse 利用 Apache Camel 的 [JMS 组件](https://camel.apache.org/components/latest/jms-component.html)通过 Java 消息服务(JMS)队列或主题交换消息。它依赖于 Spring 对声明性事务的 JMS 支持。

按照下一节中的演示，自己看看 JMS 组件如何在 Red Hat Fuse 集成中工作。

## 建立红帽保险丝项目

在这个演示中，我们将创建一个基于 Spring Boot T2 的 Fuse 应用程序，它使用骆驼路线在红帽 AMQ 和 IBM MQ 之间交换消息。首先，打开一个 shell 提示符并输入下面的`mvn`命令:

```
mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate \
-DarchetypeCatalog=https://maven.repository.redhat.com/ga/io/fabric8/archetypes/archetypes-catalog/2.2.0.fuse-760024-redhat-00001/archetypes-catalog-2.2.0.fuse-760024-redhat-00001-archetype-catalog.xml \
-DarchetypeGroupId=org.jboss.fuse.fis.archetypes \
-DarchetypeArtifactId=spring-boot-camel-xml-archetype \
-DarchetypeVersion=2.2.0.fuse-760024-redhat-00001

```

该命令创建基本的项目结构，包括`Application`类:

```
package com.sample;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}

```

到目前为止，代码应该看起来很熟悉。

## 配置代理

接下来，我们将配置两个代理。首先添加一个包含以下`JmsComponent` bean 声明的`@Configuration` bean(注意，使用`id`的方法名将声明绑定为`@Bean`):

```
package com.sample;

import com.ibm.mq.jms.*;
import org.apache.activemq.artemis.jms.client.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.UserCredentialsConnectionFactoryAdapter;

@Configuration
public class AppConfig {

    @Bean
    public JmsComponent activemq() throws Exception {
        // Create the connectionfactory which will be used to connect to Artemis
        ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory();
        cf.setBrokerURL("tcp://localhost:61616");
        cf.setUser("admin");
        cf.setPassword("admin");

        // Create the Camel JMS component and wire it to our Artemis connectionfactory
        JmsComponent jms = new JmsComponent();
        jms.setConnectionFactory(cf);
        return jms;
    }

    @Bean
    public JmsComponent wmq(){
        JmsComponent jmsComponent = new JmsComponent();
        jmsComponent.setConnectionFactory(mqQueueConnectionFactory());
        return jmsComponent;
    }

    @Bean
    public MQQueueConnectionFactory mqQueueConnectionFactory() {
    // Create the connectionfactory which will be used to connect to IBM MQ
        MQQueueConnectionFactory mqQueueConnectionFactory = new MQQueueConnectionFactory();
        mqQueueConnectionFactory.setHostName("localhost");
        try {
            mqQueueConnectionFactory.setTransportType(1);
            mqQueueConnectionFactory.setChannel("DEV.APP.SVRCONN");
            mqQueueConnectionFactory.setPort(1414);
            mqQueueConnectionFactory.setQueueManager("QM1");
        } catch (Exception e) {
           e.printStackTrace();
        }
        return mqQueueConnectionFactory;
    }

    @Bean
    public UserCredentialsConnectionFactoryAdapter userCredentialsConnectionFactoryAdapter(
            MQQueueConnectionFactory mqQueueConnectionFactory) {
        UserCredentialsConnectionFactoryAdapter userCredentialsConnectionFactoryAdapter = new UserCredentialsConnectionFactoryAdapter();
        userCredentialsConnectionFactoryAdapter.setUsername("username");
        userCredentialsConnectionFactoryAdapter.setPassword("password");
        userCredentialsConnectionFactoryAdapter.setTargetConnectionFactory(mqQueueConnectionFactory);
        return userCredentialsConnectionFactoryAdapter;
    }

}

```

红帽 AMQ 连接工厂现在在`activemq` `id`下可用。IBM MQ 连接工厂在`wmq`T3 下可用。

## 添加路线定义

我们快完成了。我们的最后一步是添加包含路由定义的`RouteBuilder`类:

```
package com.sample;

import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Component
public class CamelArtemisRouteBuilder extends RouteBuilder {

    public void configure() throws Exception {

        from("timer:mytimer?period=5000").routeId("generate-route")
                .transform(constant("HELLO from Camel!"))
                .to("activemq:queue:QueueIN");

        from("activemq:queue:QueueIN").routeId("receive-route")
                .log("Received a message - ${body} - sending to outbound queue")
                .to("wmq:queue:DEV.QUEUE.1?exchangePattern=InOnly");

    }
}

```

在上面的路由中，消息是由每五秒钟触发一次新消息的计时器组件生成的。消息通过名为`QueueIN`的红帽 AMQ 队列发送。在记录消息体之后，它们被发送到 sink 目的地，也就是`DEV.QUEUE.1`。

我们已经完成了集成设置。接下来，我们将运行应用程序并验证它是否工作正常。

## 启动代理并运行应用程序

在运行应用程序之前，我们需要启动两个代理实例。确保您的 Red Hat AMQ 服务器已启动并正在运行。如果它尚未运行，请输入以下命令:

```
./artemis run

```

接下来，启动 IBM MQ。为此，您可以使用带有开发人员许可证的 IBM MQ 的容器映像:

```
$ podman run --env LICENSE=accept --env MQ_QMGR_NAME=QM1 --publish 1414:1414 --publish 9443:9443 --detach ibmcom/mq

```

最后，使用标准的 Spring Boot Maven 插件启动 Red Hat Fuse 应用程序:

```
$ mvn spring-boot:run

```

## 验证应用程序

当应用程序启动时，您将从 Spring Boot 控制台看到控制台上记录的消息:

```
12:10:12.552 [Camel (camel-1) thread #1 - JmsConsumer[INCOMING]] INFO receive-route - Received a message - HELLO from Camel! - sending to outbound queue
13:20:37.935 [Camel (camel-1) thread #1 - JmsConsumer[INCOMING]] INFO receive-route - Received a message - HELLO from Camel! - sending to outbound queue
13:20:42.954 [Camel (camel-1) thread #1 - JmsConsumer[INCOMING]] INFO receive-route - Received a message - HELLO from Camel! - sending to outbound queue
13:20:47.966 [Camel (camel-1) thread #1 - JmsConsumer[INCOMING]] INFO receive-route - Received a message - HELLO from Camel! - sending to outbound queue

```

您可以登录到正在运行的容器进程，检查消息是否已经在 IBM MQ 端收到:

```
$ podman ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
52f0f72d56ed docker.io/ibmcom/mq:latest 5 hours ago Up 5 hours ago 0.0.0.0:1414->1414/tcp pensive_bose
```

输出显示消息已收到:

```
$ podman exec --tty --interactive 52f0f72d56ed bash bash-4.4$ /opt/mqm/samp/bin/amqsbcg DEV.QUEUE.1 QM1

MQGET of message number 1
****Message descriptor****

StrucId : 'MD ' Version : 2
Report : 0 MsgType : 8
Expiry : -1 Feedback : 0
Encoding : 546 CodedCharSetId : 850
Format : 'MQEVENT '
Priority : 0 Persistence : 0
MsgId : X'414D512073617475726E2E71756575650005D30033563DB8'
CorrelId : X'000000000000000000000000000000000000000000000000'
BackoutCount : 0
ReplyToQ : ' '
** Identity Context
UserIdentifier : ' '
AccountingToken :
X'0000000000000000000000000000000000000000000000000000000000000000'
ApplIdentityData : ' '
** Origin Context
PutApplType : '7'
PutDate : '19970417' PutTime : '15115208'
ApplOriginData : ' '
GroupId : X'000000000000000000000000000000000000000000000000'
MsgSeqNumber : '1'
Offset : '0'
MsgFlags : '0'
OriginalLength : '104'

**** Message ****

length - 104 bytes

```

## 结论

本文讨论了如何通过将 JMS 组件注册为 Spring beans 来连接多个 JMS 提供者(在本例中，是 Red Hat AMQ 和 IBM MQ)。在注册了`ConnectionFactory`bean 之后，我们将它们注入到 Camel JMS 组件配置中。然后，我们定义了 Camel JMS 组件，并将其作为 Camel route 的一部分，从应用程序的两端移动消息。

若要开始使用 Red Hat Fuse，[请访问 Red Hat Fuse 下载页面](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=jboss.fuse&downloadType=distributions)。要了解关于使用 Red Hat Fuse 进行消息代理集成的更多信息，请参见[将 IBM WebSphere MQ 与 JBoss Fuse 集成](https://access.redhat.com/solutions/1173833)。

## 关于作者

Francesco Marchioni 是位于意大利罗马的 Red Hat 中间件产品高级技术客户经理。在[mastertheboss.com](http://www.mastertheboss.com)阅读更多关于开源对 JBoss 开发者社区的贡献。