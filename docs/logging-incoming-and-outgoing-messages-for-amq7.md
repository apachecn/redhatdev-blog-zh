# 记录红帽 AMQ 7 的传入和传出消息

> 原文：<https://developers.redhat.com/blog/2018/10/26/logging-incoming-and-outgoing-messages-for-amq7>

在这篇文章中，我将讨论如何为[红帽 AMQ 7](https://developers.redhat.com/products/amq/overview/) (RHAMQ 7)捕获传入和传出的消息。如果您需要记录传入或传出的流量，或者来自代理的消息，或者在开发和/或测试期间，当您想要查看所有消息时，这可能是有利的。此外，还可能需要修改传输中的消息。使用 RHAMQ 7 拦截器，您可以拦截进出 RHAMQ 7 代理的流量。您还可以使用拦截器修改消息。

在我的[个人 GitHub 页面](https://github.com/1984shekhar/Artemis_POC/tree/master/AMQ_Interceptor/InterceptorArtemis_Core)上，有一个使用拦截器的例子，它与 RHAMQ 7 的核心协议一起工作。

创建拦截器的第一步是实现`Interceptor`接口。

```
package org.apache.artemis.activemq.api.core.interceptor;

public interface Interceptor
{
boolean intercept(Packet packet, RemotingConnection connection) throws ActiveMQException;
}

```

在上面提到的 GitHub 页面中，有一个简单的 Java 类`SimpleInterceptor.java`，它实现了`Interceptor`接口，并使用以下代码拦截消息:

```
public boolean intercept(final Packet packet, final RemotingConnection connection) throws ActiveMQException {

if (packet instanceof SessionSendMessage) {
System.out.println("SimpleInterceptor gets called!.... Packet: " + packet.getClass().getName() + "RemotingConnection: " + connection.getRemoteAddress() );
SessionSendMessage realPacket = (SessionSendMessage) packet;
Message msg = realPacket.getMessage();
if((msg.getTimestamp()>0) && msg.getUserID()!=null)
System.out.println("Msg: "+msg.toString());
}
else if (packet instanceof SessionReceiveMessage) {
System.out.println("SimpleInterceptor gets called!.... Packet: " + packet.getClass().getName() + "RemotingConnection: " + connection.getRemoteAddress() );
SessionReceiveMessage realPacket = (SessionReceiveMessage) packet;
Message msg = realPacket.getMessage();
if((msg.getTimestamp()>0) && msg.getUserID()!=null)
System.out.println("Msg: "+msg.toString());
}

return true;
}

```

需要注意的是，当数据包是`SessionSendMessage`的实例时，它是一个输入消息。当一个包是`SessionReceiveMessage`的实例时，它是一个传出消息，允许消费者/订户进一步处理它。

通过返回`true`，我们调用下一个拦截器(如果有的话)或目标。如果我们返回`false`，进一步的处理将被中止，因此既不会调用下一个拦截器，也不会向任何目标发送消息。

使用`mvn package` 构建项目，并将 JAR 复制到位置`amq_broker_home/lib`。

在代理配置文件`broker.xml`中，我们必须以如下方式为传入和传出消息指定这个拦截器:

```
<core...>
<remoting-incoming-interceptors>
   <class-name>com.mycompany.interceptor.SimpleInterceptor</class-name>
</remoting-incoming-interceptors>
<remoting-outgoing-interceptors>
   <class-name>com.mycompany.interceptor.SimpleInterceptor</class-name>
</remoting-outgoing-interceptors>
</core>
```

此时，我们可以启动 RHAMQ 7 代理开始测试。从目录`amq_broker_home/bin`运行以下命令:

```
./artemis consumer --url tcp://localhost:61616 --user admin --password admin --destination queue://TESTCP --verbose
./artemis producer --url tcp://localhost:61616 --user admin --password admin --destination queue://TESTCP --message-count 1

```

您应该会看到以下结果:

```
#For Sender
SimpleInterceptor gets called!.... Packet: org.apache.activemq.artemis.core.protocol.core.impl.wireformat.SessionSendMessageRemotingConnection: /127.0.0.1:49896 Msg: CoreMessage[messageID=0,durable=true,userID=4e094a2a-d6e0-11e8-a444-e8b1fc466329,priority=4, timestamp=Tue Oct 23 21:55:50 IST 2018,expiration=0, durable=true, address=exampleQueue,size=270,properties=TypedProperties[__AMQ_CID=4df09207-d6e0-11e8-a444-e8b1fc466329,_AMQ_ROUTING_TYPE=1]]@183519025

#For Receiver
SimpleInterceptor gets called!.... Packet: org.apache.activemq.artemis.core.protocol.core.impl.wireformat.SessionReceiveMessageRemotingConnection: /127.0.0.1:49896 Msg: CoreMessage[messageID=3333,durable=true,userID=4e094a2a-d6e0-11e8-a444-e8b1fc466329,priority=4, timestamp=Tue Oct 23 21:55:50 IST 2018,expiration=0, durable=true, address=exampleQueue,size=270,properties=TypedProperties[__AMQ_CID=4df09207-d6e0-11e8-a444-e8b1fc466329,_AMQ_ROUTING_TYPE=1]]@1835190256

```

我们还可以从位置`amq_broker_home/bin`使用以下命令来获取目的地(队列/主题)统计信息:

```
/artemis queue stat --url tcp://localhost:61621 --user admin --password admin --queueName TESTCP --verbose

```

如前所述，这个拦截器将只记录核心协议中的消息。对于 Stomp 协议，我们的拦截器应该实现接口`StompFrameInterceptor`:

```
package org.apache.activemq.artemis.core.protocol.stomp;

public interface StompFrameInterceptor extends BaseInterceptor
{
   boolean intercept(StompFrame stompFrame, RemotingConnection connection);
}

```

类似地，对于 MQTT 协议，拦截器应该实现接口`MQTTInterceptor`:

```
package org.apache.activemq.artemis.core.protocol.mqtt;

public interface MQTTInterceptor extends BaseInterceptor
{
    boolean intercept(MqttMessage mqttMessage, RemotingConnection connection);
}

```

就是这样！我希望本文能帮助您设置 RHAMQ 7 拦截器，记录或修改传入或传出的消息。

## 额外资源

以下是 Red Hat 开发者博客上的相关文章:

*   [使用 AMQP 和 Vert.x 的微服务之间的异步通信](https://developers.redhat.com/blog/2018/08/30/microservices-async-communications-amqp-vertx/)
*   [使用 jmxtrans 代理监控红帽 AMQ 7](https://developers.redhat.com/blog/2018/06/06/monitoring-red-hat-amq-7-with-the-jmxtrans-agent/)
*   [如何为红帽 AMQ 7 消息代理控制台设置 LDAP 认证](https://developers.redhat.com/blog/2018/09/21/setup-ldap-auth-amq-console/)
*   [在红帽 AMQ 经纪公司设立 RBAC](https://developers.redhat.com/blog/2018/08/06/setting-up-rbac-on-red-hat-amq-broker/)
*   [如何在红帽 JBoss EAP 7 上集成 A-MQ 6.3](https://developers.redhat.com/blog/2018/08/20/how-to-integrate-a-mq-6-3-on-red-hat-jboss-eap-7/)

*Last updated: September 3, 2019*