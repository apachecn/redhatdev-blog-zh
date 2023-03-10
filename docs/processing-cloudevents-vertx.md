# 用 Eclipse Vert.x 处理云事件

> 原文：<https://developers.redhat.com/blog/2018/12/11/processing-cloudevents-vertx>

我们的互联世界充满了由不同软件服务触发或接收的事件。其中一个大问题是，事件发布者倾向于以不同的方式描述事件，而且大多数情况下都是互不兼容的。

为了解决这个问题，来自[云本地计算基金会](https://www.cncf.io/) (CNCF)的[无服务器工作组](https://github.com/cncf/wg-serverless)最近宣布了[云事件](https://cloudevents.io/)规范的 0.2 版本。该规范旨在以通用、标准化的方式描述事件数据。在某种程度上，CloudEvent 是一个抽象的信封，带有一些描述具体事件及其数据的指定属性。

使用 CloudEvents 很简单。本文展示了如何使用 Vert.x 提供的强大的 JVM 工具包来生成或接收和处理 CloudEvents。

## 用于处理 CloudEvents 的 SDK

除了规范，来自无服务器工作组的 CloudEvents 团队正在为各种平台开发不同的 SDK，如 [JavaScript](https://github.com/cloudevents/sdk-javascript) 、 [Golang](https://github.com/cloudevents/sdk-go) 、 [C Sharp](https://github.com/cloudevents/sdk-csharp) 、 [Java](https://github.com/cloudevents/sdk-java) 和 [Python](https://github.com/cloudevents/sdk-python) 。本文将简要概述 Java SDK，以及如何在用 Eclipse Vert.x 构建的应用程序中使用它。

该 API 非常简单，包含一个通用的`CloudEvent`类以及一个创建`CloudEvent`实例的构建器:

```
final CloudEvent<MyCustomEvent> cloudEvent = new CloudEventBuilder<MyCustomEvent>()
    .data(new MyCustomEvent(...))
    .type("My.Cloud.Event.Type")
    .id(UUID.randomUUID().toString();)
    .source(URI.create("/trigger");)
    .build();
```

上面，我们使用`CloudEventBuilder`创建了一个非常简单的`CloudEvent`实例。但是，孤立来看，API 并没有显示出它的实力。

## Eclipse 绿色. x

Eclipse Vert.x 是一个在 JVM 上构建反应式应用程序的工具包。它是事件驱动和非阻塞的，这意味着应用程序可以使用少量的内核线程处理大量的并发性。有关 Vert.x. Fortun 的更多信息，请参阅下面的参考资料

幸运的是，CloudEvents Java SDK 中包含了对 Eclipse Vert.x 的支持:

```
<dependency>
    <groupId>io.cloudevents</groupId>
    <artifactId>http-vertx</artifactId>
    <version>0.2.0</version>
</dependency>
```

### 向远程服务发送云事件

现在我们有了我们的`CloudEvent`对象，捕获了我们的事件数据，我们想把它发送到远程云服务，然后它将处理它:

```
final HttpClientRequest request = vertx.createHttpClient().post(8080, "localhost", "/");

// add a client response handler
request.handler(resp -> {
    // react on the server response
});

// write the CloudEvent to the given HTTP Post request object
VertxCloudEvents.create().writeToHttpClientRequest(cloudEvent, request);
request.end();

```

创建 HTTP Post 请求后，我们设置了一个异步处理器来处理服务器的*未来*响应。最后，我们的`VertxCloudEvents`实用程序的`writeToHttpClientRequest`用于将实际的`CloudEvent`对象序列化为给定的`HttpClientRequest`。

### 使用 Vert.x 接收云事件

`VertxCloudEvents`实用程序还包含一个不同的函数来接收 Eclipse Vert.x HTTP 服务器应用程序中的`CloudEvent`:

```
import io.cloudevents.http.reactivex.vertx.VertxCloudEvents;
import io.vertx.core.http.HttpHeaders;
import io.vertx.reactivex.core.AbstractVerticle;

public class CloudEventVerticle extends AbstractVerticle {

  public void start() {

    vertx.createHttpServer()
      .requestHandler(req -> VertxCloudEvents.create().rxReadFromRequest(req)
      .subscribe((receivedEvent, throwable) -> {
        if (receivedEvent != null) {
          // I got a CloudEvent object:
          System.out.println("The event type: " + receivedEvent.getType())
        }
      }))
      .rxListen(8080)
      .subscribe(server -> {
        System.out.println("Server running!");
    });
  }
}
```

上面，我们开始一个简单的`HTTPServer`，使用 RxJava 2 的 [Vert.x API。在被动的*请求处理程序中，*我们调用`rxReadFromRequest()`方法并*订阅*它返回的 CloudEvents 以进行进一步处理。现在我们可以在自己的服务器端框架中使用`CloudEvent`对象了！](https://vertx.io/docs/vertx-rx/java2/)

## 结论和展望

使用 CloudEvents 很简单，Vert.x 提供了一个强大的 JVM 工具包，可以在我们的系统中生成或接收和处理 CloudEvents。CloudEvents 正被越来越多的工具和框架所采用，例如 [Knative](https://github.com/knative/) ，它使用 CloudEvents 以标准化的格式在不同的组件和服务之间交换数据。

CloudEvent 规范目前的 0.2 版本还处于早期阶段。然而，即使在这样的初期，它也正在产生吸引力，并证明了一个越来越有用的规范，允许应用程序之间的互操作性。

## 额外资源

*   垂直 x:
    *   *[用 Java 构建反应式微服务:异步和基于事件的应用设计](https://developers.redhat.com/books/building-reactive-microservices-java/)，*免费电子书
    *   *[介绍 Vert.x](https://developers.redhat.com/blog/2018/03/13/eclipse-vertx-first-application/)* 文章系列作者 [Clement Escoffier](https://developers.redhat.com/blog/author/cescoffier/)
        *   第 1 部分—[Vert.x 简介—我的第一个 vert . x 应用](https://developers.redhat.com/blog/2018/03/13/eclipse-vertx-first-application/)
        *   第 2 部分— [Eclipse Vert.x 应用程序配置](https://developers.redhat.com/blog/2018/03/22/eclipse-vert-x-application-configuration/)
        *   第 3 部分— [有些用 Vert.x 休息](https://developers.redhat.com/blog/2018/03/29/rest-vert-x/)
        *   第 4 部分— [访问数据，反应方式](https://developers.redhat.com/blog/2018/04/09/accessing-data-reactive-way/)
        *   第 5 部分— [当垂直 x 遇到无功延伸时](https://developers.redhat.com/blog/author/cescoffier/)
*   云事件:
    *   [Event flow:Red Hat open shift 上的事件驱动微服务](https://developers.redhat.com/blog/2018/10/15/eventflow-event-driven-microservices-on-openshift-part-1/)

*Last updated: October 20, 2022*