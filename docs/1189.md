# 访问数据——被动的方式

> 原文：<https://developers.redhat.com/blog/2018/04/09/accessing-data-reactive-way>

这是我“*介绍[Eclipse vert . x](https://vertx.io/)T4”的第四篇帖子系列。在本文中，我们将看到如何在 Eclipse Vert.x 应用程序中使用由 [vertx-jdbc-client](http://vertx.io/docs/vertx-jdbc-client/java/) 提供的异步 API 来使用 JDBC。但是在深入 JDBC 和其他 SQL 细节之前，我们先来谈谈 Vert.x `Futures`。*

## 在“Vert.x 简介”系列中

让我们先来回顾一下之前的文章:

1.  第一篇文章描述了如何用 Maven 构建一个 vert.x 应用程序并执行单元测试。
2.  第二篇文章回顾了这个应用是如何变得可配置的。
3.  第三篇文章介绍了 vertx-web，并开发了一个收藏管理应用程序。这个应用程序公开了一个 HTML/JavaScript 前端使用的 REST API。

在第四篇文章中，我们将修复应用程序的主要缺陷:内存后端。当前应用程序使用内存中的`Map`来存储产品(商品)。这非常有用，因为我们每次重启应用程序都会丢失内容。让我们使用一个数据库。在这篇文章中，我们将使用 PostgreSQL，但是你可以使用任何提供 JDBC 驱动的数据库。例如，我们的测试将使用 HSQL。与数据库的交互是异步的，使用`vertx-jdbc-client`来完成。但是在深入这些 JDBC 和 SQL 细节之前，让我们先介绍一下 Vert.x `Future`类，并解释它将如何使异步协调变得更加简单。

这篇文章的代码可以在 [Github repo](https://github.com/redhat-developer/introduction-to-eclipse-vertx) 的`post-4`目录中找到。

## 异步 API

Eclipse Vert.x 的特征之一是它的异步和非阻塞特性。使用异步 API，您不必等待结果，但是当结果准备好，操作完成时，您会得到通知...为了说明这一点，我们举一个非常简单的例子。

让我们想象一个`retrieve`方法。传统上，你会这样使用它:`String r = retrieve()`。这是一个同步 API，当`retrieve`方法返回结果时，执行继续。这个 API 的异步版本应该是:`retrieve(r -> { /* do something with the result */ })`。在这个版本中，当结果被计算出来时，你传递一个被调用的函数(在 Vert.x 行话中是`Handler`)。该函数不返回任何内容，并且在计算出结果后被调用。例如，`retrieve`方法代码可能是这样的:

```
public void retrieve(Handler resultHandler) {
    fileSystem.read(fileName, res -> {
        resultHandler.handle(res);
    });
}
```

为了避免误解，异步 API 与线程无关。正如我们在`retrieve`示例中看到的，没有涉及到线程，大多数 Vert.x 应用程序在异步和非阻塞的情况下使用非常少的线程。此外，重要的是要注意该方法是非阻塞的。在调用`resultHandler`之前，`retrieve`方法可能会返回。

异步操作也可以...失败。因此，我们需要一种方法来封装这些失败，并将它们转发给回调。由于异步性，我们不能使用`try-catch`块。为了捕捉失败或操作的结果，Vert.x 提出了`AsyncResult`类型。我们的`Handler`不再接收普通的结果，而是一个`AsyncResult`封装了成功情况下的结果，或者错误情况下发生的错误:

```
public void retrieve(
  Handler<AsyncResult> resultHandler) {
    vertx.fileSystem().readFile("fileName", ar -> {
      if (ar.failed()) {
        resultHandler.handle(
          Future.succeededFuture(ar.result().toString()));
      } else {
        resultHandler.handle(
          Future.failedFuture(ar.cause()));
      }
    });
}
```

看`if-else`块。用`AsyncResult`的时候会看到很多。这里就不详述`Future`了，后面会讲到，耐心点。目前，`Future.succeededFuture`和`Future.failedFuture`只是创建`AsyncResult`实例的工厂方法。在消费者方面，您应该:

```
retrieve(ar -> {
  if (ar.failed()) {
    // Handle the failure, the exception is 
    // retrieved using ar.cause()
    Throwable cause = ar.cause();
    // ...
   } else {
    // Made it, the result is in ar.result()
    String content = ar.result();
    // ...
   }
});
```

因此，概括地说，异步方法是一种将结果或失败作为通知转发的方法，通常调用一个回调函数来期待结果。

## 异步协调困境

一旦您有了一组异步方法，您通常希望编排它们:

1.  因此一旦另一个调用完成。
2.  并发地，所以同时调用几个动作，并且当所有/其中一个动作完成时被通知。

对于第一种情况，我们会这样做:

```
retrieve(ar -> {
  if (ar.failed()) {
    // do something to recover
   } else {
    String r = ar.result();
    // call another async method
    anotherAsyncMethod(r, ar2 -> {
      if (ar2.failed()) {
        //...
      } else {
        // ...
      }
    })
   }
});
```

您可以很快发现问题...事情开始变得混乱。嵌套的回调降低了代码的可读性，这仅仅是两个。想象一下要处理的不仅仅是这些，我们将在本文后面看到。

对于第二类作文，难度也可想而知。在每个结果处理程序中，您需要检查其他处理程序是完成了还是失败了，然后做出相应的反应。这导致复杂的代码。

## 未来和复合未来(轻松实现异步协调)

为了降低代码复杂度，Vert.x 提出了一个名为`Future`的类。`Future`是一个对象，它封装了一个动作的结果，这个动作可能已经发生，也可能还没有发生。与普通的 Java Future 不同，Vert.x `Future`是非阻塞的，当`Future`完成或`failed`时会调用一个`Handler`。`Future`类实现了`AsyncResult`,因为它表示异步计算的结果。

*关于 Java 未来的一个注意事项:*常规 Java `Future`正在阻塞。调用`get`会阻塞调用方线程，直到收到结果(或者超时)。如果还没有收到结果，Vert.x `Futures`也有一个`get`方法返回`null`。他们还希望有一个处理程序附加到他们身上，在收到结果时调用它。

使用`Future.future()`工厂方法创建一个`Future`对象:

```
Future future = Future.future();
future.complete(1); // Completes the Future with a result
future.fail(exception); // Fails the Future

// To be notified when the future has been completed 
// or failed
future.setHandler(ar -> {
  // Handler called with the result or the failure, 
  // ar is an AsyncResult
});
```

让我们重温一下我们的`retrieve`方法。我们可以返回一个`Future`对象，而不是将回调作为参数:

```
public Future retrieve() {
    Future future = Future.future();
    vertx.fileSystem().readFile("fileName", ar -> {
        if (ar.failed()) {
            future.failed(ar.cause());
        } else {
            future.complete(ar.result().toString());
        }
    });
    return future;
}
```

如上所述，理解这个`retrieve`方法可能在接收值之前返回它的`Future`是很重要的。所以，`return future;`语句是在执行`future.handle(...)`之前执行的。经验丰富的开发人员编写的代码会有所不同:

```
public Future retrieve() {
  Future future = Future.future();
  vertx.fileSystem().readFile("fileName", 
    ar -> future.handle(ar.map(Buffer::toString)));
  return future;
}
```

我们将在几分钟内讨论这个 API。但是首先，让我们看看呼叫者方面，情况没有太大变化。处理程序附加在返回的`Future`上。

```
retrieve().setHandler(ar -> {
  if (ar.failed()) {
    // Handle the failure, the exception is 
    // retrieved using ar.cause()
    Throwable cause = ar.cause();
    // ...
   } else {
    // Made it, the result is in ar.result()
    int r = ar.result();
    // ...
   }
});
```

当您需要编写异步动作时，事情变得容易多了。使用`compose`方法处理顺序合成:

```
retrieve()
  .compose(this::anotherAsyncMethod)
  .setHandler(ar -> {
    // ar.result is the final result
    // if any stage fails, ar.cause is 
    // the thrown exception
  });
```

`Future.compose`将使用前一个`Future`的结果并返回另一个`Future`的函数作为参数。通过这种方式，您可以链接许多异步操作。

并发作文呢。假设您希望调用两个不相关的操作，并在两个操作都完成时得到通知:

```
Future future1 = retrieve();
Future future2 = anotherAsyncMethod();
CompositeFuture.all(future1, future2)
  .setHandler(ar -> {
    // called when either all future have completed
    // successfully (success), 
    // or one failed (failure)
});
```

`CompositeFuture`是一个伴生类，简化了大规模并发合成。`all`不是唯一提供的运算符，您可以使用`join`、`any`...

使用`Future`和`CompositeFuture`使代码更具可读性和可维护性。Vert.x 还支持 RX Java 来管理异步合成，这将在另一篇文章中介绍。

## JDBC:是的，但是不同步

所以，现在我们已经看到了一些关于异步 API 和`Future`的基础知识，让我们来看看`vertx-jdbc-client`。这个 Vert.x 模块让我们通过 JDBC 驱动程序与数据库进行交互。这些交互是异步的，因此当您执行以下操作时:

```
String sql = "SELECT * FROM Products";
ResultSet rs = stmt.executeQuery(sql);
```

当您使用`vertx-jdbc-client`时，它变成:

```
connection.query("SELECT * FROM Products", result -> {
        // do something with the result
});
```

这个模型避免了等待结果。从数据库中检索到结果时会通知您。

*关于 JDBC* 的说明:JDBC 默认是一个阻塞 API。为了与数据库交互，Vert.x 委托给一个*工作线程*。虽然它是异步的，但也不是完全无阻塞的。然而，Vert.x 生态系统也为 MySQL 和 PostgreSQL 提供了真正无阻塞的客户端。

现在让我们修改我们的应用程序，使用数据库来存储我们的产品(文章)。

## 一些 Maven 依赖项

我们需要做的第一件事是在我们的`pom.xml`文件中声明两个新的 Maven 依赖项:

```
io.vertx
  vertx-jdbc-client
  ${vertx.version}

  org.postgresql
  postgresql
  9.4.1212
```

第一个依赖项提供了`vertx-jdbc-client`，而第二个提供了 PostgreSQL JDBC 驱动程序。如果要使用另一个数据库，请更改这种依赖关系。您还需要在代码中更改 JDBC URL 和 JDBC 驱动程序类名。

## 初始化 JDBC 客户端

现在我们已经添加了这些依赖项，是时候创建我们的 JDBC 客户端了。但是需要配置。编辑`src/main/conf/my-application-conf.json`以匹配以下内容:

```
{
  "HTTP_PORT": 8082,

  "url": "jdbc:postgresql://localhost:5432/my_read_list",
  "driver_class": "org.postgresql.Driver",
  "user": "user",
  "password": "password"
}
```

我们添加了`url`、`driver_class`、`user`和`password`条目。请注意，如果您使用不同的数据库，配置可能会有所不同。

现在配置已经完成，我们需要创建一个 JDBC 客户端的实例。在`MyFirstVerticle`类中，声明一个新字段`JDBCClient jdbc;`，并将`start`方法的结尾更新为:

```
ConfigRetriever retriever = ConfigRetriever.create(vertx);
retriever.getConfig(
  config -> {
    if (config.failed()) {
      fut.fail(config.cause());
    } else {
      // Create the JDBC client
      jdbc = JDBCClient.createShared(vertx, config.result(), 
        "My-Reading-List");
      vertx
        .createHttpServer()
        .requestHandler(router::accept)
        .listen(
          // Retrieve the port from the configuration,
          // default to 8080.
          config.result().getInteger("HTTP_PORT", 8080),
          result -> {
            if (result.succeeded()) {
              fut.complete();
            } else {
              fut.fail(result.cause());
            }
        });
    }
  }
);
```

好了，我们已经用我们的配置配置了*客户端*，我们需要一个到数据库的连接。这是使用`jdbc.getConnection`方法实现的，该方法将其结果(连接)提供给`Handler<AsyncResult>`。当与数据库的连接建立时，或者如果在这个过程中发生了不好的事情，这个处理程序会得到通知。虽然我们可以直接使用该方法，但是让我们将连接的检索提取到一个单独的方法，并返回一个`Future`:

```
private Future connect() {
  Future future = Future.future();      // 1
  jdbc.getConnection(ar ->                             // 2
      future.handle(ar.map(connection ->               // 3
        connection.setOptions(
          new SQLOptions().setAutoGeneratedKeys(true)) // 4
      )
    )
  );
  return future;                                       // 5
}
```

让我们更深入地研究一下这个方法。首先我们创建一个`Future`对象(1 ),在方法(5)结束时返回。这个`Future`的完成或失败取决于我们是否成功地检索到数据库的连接。这是在(2)中完成的。我们传递给`getConnection`的函数接收一个`AsyncResult`。`Future`有一个方法(`handle`)根据一个`AsyncResult`直接完成或失败。到`handle`相当于:

```
if (ar.failed()) {
  future.failed(ar.cause());
} else {
  future.complete(ar.result());
}
```

仅仅...更短。

然而，在将`AsyncResult`传递给`future`之前，我们想要配置连接来启用密钥生成。为此，我们使用了`AsyncResult.map`方法。该方法基于给定的实例创建另一个`AsyncResult`实例，并对结果应用映射函数。如果给定的封装了一个失败，那么创建的封装了相同的失败。如果输入成功，映射函数将应用于结果。

## 我们需要文章

现在我们有了一个 JDBC 客户端，并且有了检索数据库连接的方法，接下来是插入文章的时候了。但是因为我们使用关系数据库，我们首先需要创建表。创建以下方法:

```
private Future createTableIfNeeded(SQLConnection connection) {
    Future future = Future.future();
    vertx.fileSystem().readFile("tables.sql", ar -> {
        if (ar.failed()) {
            future.fail(ar.cause());
        } else {
            connection.execute(ar.result().toString(),
                ar2 -> future.handle(ar2.map(connection))
            );
        }
    });
    return future;
}
```

该方法还返回一个`Future`。细心的读者会发现这是我们可以在`Future.compose`结构中使用的典型方法。这个方法体非常简单。像往常一样，我们创建一个`Future`，并在主体的末尾返回它。然后，我们读取`tables.sql`文件的内容，并执行该文件中包含的唯一语句。`execute`方法将 SQL 语句作为参数，并使用结果调用给定的函数。在处理程序中，我们使用`handle`方法完成或失败未来。在这种情况下，我们希望用数据库连接来完成未来。

所以，我们需要`tables.sql`文件。创建包含以下内容的`src/main/resources/tables.sql`文件:

```
CREATE TABLE IF NOT EXISTS Articles (id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    url VARCHAR(200) NOT NULL)
```

好了，现在我们有了到数据库和表的连接。让我们插入文章，但前提是数据库为空。为此，创建`createSomeDataIfNone`和`insert`方法:

```
private Future createSomeDataIfNone(SQLConnection connection) {
  Future future = Future.future();
  connection.query("SELECT * FROM Articles", select -> {
    if (select.failed()) {
      future.fail(select.cause());
    } else {
      if (select.result().getResults().isEmpty()) {
        Article article1 = new Article("Fallacies of distributed computing",
            "https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing");
        Article article2 = new Article("Reactive Manifesto",
            "https://www.reactivemanifesto.org/");
        Future

 insertion1 = insert(connection, article1, false);
        Future

 insertion2 = insert(connection, article2, false);
        CompositeFuture.all(insertion1, insertion2)
            .setHandler(r -> future.handle(r.map(connection)));
        } else {
          // Boring... nothing to do.
          future.complete(connection);
        }
    }
  });
  return future;
}

private Future

 insert(SQLConnection connection, Article article,
  boolean closeConnection) {
  Future

 future = Future.future();
  String sql = "INSERT INTO Articles (title, url) VALUES (?, ?)";
  connection.updateWithParams(sql,
    new JsonArray().add(article.getTitle()).add(article.getUrl()),
    ar -> {
      if (closeConnection) {
          connection.close();
      }
      future.handle(
          ar.map(res -> new Article(res.getKeys().getLong(0), 
              article.getTitle(), article.getUrl()))
      );
    }
  );
  return future;
}
```

让我们从结尾和`insert`方法开始。它遵循相同的模式，并使用`updateWithParams`方法将一篇文章插入数据库。SQL 语句包含使用 JSON 数组注入的参数。请注意，参数的顺序很重要。当插入完成时(在处理程序中)，如果请求的话，我们关闭连接(`closeConnection`参数)——这是因为我们以后要重用方法。最后，我们用一个包含生成的 id 的新的`Article`来完成或者失败`future`。因此，如果插入失败，我们只是将失败转发到未来。如果插入成功，我们将它映射到一个`Article`，并用这个值完成未来。

好了，我们切换到`createSomeDataIfNone`方法。同样的模式。但是这里我们需要一点协调。事实上，我们需要先检查数据库是否为空，如果是，插入两篇文章。为了检查数据库是否为空，我们使用`connection.query`检索所有文章。如果结果不为空，我们创建两篇文章，并使用`insert`方法插入。为了执行这两个插入，我们使用了`CompositeFuture`构造。所以这两个动作是同时执行的，当两个都完成(或者一个失败)时，处理程序被调用。请注意，连接没有关闭。

## 把这些碎片拼在一起

是时候把这些零件组装起来，看看效果如何了。需要更新`start`方法来执行以下操作:

1.  检索配置(已经完成)。
2.  检索配置后，创建 JDBC 客户端(已经完成)。
3.  检索到数据库的连接。
4.  通过这种连接，如果它们不存在，则创建表。
5.  用同样的连接，检查数据库是否包含文章，如果没有，插入一些数据。
6.  关闭连接。
7.  启动 HTTP 服务器，因为我们准备好了*服务。*
8.  向`fut`报告引导过程的成功或失败。

哇...动作真多。幸运的是，我们已经以一种可以使用`Future` composition 的方式实现了几乎所有需要的方法。在`start`方法中，将代码的结尾替换为:

```
// Start sequence:
// 1 - Retrieve the configuration
//  |- 2 - Create the JDBC client
//  |- 3 - Connect to the database (retrieve a connection)
//          |- 4 - Create table if needed
//               |- 5 - Add some data if needed
//                      |- 6 - Close connection when done
//          |- 7 - Start HTTP server
//  |- 8 - we are done!
ConfigRetriever.getConfigAsFuture(retriever)
  .compose(config -> {
    jdbc = JDBCClient.createShared(vertx, config, 
      "My-Reading-List");
    return connect()
      .compose(connection -> {
        Future future = Future.future();
        createTableIfNeeded(connection)
        .compose(this::createSomeDataIfNone)
        .setHandler(x -> {
            connection.close();
            future.handle(x.mapEmpty());
         });
         return future;
      })
      .compose(v -> createHttpServer(config, router));
    }).setHandler(fut);
```

不要担心`createHttpServer`方法。我们将很快介绍它。代码首先检索配置并创建`JDBCClient`。然后，我们检索一个数据库连接并初始化我们的数据库。请注意，连接在所有情况下都是关闭的(即使失败)。当数据库建立后，我们启动 HTTP 服务器。最后，当一切都完成后，我们向`fut`报告结果(成功或失败),告诉 Vert.x 我们是否准备好工作了。

*关于关闭连接的注意事项*:完成后不要忘记关闭 SQL 连接。该连接将被交还给连接池并被回收。

`createHTTPServer`方法非常简单，遵循相同的模式:

```
private Future createHttpServer(JsonObject config, Router router) {
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

注意`mapEmpty`。该方法返回一个`Future`，因为我们不关心 HTTP 服务器。要从一个`AsyncResult`创建一个`AsyncResult`，使用`mapEmpty`方法，丢弃封装的结果。

## 在 JDBC 之上实现 REST API

至此，我们已经设置好了一切，但是我们的 API 仍然依赖于我们的内存后端。是时候在 JDBC 之上重新实现我们的 REST API 了。但是首先，我们需要一些关注与数据库交互的实用方法。提取这些方法是为了便于理解。

首先，让我们添加`query`方法:

```
private Future<List

> query(SQLConnection connection) {
    Future<List

> future = Future.future();
    connection.query("SELECT * FROM articles", result -> {
            connection.close();
            future.handle(
                result.map(rs -> 
                  rs.getRows().stream()
                    .map(Article::new)
                    .collect(Collectors.toList()))
            );
        }
    );
    return future;
}
```

这个方法再次使用相同的模式:它创建一个`Future`对象并返回它。当基础动作完成或失败时，未来完成或失败。这里的操作是一个数据库查询。该方法执行查询，并在成功后，为每个*行*创建一个新的`Article`。另外，请注意，无论查询成功与否，我们都会关闭连接。重要的是释放连接，这样可以回收。

同样，让我们实现`queryOne`:

```
private Future

 queryOne(SQLConnection connection, 
  String id) {
  Future

 future = Future.future();
  String sql = "SELECT * FROM articles WHERE id = ?";
  connection.queryWithParams(sql, 
    new JsonArray().add(Integer.valueOf(id)),
    result -> {
        connection.close();
        future.handle(
          result.map(rs -> {
            List rows = rs.getRows();
            if (rows.size() == 0) {
              throw new NoSuchElementException(
                "No article with id " + id);
            } else {
              JsonObject row = rows.get(0);
              return new Article(row);
            }
          })
      );
  });
  return future;
}
```

这个方法使用`queryWithParams`在查询中注入文章 id。在结果处理程序中，还需要做一些工作，因为我们需要检查文章是否被找到。如果没有，我们抛出一个会使`future`失败的`NoSuchElementException`。这让我们可以生成`404`响应。

我们已经完成了查询，我们需要更新和删除的方法。他们在这里:

```
private Future update(SQLConnection connection, 
    String id, Article article) {
  Future future = Future.future();
  String sql = "UPDATE articles SET title = ?, url = ? WHERE id = ?";
  connection.updateWithParams(sql, 
    new JsonArray().add(article.getTitle())
      .add(article.getUrl())
      .add(Integer.valueOf(id)
    ),
    ar -> {
      connection.close();
      if (ar.failed()) {
        future.fail(ar.cause());
      } else {
        UpdateResult ur = ar.result();
        if (ur.getUpdated() == 0) {
           future.fail(new NoSuchElementException(
           "No article with id " + id));
        } else {
           future.complete();
        }
     }
  });
  return future;
}

private Future delete(SQLConnection connection, 
  String id) {
  Future 
```

它们非常相似，遵循相同的模式(再次！).

这很好，但是它没有实现我们的 REST API。所以，现在让我们专注于此。只是为了刷新我们的思维，以下是我们需要更新的方法:

*   `getAll`返回所有文章。
*   `addOne`插入新文章。请求正文中给出了文章详细信息。
*   `deleteOne`删除特定的文章。id 作为一个*路径参数*给出。
*   `getOne`提供特定文章的 JSON 表示。id 作为一个*路径参数*给出。
*   `updateOne`更新特定文章。id 作为一个*路径参数*给出。新的详细信息在请求正文中。

因为我们已经用数据库自己的方法提取了数据库交互，所以实现这个方法很简单。例如，`getAll`方法是:

```
private void getAll(RoutingContext rc) {
    connect()
        .compose(this::query)
        .setHandler(ok(rc));
}
```

我们使用`connect`方法检索一个连接。然后我们用`query`方法组合(顺序组合)它，并附加一个处理程序。这个处理程序是在`ActionHelper`类中提供的`ok(rc)`。它主要提供 JSON 表示或者管理错误响应(`500`、`404`)。

按照相同的模式，其他方法实现如下:

```
private void addOne(RoutingContext rc) {
  Article article = rc.getBodyAsJson().mapTo(Article.class);
  connect()
    .compose(connection -> insert(connection, article, true))
    .setHandler(created(rc));
}

private void deleteOne(RoutingContext rc) {
  String id = rc.pathParam("id");
  connect()
    .compose(connection -> delete(connection, id))
    .setHandler(noContent(rc));
}

private void getOne(RoutingContext rc) {
  String id = rc.pathParam("id");
  connect()
    .compose(connection -> queryOne(connection, id))
    .setHandler(ok(rc));
}

private void updateOne(RoutingContext rc) {
  String id = rc.request().getParam("id");
  Article article = rc.getBodyAsJson().mapTo(Article.class);
  connect()
    .compose(connection ->  update(connection, id, article))
    .setHandler(noContent(rc));
}
```

## 测试，测试，再测试

如果我们现在运行应用程序测试，它会失败。首先，我们需要更新配置以传递 JDBC URL 和相关细节。但是等等...我们还需要一个数据库。我们不一定要在单元测试中使用 PostgreSQL。让我们使用内存数据库 HSQL。为此，我们首先需要在`pom.xml`中添加以下依赖项:

```
org.hsqldb
    hsqldb
    2.4.0
    test
```

但是等等，如果你已经使用了 JDBC 或者数据库，你会知道每个数据库使用不同的方言(这就是标准的力量)。这里，我们不能使用相同的表创建语句，因为 HSQL 不理解 PostgreSQL 方言。因此创建具有以下内容的`src/test/resources/tables.sql`:

```
CREATE TABLE IF NOT EXISTS Articles (id INTEGER IDENTITY,
    title VARCHAR(200),
    url VARCHAR(200))
```

这是 HSQL 方言中的等价语句。这怎么可能呢？当 Vert.x 读取一个文件时，它也会检查*类路径*(并且`src/test/resources`包含在*测试类路径*中)。运行测试时，这个文件取代了我们创建的初始文件。

我们需要稍微更新我们的测试来配置`JDBCClient`。在`MyFirstVerticleTest`类中，将在`setUp`方法中创建的`DeploymentOption`对象更改为:

```
DeploymentOptions options = new DeploymentOptions()
    .setConfig(new JsonObject()
        .put("HTTP_PORT", port)
        .put("url", "jdbc:hsqldb:mem:test?shutdown=true")
        .put("driver_class", "org.hsqldb.jdbcDriver")
);
```

除了`HTTP_PORT`，我们还放了 JDBC 的 url 和 JDBC 驱动的类。

现在，您应该能够使用:`mvn clean test`运行测试了。

## 表演时间

这次我们想使用 PostgreSQL 实例。我将使用 docker，但使用您最喜欢的方法。使用 docker，我如下启动我的实例:

```
docker run --name some-postgres -e POSTGRES_USER=user \
    -e POSTGRES_PASSWORD=password \
    -e POSTGRES_DB=my_read_list \
    -p 5432:5432 -d postgres
```

现在让我们运行我们的应用程序:

```
mvn compile vertx:run
```

打开浏览器到 http://localhost:8082/assets/index . html，你
应该会看到使用数据库的应用程序。这一次，产品存储在持久存储在文件系统上的数据库中。因此，如果我们停止并重新启动应用程序，数据就会恢复。

如果你想打包应用程序，运行`mvn clean package`。然后使用以下命令运行应用程序:

```
java -jar target/my-first-app-1.0-SNAPSHOT.jar \
    -conf src/main/conf/my-application-conf.json
```

## 结论

这是我们系列的第四篇文章，讨论了两个主题。首先，我们介绍了异步合成以及`Future`如何帮助管理顺序和并发合成。使用`Future`，您在实现中遵循一个通用的模式，一旦您掌握了它，这是非常简单的。其次，我们已经看到了如何使用 JDBC 来实现我们的 API。因为我们使用`Future`，使用异步 JDBC 相当简单。

你可能会对异步开发模式感到惊讶，但是一旦你开始使用它，就很难再回来。异步和事件驱动的架构代表了我们周围世界的工作方式。拥抱这些会给你超能力。

在下一篇文章中，我们将看到如何使用 [RX Java 2 来代替未来的](https://developers.redhat.com/blog/2018/04/18/eclipse-vertx-reactive-extensions/)。不要忘记代码可以在这个 [Github 库中找到。](https://github.com/redhat-developer/introduction-to-eclipse-vertx)

敬请关注，祝编码愉快！

*Last updated: April 20, 2018*