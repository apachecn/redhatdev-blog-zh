# 当垂直 x 遇到电抗性延伸时(垂直 x 介绍第 5 部分)

> 原文：<https://developers.redhat.com/blog/2018/04/18/eclipse-vertx-reactive-extensions>

这篇文章是我的 [I *介绍 Eclipse vert . x*T4 系列的第五篇文章。在](https://developers.redhat.com/blog/author/cescoffier/)[的上一篇文章](https://developers.redhat.com/blog/2018/04/09/accessing-data-reactive-way/)中，我们看到了 Vert.x 如何与数据库交互。为了驯服 Vert.x 的异步本质，我们使用了`Future`对象。在这篇文章中，我们将看到另一种管理异步代码的方法:反应式编程。我们将会看到 Vert.x 和反应式延伸是如何给你超能力的。

先用之前的帖子来刷新一下我们的记忆:

*   第一篇文章描述了如何用 Apache Maven 构建一个 Vert.x 应用程序并执行单元测试。
*   第二篇文章描述了这个应用程序是如何变得可配置的。
*   [第三篇](https://developers.redhat.com/blog/2018/03/29/rest-vert-x)介绍`vertx-web`，一个收藏管理应用被开发出来。这个应用程序公开了一个 HTML/JavaScript 前端使用的 REST API。
*   [在第四篇文章](https://developers.redhat.com/blog/2018/04/09/accessing-data-reactive-way/)中，我们用数据库替换了内存后端，并引入了`Future`来协调我们的异步操作。

在这篇文章中，我们不打算增加新的功能。相反，我们将探索另一种编程范式:反应式编程。

这篇文章的代码可以在 [GitHub repo](https://github.com/redhat-developer/introduction-to-eclipse-vertx) 的`post-5`目录中找到。

## 反应性思维

忘掉你所知道的关于代码的一切，四处看看。用代码对这个世界建模是具有挑战性的。作为开发人员，我们倾向于使用反直觉的方法。自 20 世纪 80 年代以来，面向对象计算一直被视为银弹。我们世界中的每个实体都由一个包含字段和公开方法的对象来表示。大多数时候，与这些对象的交互是使用一个阻塞和同步协议来完成的。您调用一个方法并等待响应。但是...我们生活的世界是异步的。交互是通过事件、消息和刺激来完成的。为了克服面向对象的局限性，出现了许多模式和范例。最近，函数式编程正在卷土重来，不是取代面向对象，而是补充面向对象。*反应式编程*是一种功能性的事件驱动编程方法，与常规的面向对象范例结合使用。

几年前，微软为。NET 称为[反应式扩展](http://reactivex.io/)(也称为 ReactiveX 或 RX)。RX 是一个用*可观察流*进行异步编程的 API。这个 API 已经被移植到多种语言中，比如 JavaScript、Python、C++和 Java。

让我们观察一下我们的世界。观察运动中的实体:交通堵塞、天气、对话和金融市场。事物是同时运动和发展的。多个*事情*同时发生，有时独立发生，有时以协调的方式发生。每个对象都在创建一个事件的*流*。例如，您的鼠标光标位置正在移动。位置序列是一个流。房间里的人数可能是稳定的，但有人可以进来或出去，产生一个新的值。所以我们有了另一个价值流。反应式编程背后有一个基本原则:*事件是数据，数据是事件*。

理解 RX 和异步编程的重要一点是流的异步特性。您观察一个流，当流发出一个项目时，您会得到通知。你不知道那会在什么时候发生，但是你在观察。这个观察是使用`subscribe`操作完成的。

[![](img/a2157d2fe52a6ea4a5c5e1c4838151c0.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/04/streams-reactive.png)

RxJava 是 Java 编程语言 RX 的直接实现。这是一个非常流行的 Java 反应式编程库，应用于网络数据处理和 JavaFX 和 Android 的图形用户界面。RxJava 是 Java 中反应式库的通用语言，它提供了以下五种类型来描述发布者:

|  | 流中的项数 | xJava 2 类型 | RX 签名 | 回调签名 | 未来签名 |
| 通知，数据流 | 0..n | 可观察的、可流动的 | 
可观察的水流()可流动的水流() | ReadStream 方法() | 不适用的 |
| 产生结果的异步操作 | one | 单一的 | 单一 get() | `void get(句柄<asyncresult>句柄)` | 未来获取() |
| 不产生结果或只产生一个结果的异步操作 | 0..1 | 可能 | `也许 findById(字符串 Id)` | `void get(字符串 id，处理程序<asyncresult>处理程序) | 未来获取(字符串 id)` |
| 不产生结果的异步操作 | Zero | 可完成 | 可完成同花顺() | `无效冲洗(处理器<asyncresult>处理器)` | 未来同花顺() |

`Observable`和`Flowable`的区别在于`Flowable`处理背压(实现反应流协议),而`Observable`不处理。`Flowable`更适合来自支持背压的数据源(例如，TCP 连接)的大数据流，而`Observable`更适合处理无法施加背压的“热”可观察对象(例如，GUI 事件)。

这篇文章不是对反应式编程或 RX 的介绍。如果你需要一个关于反应式编程和 RX 的入门课程，请查看本教程。

在上一篇文章中，我们使用了`Future`来编写异步操作。在这篇文章中，我们将使用 streams 和 RxJava。怎么会？感谢 Vert.x 和 RxJava 2 APIs。事实上，Vert.x 提供了一组 *RX 认证的*API。但是，不要忘记:

*   没有 Vert.x 也可以使用 RxJava
*   没有 RxJava 也可以使用 Vert.x

将它们结合起来会给你带来超能力，因为它用 RxJava 流和操作符的能力扩展了 Vert.x 的异步执行模型。

## 说够了；给我看一些代码

它总是从一个 Maven 依赖项开始。在您的`pom.xml`文件中添加以下内容:

```
io.vertx
   vertx-rx-java2
   ${vertx.version}
```

然后，打开`io.vertx.intro.first.MyFirstVerticle`类，将导入语句替换为:

```
import io.reactivex.Completable;
import io.reactivex.Single;
import io.vertx.core.Future;
import io.vertx.core.json.JsonArray;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.sql.SQLOptions;
import io.vertx.reactivex.CompletableHelper;
import io.vertx.reactivex.config.ConfigRetriever;
import io.vertx.reactivex.core.AbstractVerticle;
import io.vertx.reactivex.core.buffer.Buffer;
import io.vertx.reactivex.core.http.HttpServerResponse;
import io.vertx.reactivex.ext.jdbc.JDBCClient;
import io.vertx.reactivex.ext.sql.SQLConnection;
import io.vertx.reactivex.ext.web.Router;
import io.vertx.reactivex.ext.web.RoutingContext;
import io.vertx.reactivex.ext.web.handler.BodyHandler;
import io.vertx.reactivex.ext.web.handler.StaticHandler;

import java.util.List;
import java.util.NoSuchElementException;
import java.util.stream.Collectors;
```

注意`io.vertx.reactivex`包。它是实现 Vert.x RX API 的地方。所以，我们现在不是延长`io.vertx.core.AbstractVerticle`，而是延长`io.vertx.reactivex.core.AbstractVerticle`。注入的`vertx`实例提出了以`rx`前缀开始的新方法，比如`rxDeployVerticle`或`rxClose`。以`rx`为前缀的方法返回 RxJava 2 类型，如`Single`或`Completable`。

## 从回归未来到回归单一和完整

为了从 RX API 中获益并能够使用 RX 操作符，我们需要使用 RX 类型。例如，以前我们有这样一个例子:

```
private Future createHttpServer(JsonObject config, 
  Router router) {
  Future future = Future.future();
  vertx
    .createHttpServer()
    .requestHandler(router::accept)
    .listen(
      config.getInteger("HTTP_PORT", 8080),
      res -> future.handle(res.mapEmpty())
    );
  return future;
}
```

`Future`映射到 RX 中的`Completable`，*即*一个只是表示其完成的流。因此，对于 RX，该代码如下所示:

```
private Completable createHttpServer(JsonObject config,
  Router router) {
  return vertx
    .createHttpServer()
    .requestHandler(router::accept)
    .rxListen(config.getInteger("HTTP_PORT", 8080))
    .toCompletable();
}
```

你看出区别了吗？我们使用`rxListen`方法返回一个`Single`。因为我们不需要服务器，所以我们使用`toCompletable`方法将它转换成一个`Completable`。因为我们使用了 *rx 化的* `vertx`实例，所以`rxListen`是可用的。

现在让我们重写`connect`方法。`connect`正在返回一个`Future`。这被翻译成`Single`:

```
private Single connect() {
  return jdbc.rxGetConnection()
    .map(c -> c.setOptions(
       new SQLOptions().setAutoGeneratedKeys(true)));
}
```

`jdbc`客户端也提供了一个`rx` API。`rxGetConnection`返回一个`Single`。为了能够生成密钥，我们使用了`map`方法。`map`从观察到的`Single`中获取结果，然后*使用*映射器*功能将其转换为*。这里我们只是修改选项。

遵循同样的原则，`insert`方法重写如下:

```
private Single insert(SQLConnection connection, 
 Article article, boolean closeConnection) {
  String sql = "INSERT INTO Articles (title, url) VALUES (?, ?)";
  return connection
    .rxUpdateWithParams(sql,
      new JsonArray().add(article.getTitle()).add(article.getUrl()))
    .map(res -> new Article(res.getKeys().getLong(0),
      article.getTitle(), article.getUrl()))
    .doFinally(() -> {
      if (closeConnection) {
        connection.close();
      }
    });
}
```

这里，我们使用`rxUpdateWithParams`执行`INSERT`语句。结果就转化成了一个`Article`。注意`doFinally`。当操作完成或失败时，调用此方法。在这两种情况下，如果被请求，我们关闭连接。

同样的方法也适用于使用`rxQuery`方法的`query`方法:

```
private Single query(SQLConnection connection) {
  return connection.rxQuery("SELECT * FROM articles")
    .map(rs -> rs.getRows().stream()
      .map(Article::new)
      .collect(Collectors.toList())
    )
    .doFinally(connection::close);
}
```

如果搜索的文章不存在，`queryOne`需要*向*抛出一个错误:

```
private Single queryOne(SQLConnection connection, String id) {
  String sql = "SELECT * FROM articles WHERE id = ?";
  return connection.rxQueryWithParams(sql,
    new JsonArray().add(Integer.valueOf(id))
    )
    .doFinally(connection::close)
    .map(rs -> {
      List rows = rs.getRows();
      if (rows.size() == 0) {
          throw new NoSuchElementException(
            "No article with id " + id);
      } else {
          JsonObject row = rows.get(0);
          return new Article(row);
        }
    });
}
```

由*映射器*函数抛出的异常被传播到流。因此观察者可以对其做出反应并恢复。

## 转换类型

我们已经看到上面的`toCompletable`方法丢弃了来自`Single`的结果，只是通知订户操作成功完成或失败。在`update`和`delete`方法中，我们需要做几乎相同的事情。我们执行 SQL 语句，如果我们意识到这些语句没有更改任何行，我们将报告一个错误。为了实现这一点，我们使用了`flatMapCompletable`。这个方法是`flatMap`家族的一部分，它是一个非常强大的 RX 操作符。此方法将函数作为参数。对于被观察的流发出的每个项目，都调用这个函数。如果流是一个`Single`，它将被调用零次(错误情况)或一次(操作成功并有结果)。与`map`运算符不同，`flatMap`函数返回一个流。例如，在我们的上下文中，用一个`UpdateResult`调用`flatMapCompletable`函数并返回一个`Completable`:

```
private Completable update(SQLConnection connection, String id,
  Article article) {
  String sql = "UPDATE articles SET title = ?,
    url = ? WHERE id = ?";
  JsonArray params = new JsonArray().add(article.getTitle())
    .add(article.getUrl())
    .add(Integer.valueOf(id));
  return connection.rxUpdateWithParams(sql, params)
    .flatMapCompletable(ur ->
      ur.getUpdated() == 0 ?
        Completable
            .error(new NoSuchElementException(
                "No article with id " + id))
        : Completable.complete()
    )
    .doFinally(connection::close);
}

private Completable delete(SQLConnection connection, String id) {
  String sql = "DELETE FROM Articles WHERE id = ?";
  JsonArray params = new JsonArray().add(Integer.valueOf(id));
  return connection.rxUpdateWithParams(sql, params)
    .doFinally(connection::close)
    .flatMapCompletable(ur ->
        ur.getUpdated() == 0 ?
          Completable
              .error(new NoSuchElementException(
                  "No article with id " + id))
          : Completable.complete()
    );
}
```

在这两种情况下，我们检查更新的行数，如果为 0，则产生一个失败的`Completable`。因此订户要么收到成功消息(`Completable.complete`)，要么收到错误消息(`Completable.error`)。注意，这段代码也可以使用前面的方法:使用`map`操作符，抛出一个异常，并使用`toCompletable`丢弃结果。

显然，我们也可以将一个`Completable`转换成一个`Single`:

```
private Single createTableIfNeeded(
  SQLConnection connection) {
    return vertx.fileSystem().rxReadFile("tables.sql")
        .map(Buffer::toString)
        .flatMapCompletable(connection::rxExecute)
        .toSingleDefault(connection);
}
```

`rxExecute`返回一个`Completable`。但是这里我们需要转发一下`SQLConnection`。幸运的是，`toSingleDefault`操作器将`Completable`转换成发出给定值的`Single`。

## 编写异步操作

到目前为止，我们正在使用`rx`方法并调整结果。但是如何处理顺序构图呢？执行第一个操作，然后用第一个操作的结果执行第二个操作？这可以使用`flatMap`操作符来完成。如上所述，`flatMap`是一个非常强大的运营商。它接收一个函数作为参数，与`map`操作符不同，这个函数返回一个流(所以`Single`、`Maybe`、`Completable`...).对于观察到的流中的每一项都调用这个函数，返回的流被*展平*，因此这些项被序列化为一个流。因为流是异步构造，所以调用`flatMap`会创建一个顺序组合。我们来看看`createSomeDataIfNone`的方法。初始实现如下:

```
private Future createSomeDataIfNone(
  SQLConnection connection) {
  Future future = Future.future();
  connection.query("SELECT * FROM Articles", select -> {
    if (select.failed()) {
      future.fail(select.cause());
    } else {
      if (select.result().getResults().isEmpty()) {
        Article article1 = new Article("Fallacies of distributed computing",            "https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing");
        Article article2 = new Article("Reactive Manifesto",
            "https://www.reactivemanifesto.org/");
        Future insertion1 = insert(connection, article1, false);
        Future insertion2 = insert(connection, article2, false);
        CompositeFuture.all(insertion1, insertion2)
            .setHandler(r -> future.handle(r.map(connection)));
      } else {
        future.complete(connection);
      }
    }
  });
  return future;
}
```

在这个方法中，我们执行一个查询，并根据结果插入文章。RX 实现如下:

```
private Single createSomeDataIfNone(
  SQLConnection c) {
  return c.rxQuery("SELECT * FROM Articles")
    .flatMap(rs -> {
      if (rs.getResults().isEmpty()) {
        Article article1 = new Article("Fallacies of distributed computing",
            "https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing");
        Article article2 = new Article("Reactive Manifesto",
            "https://www.reactivemanifesto.org/");
        return Single.zip(
            insert(connection, article1, false),
            insert(connection, article2, false),
            (a1, a2) -> c
        );
      } else {
          return Single.just(c);
        }
    });
}
```

首先，我们执行查询。然后，当我们得到结果时，调用传递给`flatMap`方法的函数，实现顺序合成。你可能想知道错误的情况。我们不需要处理它，因为错误会传播到流中，最终的观察者会收到它。发生错误时不会调用该函数。

异步操作可以并发发生。但是有时候你需要知道他们什么时候完成。这叫*平行构图*。`zip`操作符允许您这样做。在`createSomeDataIfNone`中，我们插入两篇文章。这个操作是用`insert`(返回一个`Single`)完成的。`zip`操作符观察到`Single`的两次给定出现，并在两次都完成时调用作为最后一个参数传递的方法。在这种情况下，我们只需转发`SQLConnection`。

## 合成一切以做好准备

我们已经重写了大部分函数，但是我们需要修改`start`方法。记住我们需要实现的开始顺序:

```
// Start sequence:
// 1 - Retrieve the configuration
//  |- 2 - Create the JDBC client
//  |- 3 - Connect to the database (retrieve a connection)
//    |- 4 - Create table if needed
//      |- 5 - Add some data if needed
//        |- 6 - Close connection when done
//    |- 7 - Start HTTP server
//  |- 9 - we are done!
```

这个组合可以使用`flatMap`操作符来实现:

```
retriever.rxGetConfig()
  .doOnSuccess(config ->
    jdbc = JDBCClient.createShared(vertx, config, 
      "My-Reading-List"))
  .flatMap(config ->
    connect()
      .flatMap(connection ->
          this.createTableIfNeeded(connection)
              .flatMap(this::createSomeDataIfNone)
              .doAfterTerminate(connection::close)
      )
      .map(x -> config)
  )
  .flatMapCompletable(c -> createHttpServer(c, router))
  .subscribe(CompletableHelper.toObserver(fut));
```

`doOnSuccess`是一个动作操作符，它从被观察的流中接收项目，并让您实现一个*副作用*。这里我们指定了`jdbc`字段。

然后我们使用`flatMap`操作符来编排我们不同的动作。再看`doAfterTerminate`。这个操作符让我们在使用完整个流时关闭连接，这对于清理非常有用。

这段代码中有一个重要的部分。到目前为止，我们返回了 RX 类型，但从未调用过`subscribe`。如果你不订阅，什么都不会发生:流是懒惰的。*所以千万别忘了订阅*。订阅实现了管道并触发了排放。在我们的代码中，它触发启动序列。传递给`subscribe`方法的参数只是向传递给`start`方法的`Future`对象报告失败和成功。基本上，它将一个`Future`映射到一个`Subscriber`。

## 实现 HTTP 操作

我们差不多完成了。我们只需要更新我们的 HTTP 动作，即在 HTTP 请求上调用的方法。为了简化代码，让我们修改一下`ActionHelper`类。这个类提供了返回`Handler<AsyncResult>`的方法。但是这种类型不适合需要订阅者的 RX APIs。让我们用返回更合适类型的方法来替换这些方法:

```
private static  BiConsumer writeJsonResponse(
  RoutingContext context, int status) {
  return (res, err) -> {
    if (err != null) {
      if (err instanceof NoSuchElementException) {
        context.response().setStatusCode(404)
          .end(err.getMessage());
      } else {
        context.fail(err);
      }
    } else {
      context.response().setStatusCode(status)
        .putHeader("content-type", 
            "application/json; charset=utf-8")
        .end(Json.encodePrettily(res));
    }
  };
}

static  BiConsumer; ok(RoutingContext rc) {
  return writeJsonResponse(rc, 200);
}

static  BiConsumer created(RoutingContext rc) {
  return writeJsonResponse(rc, 201);
}

static Action noContent(RoutingContext rc) {
  return () -> rc.response().setStatusCode(204).end();
}

static Consumer onError(RoutingContext rc) {
  return err -> {
      if (err instanceof NoSuchElementException) {
          rc.response().setStatusCode(404)
           .end(err.getMessage());
      } else {
          rc.fail(err);
      }
  };
}
```

现在我们已经准备好实现我们的 HTTP 操作方法了。回到`MyFirstVerticle`类；用下面的代码替换动作方法:

```
private void getAll(RoutingContext rc) {
  connect()
      .flatMap(this::query)
      .subscribe(ok(rc));
}

private void addOne(RoutingContext rc) {
  Article article = rc.getBodyAsJson()
    .mapTo(Article.class);
  connect()
      .flatMap(c -> insert(c, article, true))
      .subscribe(created(rc));
}

private void deleteOne(RoutingContext rc) {
  String id = rc.pathParam("id");
  connect()
      .flatMapCompletable(c -> delete(c, id))
      .subscribe(noContent(rc), onError(rc));
}

private void getOne(RoutingContext rc) {
  String id = rc.pathParam("id");
  connect()
      .flatMap(connection -> queryOne(connection, id))
      .subscribe(ok(rc));
}

private void updateOne(RoutingContext rc) {
  String id = rc.request().getParam("id");
  Article article = rc.getBodyAsJson()
    .mapTo(Article.class);
  connect()
      .flatMapCompletable(c -> update(c, id, article))
      .subscribe(noContent(rc), onError(rc));
}
```

如您所见，这些方法是使用我们之前看到的操作符实现的。它们包含写 HTTP 响应的`subscribe`调用。就这么简单...

## 结论

我们完了！在这篇文章中，我们修改了代码，使用了反应式编程和 RxJava 2。Vert.x 和 RxJava 的结合将你的*反应力*带到了另一个层次。您可以非常容易地组合和处理异步操作和流。

现在，不要忘记没有什么是免费的。RX 可能很难理解。这可能看起来很奇怪。根据你的背景，你可能更喜欢`Future`和复试。Vert.x 为您提供选择，您可以自由选择您喜欢的型号。

如果你想更进一步，这里有一些资源:

*   [RxJava 2 教程](http://escoffier.me/rxjava-hol/)
*   [关于 RxJava 和 Vert.x 的文章](http://www.javamagazine.mozaicreader.com/JanFeb2018#&pageSet=32&page=0&contentItem=0)
*   [展示如何使用 Vert.x 实施微服务的免费小册子](https://developers.redhat.com/promotions/building-reactive-microservices-in-java/)

本系列的下一篇文章将介绍我们的应用程序在 Kubernetes 和 OpenShift 上的部署。

敬请关注，祝编码愉快！

*Last updated: April 22, 2018*