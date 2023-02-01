# 使用 AMQP 和 Vert.x 的微服务之间的异步通信

> 原文：<https://developers.redhat.com/blog/2018/08/30/microservices-async-communications-amqp-vertx>

[微服务是大多数新型现代软件解决方案的首选架构](https://developers.redhat.com/topics/microservices/)。它们(大部分)被设计来做一件事，并且它们必须相互对话来完成一个业务用例。微服务之间的所有通信都是通过网络调用进行的；这种模式避免了服务之间的紧密耦合，并提供了服务之间更好的分离。

基本上有两种通信方式:同步和异步。正确应用这两种风格是请求-应答和事件驱动模式的基础。在请求-回复模式的情况下，客户端发起请求，并且通常同步等待回复。但是，在有些情况下，客户端可能决定不等待，而是向另一方注册一个回调，这是异步方式的请求-回复模式的一个示例。

在本文中，我通过让两个服务通过高级消息队列协议( [AMQP](https://www.amqp.org/about/what) )相互通信来展示异步请求-应答的方法。AMQP 是在应用程序或组织之间传递业务消息的开放标准。虽然本文关注的是请求-回复模式，但是同样的代码也可以用于开发其他场景，比如[事件源](https://thenewstack.io/microservices-its-all-about-the-events/)。使用异步模型进行通信对于实现[聚合器模式](http://blog.arungupta.me/microservice-design-patterns/)非常有益。

我将使用 [Apache QPid Proton](https://qpid.apache.org) (或[红帽 AMQ 互联](https://developers.redhat.com/products/amq/overview/))作为消息路由器和 Vert.x AMQP 桥，用于两个服务之间的通信。

## 解决方案组件

该演示包括三个部分:

1.  *frontend* :这是一个用 Java 编写的服务，提供一个 HTTP 端点来接收来自客户端的调用。收到请求后，*前端*服务将调用发送到 QPid 路由器，并注册一个回复处理器。当响应可用时，Vert.x AMQP 桥将调用回复处理程序。代码库中的`frontend`文件夹是这个项目的宿主。
2.  QPid 路由器:*前端*进程接收呼叫并向 QPid 队列发送消息。Vert.x 会自动添加一个`correlationId`作为消息属性，以标识对原始请求的响应。
3.  *后端:**后端*组件监听来自 QPid 路由器的呼叫中的消息，对其进行处理(例如，进行计算或保存在数据库中)，并将响应发送回 QPid 路由器。然后，QPid 路由器将用响应通知*前端*组件。代码库中的`backend`文件夹是这个项目的宿主。

## 信息流

跨不同组件的消息的基本流程如下。这个流的完整细节以及相关的标题可以在这里找到。

1.  *前端*服务将向 QPid 服务器发送消息，并提供回复处理程序。Vert.x 会自动填充请求-回复通信所需的标头。
2.  接收应用程序，即*后端*服务，使用该消息并向 QPid 服务器发回回复。Vert.x 填充请求-回复通信所需的必需头。
3.  QPid 服务器将回复消息发送给*前端*服务的回复处理器。Vert.x bridge 自动处理回复处理程序的调用。

克隆这个 [GitHub repo](https://github.com/masoodfaisal/service-comms-amqp-vertx) 以获得示例代码。

## 如何运行示例:快速入门

通过发出以下命令，您可以使用 Docker Compose 文件来运行该示例的所有三个组件:

```
docker-compose up
```

## 如何运行示例:艰难的方式

本节总结了如何单独运行每个组件。你需要以下软件在你的笔记本电脑上运行它们。

1.  [Docker](https://www.docker.com/get-started) (用于执行 Apache Qpid 路由器)
2.  [打开 JDK 8](https://developers.redhat.com/products/openjdk/overview/) (编译*前端*和*后端*服务组件)
3.  [Maven 3.2](https://maven.apache.org/download.cgi) (两个服务都用 Maven)
4.  贝吉塔作为一个 HTTP 客户端(或者你可以使用你最喜欢的工具)

### 执行

*   使用以下命令启动本地 QPid 路由器:

```
docker run -it -p 5672:5672 ceposta/qdr
```

*   编译并执行*前端*服务:

```
cd frontend 
mvn clean install
java -jar target/frontend-service-full.jar
```

*   编译并执行*后端*服务:

```
cd backend 
mvn clean install
java -jar target/backend-service-full.jar
```

## 测试

[贝吉塔](https://github.com/tsenart/vegeta)，一个用于 HTTP 负载测试的开源工具，可以用来向*前端*组件发送请求。

```
echo "GET http://localhost:8080/" | ./vegeta attack -duration=60s -rate=50 | tee results.bin | ./vegeta report
```

## 验证消息数量和延迟

QPid 提供了一个超快速的主干，作为服务间通信的异步集线器。一旦您完成了应用程序的测试，您就可以使用它的`IMAGE ID`登录到 QPid 路由器的 Docker 容器，并运行`qdstat`来查看各种指标。

```
docker exec <container-name> qdstat -c
docker exec <container-name> qdstat -l
docker exec <container-name> qdstat -a

```

## 结论

[Apache QPid](https://qpid.apache.org) 为微服务之间的通信提供了一个超快的主干。由于 AMQP 是一种有线级协议，所以用其他堆栈编写的服务(如 NET)也可以使用相同的通信通道。Java 开发人员可以使用 Vert.x AMQP 桥轻松适应基于 AMQP 的异步服务间通信模式。

*Last updated: January 17, 2022*