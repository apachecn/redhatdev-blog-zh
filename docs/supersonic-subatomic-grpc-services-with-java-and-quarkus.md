# Java 和 Quarkus 的超音速亚原子 gRPC 服务

> 原文：<https://developers.redhat.com/blog/2020/12/23/supersonic-subatomic-grpc-services-with-java-and-quarkus>

[gRPC](https://grpc.io) 是一个开源的[远程过程调用](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC)框架。它于 2015 年由谷歌发布，现在是[云本地计算基金会](https://cncf.io)的一个孵化项目。这篇文章介绍了 gRPC，同时解释了它的底层架构以及它与 HTTP 上的 REST 的比较。您还将开始使用 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 来实现和消费 gRPC 服务。

## gRPC 中的远程方法调用

等等，这是什么？你是说远程方法调用吗？这不就是我们在 90 年代用 CORBA、和做的事情吗？

从概念上来说，是的。问题是，“那些老技术与 gRPC 这样的现代框架有什么关系？”

gRPC 类似于 CORBA 和 RMI，因为它们需要生成和使用客户机和服务器绑定。然而，这确实是相似之处的终点。使用的底层传输机制和可用于当今技术框架的工具是主要的区别。

gRPC 使用 [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) 进行传输，使用[协议缓冲区](https://developers.google.com/protocol-buffers/docs/proto3)作为其接口定义语言(IDL)和底层消息交换格式。此外，工具内置于框架中，用于为许多语言和框架生成跨平台的客户端和服务器绑定。

## 比较 gRPC 和 REST

好，所以你说 gRPC 使用 [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) 进行传输？我已经在通过 HTTP 使用 [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) ，所以我为什么要关心 gRPC 呢？

契约实现是这两种技术之间的关键区别。在 REST 中，您的契约(想想 [OpenAPI](https://www.openapis.org/) )更多地关注资源，并使用标准实践，将哪些 HTTP“动词”映射到这些资源上的哪些动作。在大多数情况下，REST 使用 JSON 和 XML 作为请求和响应的主体格式。这些格式被优化为人类可读的，因此需要进行处理以能够被机器读取和产生。

REST 也没有为实现契约提供指导和实现细节。您不需要生成任何代码来实现您的契约。如果您确实想要生成实现契约的客户端或服务器组件的代码，那么使用哪种工具也没有标准化。

与 REST 关注资源相反，gRPC 关注定义服务。这些定义指定了可以在该服务上调用的方法及其参数和返回类型。因此，gRPC 服务是*强类型系统*，这意味着系统两端的契约都是定义良好的。gRPC 还包括现成的客户机和服务器代码生成工具。这使开发人员能够专注于在服务器端构建业务逻辑，并从客户端调用服务。两端通信层的实现是为你生成的。

此外,“跨线”传输被优化为机器可读。它几乎消除了 JSON 和 XML 所需的转换开销。此外，gRPC 提供了开箱即用的身份验证、跟踪和健康监控支持！这使得 gRPC 成为一个非常适合低延迟、高可伸缩的分布式系统的框架。另一方面，gRPC 不太适合不支持 protobuf 这样的二进制协议的系统。

事实上， [Kubernetes](https://kubernetes.io/) 中的许多内部服务和 API 都是用 gRPC 实现的！

## 选择正确的框架

那我们该怎么办？与当今的大多数技术一样，选择一种技术而非另一种技术并不是一个非此即彼的问题。我们需要选择正确的工具来解决正确的问题。许多组织选择将 REST 用于面向公众的应用程序，将 gRPC 用于内部服务对服务的通信。

RESTful 端点[富含超媒体](https://en.wikipedia.org/wiki/HATEOAS)，易于自我发现。当您希望公开服务供其他组织(或组织内的其他业务单位)使用时，这些特性使 REST 成为理想的选择。很容易代理 RESTful 服务并使用 API 管理来控制对它们的访问，实施速率限制、访问控制等策略。

但是在面向公众的服务背后的下游服务网络呢？拥有高吞吐量/低延迟的强类型契约不是很有意义吗？gRPC 在接收数据时比 REST 快 7 倍，在发送数据时比 REST 快 10 倍。

## 在 quartus 中使用 grpc

超音速，亚原子，Java 来拯救。 [Quarkus](https://quarkus.io) 是一个优化的 [Java 栈](https://developers.redhat.com/topics/enterprise-java),由同类最佳的库和标准精心打造，专为 OpenJDK HotSpot 和 GraalVM 定制。对 gRPC 的支持是在版本 1.5 中[引入到 Quarkus 的。正如您将看到的，Quarkus 使得实现和使用 gRPC 服务变得非常容易。](https://quarkus.io/blog/quarkus-grpc/)

### 实施 gRPC 服务

添加 Quarkus gRPC 扩展名(`mvn quarkus:add-extension -Dextensions=”grpc”`)后，您只需要在您的`src/main/proto`目录中定义您的 [protobuf](https://developers.google.com/protocol-buffers/docs/proto3) 文件。一个这样的例子(`helloworld.proto`)可能是这样的:

```
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.quarkus.example";
option java_outer_classname = "HelloWorldProto";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

当您运行`mvn compile`时， [Quarkus maven 插件](https://quarkus.io/guides/maven-tooling)触发 gRPC 生成器生成 gRPC 基本实现类。Quarkus 还为您生成了一个[健康检查服务](https://quarkus.io/guides/grpc-service-implementation#health-check)，这样客户端(比如 Kubernetes 调度程序)就可以检查您的服务是否还在运行。

接下来，您需要决定是否要实现一个[阻塞服务](https://quarkus.io/guides/grpc-service-implementation#implementing-a-service-with-the-default-grpc-api)，一个[反应服务](https://quarkus.io/guides/grpc-service-implementation#implementing-a-service-with-the-mutiny-api)(使用[兵变反应库](https://smallrye.io/smallrye-mutiny/)，或者一个[返回数据流的服务](https://quarkus.io/guides/grpc-service-implementation#handling-streams)。gRPC 提供了所有必要的基本实现类来扩展和实现您的业务功能。在 Quarkus' [开发模式](https://quarkus.io/guides/maven-tooling#development-mode)下运行时，gRPC [反射服务](https://quarkus.io/guides/grpc-service-implementation#reflection-service)自动启用。反射服务使得像 [grpcurl](https://github.com/fullstorydev/grpcurl) 或 [grpcox](https://github.com/gusaul/grpcox) 这样的工具能够与您的服务进行交互，或者让其他客户端服务调用您的服务。

### 消费 gRPC 服务

消费 gRPC 服务类似于实现 gRPC 服务。首先，需要将 gRPC 扩展添加到您的项目中。其次，您需要定义您的 protobuf 文件。编译时， [Quarkus maven 插件](https://quarkus.io/guides/maven-tooling)生成阻塞和反应版本的客户端存根。然后，您可以使用以下方法之一将它们注入到您的应用程序中:

```
@Inject
@GrpcService("hello-service")
MutinyGreeterGrpc.MutinyGreeterStub mutinyHelloService;

@Inject
@GrpcService("hello-service")
GreeterGrpc.GreeterBlockingStub blockingHelloService;

@Inject
@GrpcService("hello-service")
Channel channel;
```

最后，您可以使用客户端存根来调用服务。下面是如何公开调用 gRPC 服务的 REST API:

```
@GET
@Path("/blocking/{name}")
public String helloBlocking(@PathParam("name") String name) {
  return this.blockingHelloService
             .sayHello(
               HelloRequest.newBuilder()
                           .setName(name)
                           .build()
              ).getMessage();
}

@GET
@Path("/mutiny/{name}")
public Uni<String> helloMutiny(@PathParam("name") String name) {
  return this.mutinyHelloService
             .sayHello(
               HelloRequest.newBuilder()
                           .setName(name)
                           .build()
             )
             .onItem()
             .transform(HelloReply::getMessage);
}
```

这并不太难！夸库斯真的让这变得很容易！

## 总结

如果你想看更多这样的例子，可以看看这些展示实际情况的短片。

**如何在 Quarkus 上实现阻塞 gRPC 服务(第一部分)**

[https://www.youtube.com/embed/hecjU-Iixf8?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/hecjU-Iixf8?autoplay=0&start=0&rel=0)

**如何在夸尔库斯兵变(第二部分)中使用反应式 gRPC 服务**

[https://www.youtube.com/embed/KEggeANxGes?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/KEggeANxGes?autoplay=0&start=0&rel=0)

我还推荐[quar kus“Q”提示——探索 Quarkus GRPC 扩展](https://www.youtube.com/watch?v=anGcuMJPkQY)。

想了解更多关于夸尔库斯的信息吗？查看本文,了解 Quarkus 如何比其他 Java 框架节省多达 64%的云资源。

Quarkus 的商业支持是作为红帽运行时间的一部分提供的，同时也是 T2 红帽 OpenShift 订阅的一部分。

## 参考

以下是一些供进一步阅读的参考资料:

*   [在 Quarkus 上开始使用 gRPC](https://quarkus.io/guides/grpc-getting-started)
*   [在 Quarkus 实施 gRPC 服务](https://quarkus.io/guides/grpc-service-implementation)
*   [使用 Quarkus 的 gRPC 服务](https://quarkus.io/guides/grpc-service-consumption)

*Last updated: December 21, 2020*