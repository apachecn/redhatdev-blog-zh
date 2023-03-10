# A-MQ 7 测试版组件

> 原文：<https://developers.redhat.com/articles/amq7beta>

# A-MQ 组件

A-MQ 是一套服务器和客户机，它们一起工作，形成一个可靠的分布式应用程序的基础。

## A-MQ 代理

A-MQ 代理是一个纯 Java 的多协议消息代理。它构建在高效的异步核心之上，具有用于消息持久性的快速本地日志和用于高可用性的无共享状态复制选项。

*   **持久性** -一个快速的本地 IO 日志或一个基于 JDBC 的商店

*   **高可用性** -共享存储或无共享状态复制

*   **高级排队** -末值队列、消息组、主题层次结构和大消息支持

*   **多协议** - AMQP 1.0、MQTT、STOMP、OpenWire 和 HornetQ 核心

*   **集成** -与 Red Hat JBoss EAP 完全集成

A-MQ 代理基于 [Apache ActiveMQ Artemis](https://activemq.apache.org/artemis/) 项目。

## A-MQ 互连

A-MQ 互连是一个高速、低延迟的 AMQP 1.0 消息路由器。您可以部署多个 A-MQ 互连路由器来构建一个容错消息传递网络，将您的客户端和代理连接到一个无缝结构中。

*   **灾难恢复** -跨地域部署冗余网络路由器

*   **高级路由** -控制网络上消息的分发和处理

*   **集成** -连接客户端、代理和独立服务

*   **管理** -简化的管理让大型部署变得切实可行

A-MQ 互连基于 [Apache Qpid Dispatch](https://qpid.apache.org/components/dispatch-router/index.html) 项目。

## A-MQ 客户端

A-MQ Clients 是一套 AMQP 1.0 消息传递 API，允许您将任何应用程序变成消息传递应用程序。它既包括 JMS 之类的行业标准 API，也包括新的事件驱动 API，这使得在任何地方集成消息传递都很容易。

*   **新的事件驱动 API**-快速、高效的消息传递，可在任何地方集成

*   **行业标准 API**-JMS 1.1 和 2.0

*   **广泛的语言支持** - C++，Java，JavaScript，Python，和。网

*   **广泛的可用性** -基于 Linux、Windows 和 JVM 的环境

A-MQ 客户端基于以下项目。

*   [阿帕奇 Qpid JMS](https://qpid.apache.org/components/jms/index.html)

*   [阿帕奇 Qpid 质子](https://qpid.apache.org/proton/index.html)

*   [蔚蓝 AMQP.Net Lite](https://github.com/Azure/amqpnetlite)

## A-MQ 控制台

A-MQ 控制台是 A-MQ 部署的中心控制点。它是一个基于 web 的管理控制台，能够实时监控和修改部署。

A-MQ 控制台基于 [JBoss Hawtio](http://hawt.io/) 项目。

*Last updated: November 19, 2020*