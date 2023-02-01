# 介绍 Eclipse Vert.x 4.0 的 Red Hat 版本

> 原文：<https://developers.redhat.com/blog/2021/01/21/introducing-the-red-hat-build-of-eclipse-vert-x-4-0>

如果您对反应式、非阻塞和异步 Java 开发感兴趣，您可能熟悉 [Eclipse Vert.x](https://vertx.io/) 。该项目始于 2011 年，并于 2013 年成功转移到 Eclipse 基金会。从那时起，Vert.x 经历了九年的严格发展，成长为一个欣欣向荣的社区。它是使用最广泛的反应式框架之一，支持多种扩展，包括使用 [Kafka](https://developers.redhat.com/topics/kafka-kubernetes) 或 Artemis 进行消息传递或流传输的扩展，使用 gRPC 和 GraphQL 开发应用程序，以及[等等](https://access.redhat.com/articles/3348731)。

Eclipse Vert.x 4.0 的 [Red Hat build 现已全面上市。这个版本改进了 Vert.x 的核心 API 和处理。进行迁移的开发人员可以期待未来和承诺、分布式跟踪和部署在](https://access.redhat.com/documentation/en-us/red_hat_build_of_eclipse_vert.x/4.0/html/release_notes_for_eclipse_vert.x_4.0/index) [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 上的增强。在本文中，我将介绍这些更新，并提供在 OpenShift 上迁移和部署 Eclipse Vert.x 4.0 应用程序的技巧。

**注意**:关于从 Vert.x 3.x 迁移到 Vert.x 4.0 的详细介绍，请参见 Eclipse Vert.x 4.0 迁移指南的[红帽版本。](https://access.redhat.com/documentation/en-us/red_hat_build_of_eclipse_vert.x/4.0/html/eclipse_vert.x_4.0_migration_guide/)

## <future>来了！</future>

`Future`是一个`AsyncResult<T>`，可以用来在 Vert.x 4.0 中创建异步操作。每个异步方法都返回一个`Future`对象、`success`或`failure`，作为调用的结果:

```
FileSystem fs = vertx.fileSystem();

Future<FileProps> future = fs.props("/my_file.txt");

future.onComplete((AsyncResult<FileProps> ar) -> {

if (ar.succeeded()) {

    FileProps props = ar.result();

    System.out.println("File size = " + props.size());

} else {

    System.out.println("Failure: " + ar.cause().getMessage());

}

});

```

如果您喜欢使用回调并获得一个`Handler`返回，Vert.x 4.0 仍然实现了`props`，如下所示:

```
FileSystem props(String path,Handler<AsyncResult<FileProps>> handler)

```

从 Vert.x 3.x 迁移到 Vert.x 4.0 的开发人员也可以将`Future`用于回调:

```
WebClient client = WebClient.create(vertx);

HttpRequest request = client.get("/resource");

Future<HttpResponse> response = request.send();

response.onComplete(ar -> {

if (ar.succeeded()) {

    HttpResponse response = ar.result();

} else {

    Throwable failure = ar.cause();

}

});

```

期货的错误处理比回调更简单。你不需要跟踪自己回到每一个回调，你可以只处理一次失败，在写作的结尾。Futures 还允许您并行或顺序地编写异步事件。总而言之，这个特性大大简化了用 Vert.x 进行应用程序编程。

## 承诺

一个*承诺*代表了一个可能发生也可能没有发生的行动的可写的一面。每个承诺都有一个 [future()](https://vertx.io/docs/apidocs/io/vertx/core/Promise.html#future--) 方法，该方法为给定的承诺返回一个`Future`。你可以同时使用承诺和未来来获得完成通知。下面的例子显示了`HttpServerVerticle`使用一个承诺。

```
package com.example.starter;

import io.vertx.core.AbstractVerticle;
import io.vertx.core.Promise;

public class MainVerticle extends AbstractVerticle {

@Override

public void start(Promise<Void> startPromise) throws Exception {

vertx.createHttpServer().requestHandler(req -> {

req.response()

  .putHeader("content-type", "text/plain")

  .end("Hello from Vert.x!");

  }).listen(8888, http -> {

    if (http.succeeded()) {
        startPromise.complete();
        System.out.println("HTTP server started on port 8888");

    } else {
        startPromise.fail(http.cause());
   }

});

}

}

```

在这种情况下，该方法返回与承诺相关联的[未来](https://vertx.io/docs/apidocs/io/vertx/core/Future.html)。我们使用`Future`在承诺完成时发送通知，并检索其值。此外，promise 扩展了`Handler<AsyncResult<T>>` ,因此我们可以将其用作回调。

**注意**:在将您的应用程序迁移到 Eclipse Vert.x 4.0 之前，请检查是否有弃用和删除。当您使用不推荐使用的 API 时，编译器将生成警告。

## 分布式跟踪

现代分布式软件应用程序中的许多组件都有自己的操作生命周期。面临的挑战是跟踪各种事件，并跨组件关联这些信息。跟踪让我们了解系统的状态，以及系统遵守关键性能指标(KPI)的程度。例如，为了找出有多少人在灾难救援中得到帮助，我们必须在分布式软件系统中关联和跟踪数据。我们可以使用这些信息来评估软件是否满足我们的业务需求。

分布式跟踪让我们能够可视化和理解软件应用程序之间交互的事件链和流程。对于微服务环境中的分布式跟踪，我们可以使用带有 [Jaeger](https://www.redhat.com/en/topics/microservices/what-is-jaeger) 的 Vert.x 4.0，Jaeger 是一个 [OpenTracing](https://opentracing.io) 客户端，它是云本地计算基础的一部分。

**注**:关于安装 Jaeger 的说明，参见 Red Hat OpenShift [配置和部署 Jaeger 指南](https://docs.openshift.com/container-platform/4.6/jaeger/jaeger_install/rhbjaeger-deploying.html)。

一旦我们安装了 Jaeger，我们就可以使用以下 Vert.x 组件来记录跟踪:

*   HTTP 服务器和 HTTP 客户端
*   Eclipse Vert.x SQL 客户端
*   eclipse green . x Kaka 客户端

这些组件中的每一个都实现了以下`TracingPolicy`:

*   [传播](https://vertx.io/docs/apidocs/io/vertx/core/tracing/TracingPolicy.html#PROPAGATE):组件报告活动轨迹中的跨度。
*   [ALWAYS](https://vertx.io/docs/apidocs/io/vertx/core/tracing/TracingPolicy.html#ALWAYS) :组件报告活动轨迹中的跨度或创建新的活动轨迹。
*   [忽略](https://vertx.io/docs/apidocs/io/vertx/core/tracing/TracingPolicy.html#IGNORE):忽略对有问题组件的跟踪。

下面是一个简单的跟踪策略实现示例:

```
HttpServer server = vertx.createHttpServer(new HttpServerOptions()

.setTracingPolicy(TracingPolicy.IGNORE)

);

```

请参见 [Vert.x Opentracing 示例](https://github.com/vert-x3/vertx-examples/tree/4.x/opentracing-examples)库，了解有关 Vert.x 和 Jaeger 的更详细的跟踪示例。

## OpenShift 的计量标签

现在，您可以向 OpenShift 上运行的 Eclipse Vert.x 应用程序添加计量标签。客户使用标签来跟踪他们已订阅的部署。Eclipse Vert.x 使用以下计量标签:

*   com . red hat . component-name:vert . x
*   组件类型:应用程序
*   com . red hat . component-版本:4.0.0
*   com . Red Hat . product-name:" Red _ Hat _ Runtimes "
*   com . red hat . product-version:2021/Q1

有关计量标签的更多信息，请参见 OpenShift 4.6 文档。

## 将 Eclipse Vert.x 应用程序部署到 OpenShift

Eclipse JKube 是一个插件和库的集合，用于使用 Docker、Jib 或源到映像(S2I)构建策略来构建容器映像。与它的前身 Fabric8 不同，Eclipse JKube 简化了在 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 和 OpenShift 上的 Java 开发。开发人员可以专注于创建应用程序，而无需深入细节，例如创建清单。

Eclipse JKube 将清单作为 Maven 构建的一部分，然后在编译时生成并部署它们。JKube 插件自动为您生成资源清单，您可以在以后应用它。下面是一个如何使用 Eclipse JKube 在 OpenShift 上运行应用程序的示例:

```
# The following commmands will create your OpenShift resource descriptors.

mvn clean oc:resource -Popenshift

# Starting the S2I build

mvn package oc:build -Popenshift

# Deploying to OpenShift

mvn oc:deploy -Popenshift

```

更多细节请参见这个使用 Eclipse JKube 插件的示例项目。

## 打包和部署

现在，您可以将您的应用程序打包并部署到 OpenShift，并在 [Red Hat Enterprise Linux 8](https://developers.redhat.com/topics/linux) 上为 [Red Hat OpenJDK 8 和 11](https://developers.redhat.com/products/openjdk/overview) 提供符合开放容器倡议(OCI)的[通用基础映像](https://developers.redhat.com/articles/ubi-faq)。

此外，Vert.x `vertx-web-client.js`现在发布在 NPM 知识库中，不再作为 Maven 工件提供。可以从[@ vertx/event bus-bridge-client . js](https://www.npmjs.com/package/@vertx/eventbus-bridge-client.js)访问客户端。

## 反应式编程新手？

如果您是反应式编程的新手，您可以使用[自定进度的场景](https://learn.openshift.com/middleware/courses/middleware-vertx/)来学习和体验 Vert.x 或了解 Red Hat 运行时中的其他技术。每个场景都提供了一个预配置的 Red Hat OpenShift 实例，无需任何下载或配置即可从您的浏览器访问该实例。

对于喜欢深入研究的开发人员，我推荐阅读[朱利安·庞奇](https://developers.redhat.com/blog/author/jponge/)的 [*Vert.x in Action*](https://www.manning.com/books/vertx-in-action) 。

## 获得 Vert.x 4.0 的红帽版本

通过订阅 [Red Hat Runtimes](https://www.redhat.com/en/products/runtimes) ，Red Hat 客户可以获得对 Eclipse Vert.x 的支持。红帽的运行时支持是根据[红帽产品更新和支持生命周期](https://access.redhat.com/support/policy/updates/jboss_notes/)安排的。

如果您是 Eclipse Vert.x 的新手，并且想要了解更多，请访问我们的实时学习门户网站，获取有指导的[教程](https://learn.openshift.com/middleware/courses/middleware-vertx/)，或者查看[产品文档](https://access.redhat.com/documentation/en-us/red_hat_build_of_eclipse_vert.x/4.0/)了解技术细节。您还可以在 Red Hat 运行时上查看 Eclipse Vert.x 的[支持的配置](https://access.redhat.com/articles/3985941)和[组件细节](https://access.redhat.com/articles/3348731)，并查看[迁移指南](https://access.redhat.com/documentation/en-us/red_hat_build_of_eclipse_vert.x/4.0/html/eclipse_vert.x_4.0_migration_guide/)了解从 Eclipse Vert.x 3.x 迁移到 Vert.x 4.0 的详细介绍。

*Last updated: January 20, 2021*