# Eclipse Vert.x 3.9 的 Red Hat 版本带来了流畅的 API 查询

> 原文：<https://developers.redhat.com/blog/2020/05/25/red-hat-build-of-eclipse-vert-x-3-9-brings-fluent-api-query>

[Red Hat runtime](https://developers.redhat.com/middleware/)为有云原生应用开发需求的开发人员、架构师和 IT 领导提供了一套全面的框架、运行时和编程语言。Red Hat 运行时的最新更新是 Red Hat 构建的 Eclipse Vert.x 版本 3.9。红帽运行时为应用开发者提供各种应用运行时，让他们在[红帽 OpenShift 容器平台](https://developers.redhat.com/openshift/)上运行。

流畅的 API 是 Vert.x 中的一种常见模式，它允许多个方法调用链接在一起。例如:

```
request.response().putHeader("Content-Type", "text/plain").write("some text").end();
```

像这样链接调用还允许您编写不太冗长的代码。

在 3.9 中，您现在可以通过在 Fluent API 中包含`Query`来创建预准备语句和收集器查询。如果您熟悉 JDBC，`PreparedStatement`让您创建和执行语句。此外，您可以运行多个交互，比如光标或流操作。

## 创建预准备语句

要创建预准备语句，请执行以下操作:

```
connection.prepare(sql, ar1 -> {
  if (ar1.succeded()) {
    PreparedStatement ps = ar1.result();
    PreparedQuery<RowSet<Row>> pq = ps.query();
    pq.execute(tuple, ar2 -> ...);

// Or fluently
    ps.query().execute(tuple, ar2 -> ...);
  }
});
```

## 创建收集器查询

您也可以在 Vert.x 中使用 Java 收集器。例如，要创建收集器查询:

```
PreparedQuery<RowSet<Row>> query = client.preparedQuery(sql);
PreparedQuery<SqlResult<List<Row>> collectedQuery = query.collecting(Collectors.toList());
collectedQuery.execute(tuple, ar -> ...);

// Or fluently
client.preparedQuery(sql).collecting(Collectors.toList()).execute(tuple, ar -> ...);
```

参见 GitHub 上的这些 [Vert.x 示例。](https://github.com/vert-x3/vertx-examples/tree/master/cassandra-examples/src/main/java/io/vertx/example/cassandra/cassandra/prepared)

## 用承诺代替未来

这个版本中的另一个重要变化是不再使用下面的方法:`start(Future<Void>)`和`stop(Future<Void>)`。 [`future()`](https://vertx.io/docs/apidocs/io/vertx/core/Promise.html#future--) 方法返回与承诺相关联的 [`Future`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html) 。然后，未来可用于获得承诺完成的通知并检索其价值。

而是用`start(Promise<Void>)`和`stop(Promise<Void>)`。这些方法表示一个动作的可写部分，这个动作可能已经发生，也可能还没有发生。承诺扩展了`Handler<AsyncResult<T>>`,因此它可以被用作回调。

## 证明文件

要了解更多细节，看看支持的配置和组件细节:[红帽运行时 Eclipse Vert.x 支持的配置](https://access.redhat.com/articles/3348741#VERTX_3_x)和[红帽运行时 Eclipse Vert.x 3.9 组件细节](https://access.redhat.com/articles/4977141)。如果你是 Eclipse Vert.x 的新手，并且想了解更多，请访问我们的[实时学习门户网站，获取有指导的](https://learn.openshift.com/middleware/courses/middleware-vertx/)教程或[文档](https://access.redhat.com/documentation/en-us/red_hat_build_of_eclipse_vert.x/3.9/)以获取详细信息。

## 开发者互动学习场景

这些[自定进度的场景](https://learn.openshift.com/middleware/courses/middleware-vertx/)为您提供了一个预配置的 Red Hat OpenShift 实例，无需任何下载或配置即可从您的浏览器访问。使用它来学习和试验 Vert.x 或了解 Red Hat 运行时中的其他技术，并了解它们如何帮助解决现实世界中的问题。

## 获得对 Eclipse Vert.x 的支持

Red Hat 客户可以通过订阅 [Red Hat Runtimes](https://www.redhat.com/en/products/runtimes) 来获得对 Eclipse Vert.x 的支持。请联系您当地的 Red Hat 代表或 [Red Hat 销售部](https://www.redhat.com/en/about/contact/sales)，了解如何享受 Red Hat 及其全球合作伙伴网络提供的世界级支持。

展望未来，根据 [Red Hat 产品更新和支持生命周期](https://access.redhat.com/support/policy/updates/jboss_notes/)，客户可以期待对 Eclipse Vert.x 和其他运行时的支持。

## Red Hat Runtimes 中 Eclipse Vert.x 背后的人

这个产品是由 Red Hat 的运行时产品和工程团队以及 [Eclipse Vert.x](https://vertx.io/) 上游社区共同开发的。它涉及了许多小时的开发、测试、文档编写、更多的测试，以及与更广泛的客户、合作伙伴和 Vert.x 开发人员的 Red Hat 社区合作，以整合大大小小的贡献。

*Last updated: June 26, 2020*