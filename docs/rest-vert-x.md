# 有些与 Vert.x 有关(Vert.x 介绍的第 3 部分)

> 原文：<https://developers.redhat.com/blog/2018/03/29/rest-vert-x>

这篇文章是关于[Eclipse vert . x](https://vertx.io/)T3*介绍系列的第三篇。所以，我们来快速回顾一下之前帖子的内容。在[的第一篇文章](https://developers.redhat.com/blog/2018/03/13/eclipse-vertx-first-application/)中，我们开发了一个非常简单的 Eclipse Vert.x 应用程序，并看到了如何测试、打包和执行这个应用程序。在第二篇文章中，我们看到了这个应用程序如何变得可配置，以及我们如何在测试中使用随机端口。*

嗯，没什么特别的...这次让我们更进一步，开发一个粗糙的/ REST 式的应用程序。因此，一个应用程序使用 REST API 公开一个与后端交互的 HTML 页面。API 的 RESTfulness 级别不是本文的主题；我让你来决定，因为这是一个非常棘手的话题。

换句话说，我们将会看到:

*   这是一个让你使用 Vert.x 轻松创建网络应用的框架。
*   如何公开静态资源？
*   如何开发一个 REST API？

这篇文章中开发的代码可以在[https://github . com/red hat-developer/introduction-to-eclipse-vertx](https://github.com/redhat-developer/introduction-to-eclipse-vertx)资源库的`post-3`目录中找到。我们将从`post-2`代码库开始。

那么，我们开始吧。

## 绿色. x Web

正如您在之前的帖子中可能已经注意到的，只使用`Vert.x Core`处理复杂的 HTTP 应用程序有点麻烦。这就是`Vert.x Web`背后的主要原因。它使 Web 应用程序的开发变得非常容易，而无需改变理念。

要使用 Vert.x Web，您需要更新`pom.xml`文件以添加以下依赖项:

```
io.vertx
  vertx-web
  ${vertx.version}
```

这是你使用 Vert.x Web 唯一需要的东西。很甜蜜，不是吗？

让我们现在使用它。记住，在之前的帖子中，当我们请求`http://localhost:8080`时，我们回复了一条很好的 *Hello World* 消息。让我们用 Vert.x Web 做同样的事情。打开`io.vertx.intro.first.MyFirstVerticle`类并将启动方法改为:

```
@Override
public void start(Future fut) {
  // Create a router object.
  Router router = Router.router(vertx);

  // Bind "/" to our hello message - so we are 
  // still compatible with out tests.
  router.route("/").handler(rc -> {
      HttpServerResponse response = rc.response();
      response
          .putHeader("content-type", "text/html")
          .end("</pre>
<h1>Hello from my first Vert.x 3 app</h1>
<pre>
");
  });

  ConfigRetriever retriever = ConfigRetriever.create(vertx);
  retriever.getConfig(
      config -> {
          if (config.failed()) {
              fut.fail(config.cause());
          } else {
              // Create the HTTP server and pass the 
              // "accept" method to the request handler.
              vertx
                  .createHttpServer()
                  .requestHandler(router::accept)
                  .listen(
                      // Retrieve the port from the 
                      // configuration, default to 8080.
                      config().getInteger("HTTP_PORT", 8080),
                      result -> {
                          if (result.succeeded()) {
                              fut.complete();
                          } else {
                              fut.fail(result.cause());
                          }
                      }
                  );
          }
      }
  );
}
```

您可能会对这段代码的长度感到惊讶(与前面的代码相比)。但是正如我们将要看到的，这将使我们的应用程序看起来像是服用了类固醇，所以请耐心等待。

如你所见，我们从创建一个`Router`对象开始。路由器是 Vert.x Web 的基石。这个对象负责将 HTTP 请求分派给右边的*T2 处理程序。另外两个概念在 Vert.x Web 中非常重要:*

*   路由——允许您定义如何分派请求。
*   处理程序——处理请求和写入结果的实际操作。处理程序可以被链接。

如果你理解了这三个概念，你就理解了 Vert.x Web 中的一切。

让我们首先关注这个片段:

```
router.route("/").handler(rc -> {
  HttpServerResponse response = rc.response();
  response
      .putHeader("content-type", "text/html")
      .end("</pre>
<h1>Hello from my first Vert.x 3 app</h1>
<pre>
");
});
```

它将到达`/`的请求路由到给定的处理程序。处理程序接收一个`RoutingContext`对象。这个处理程序与我们之前的代码非常相似，这很正常，因为它处理相同类型的对象:`HttpServerResponse`。

现在让我们来看看代码的其余部分:

```
//...
vertx
    .createHttpServer()
    .requestHandler(router::accept)
    .listen(
        config().getInteger("HTTP_PORT", 8080),
        result -> {
          if (result.succeeded()) {
            fut.complete();
          } else {
            fut.fail(result.cause());
          }
        }
    );
}
```

除了我们更改了请求处理程序之外，代码基本上和以前一样。我们将`router::accept`传递给处理程序。你可能不熟悉这个符号。这是对一个方法的引用(这里是来自`router`对象的方法`accept`)。换句话说，它指示 vert.x 在收到请求时调用路由器的 accept 方法。

让我们试着看看这是否可行:

```
mvn clean package
java -jar target/my-first-app-1.0-SNAPSHOT.jar
```

在你的浏览器中打开`http://localhost:8080`，你应该会看到*你好*的消息。由于我们没有改变应用程序的行为，我们的测试仍然有效。

## 公开静态资源

好了，我们有了第一个使用 vert.x web 的应用程序。让我们来看看一些好处。让我们从服务静态资源开始，比如一个`index.html`页面。在我们进一步讨论之前，我应该首先声明:“我们在这里将要看到的 HTML 页面丑得要命:我不是一个 UI 家伙”。我还应该补充一点，可能有很多更好的方法来实现这一点，我应该学习无数的框架，但这不是重点。我试图保持事情简单，文章只依赖 JQuery 和 Bootstrap，所以如果你懂一点 JavaScript，你可以理解和编辑页面。

让我们创建一个 HTML 页面，作为应用程序的入口点。在`src/main/resources/assets`中创建一个`index.html`页面，内容来自[这里](https://raw.githubusercontent.com/redhat-developer/introduction-to-eclipse-vertx/master/post-3/src/main/resources/assets/index.html)。因为它只是一个带有一点 JavaScript 的 HTML 页面，所以我们不会在这里详述内容。

基本上，这个页面是一个简单的 CRUD UI，用来管理我尚未阅读的文章。它是以一种通用的方式制作的，所以你可以把它转换成你自己的东西。阅读列表显示在主表格中。您可以创建新的阅读列表、编辑列表或删除列表。这些动作依赖于通过 AJAX 调用的 REST API(我们将要实现)。仅此而已。

一旦创建了这个页面，编辑`io.vertx.blog.first.MyFirstVerticle`类并将`start`方法改为:

```
@Override
public void start(Future fut) {
  // Create a router object.
  Router router = Router.router(vertx);

  router.route("/").handler(rc -> {
    HttpServerResponse response = rc.response();
    response
        .putHeader("content-type", "text/html")
        .end("</pre>
<h1>Hello from my first Vert.x 3 app</h1>
<pre>
");
  });
  // Serve static resources from the /assets directory
  router.route("/assets/*")
    .handler(StaticHandler.create("assets"));

  ConfigRetriever retriever = ConfigRetriever.create(vertx);
  retriever.getConfig(
    config -> {
      if (config.failed()) {
        fut.fail(config.cause());
      } else {
        // Create the HTTP server and pass 
        // the "accept" method to the request
        // handler.
        vertx
            .createHttpServer()
            .requestHandler(router::accept)
            .listen(
              // Retrieve the port from the 
              // config, default to 8080.
              config().getInteger("HTTP_PORT", 8080),
              result -> {
                if (result.succeeded()) {
                    fut.complete();
                } else {
                    fut.fail(result.cause());
                }
              }
            );
      }
    }
  );
}
```

与前面代码的唯一区别是`router.route("/assets/*").handler(StaticHandler.create("assets"));`行。那么，这条线是什么意思？其实挺简单的。它将`/assets/*`上的请求路由到存储在`assets`目录中的资源。所以我们的`index.html`页面将使用`http://localhost:8080/assets/index.html`来提供服务。

所以，我相信你已经迫不及待地想看我们漂亮的 HTML 页面了。让我们构建并运行应用程序:

```
mvn clean package
java -jar target/my-first-app-1.0-SNAPSHOT.jar
```

现在，打开你的浏览器到`http://localhost:8080/assets/index.html`。它在这里...
丑吧？我告诉过你。

你可能也注意到了，桌子是空的。这是因为我们还没有实现 REST API。让我们现在做那件事。

## 带有 Vert.x Web 的 REST API

Vert.x Web 使 REST API 的实现变得非常容易，因为它基本上将您的 URL 路由到正确的处理程序。API 非常简单，其结构如下:

*   `GET /api/articles` = >获取所有文章(`getAll`)。
*   `GET /api/articles/:id` = >获取对应 id 的文章(`getOne`)。
*   `POST /api/articles` = >新增一条(`addOne`)。
*   `PUT /api/articles/:id` = >更新一篇文章(`updateOne`)。
*   `DELETE /api/articles/id` = >删除一篇文章(`deleteOne`)。

### 我们需要一些数据...

但是在继续之前，让我们创建我们的*数据对象；*代表一个`Article`的类。创建包含以下内容的`src/main/java/io/vertx/intro/first/Article.java`:

```
package io.vertx.intro.first;

import java.util.concurrent.atomic.AtomicInteger;

public class Article {

  private static final AtomicInteger COUNTER
    = new AtomicInteger();

  private final int id;

  private String title;

  private String url;

  public Article(String title, String url) {
      this.id = COUNTER.getAndIncrement();
      this.title = title;
      this.url = url;
  }

  public Article() {
      this.id = COUNTER.getAndIncrement();
  }

  public int getId() {
      return id;
  }

  public String getTitle() {
      return title;
  }

  public Article setTitle(String title) {
      this.title = title;
      return this;
  }

  public String getUrl() {
      return url;
  }

  public Article setUrl(String url) {
      this.url = url;
      return this;
  }
}
```

这是一个非常简单的 *bean* 类(getters 和 setters 也是如此)。我们选择这种格式是因为 Vert.x 依赖于 Jackson 将对象映射到 JSON 或从 JSON 映射。

现在，让我们创建几篇文章。在`MyFirstVerticle`类中，添加以下代码:

```
// Store our readingList
private Map readingList = new LinkedHashMap();
// Create a readingList
private void createSomeData() {
    Article article1 = new Article(
        "Fallacies of distributed computing", 
        "https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing");
    readingList.put(article1.getId(), article1);
    Article article2 = new Article(
        "Reactive Manifesto", 
        "https://www.reactivemanifesto.org/");
    readingList.put(article2.getId(), article2);
}
```

然后，在`start`方法中，调用`createSomeData`方法:

```
@Override
public void start(Future fut) {

  createSomeData();

  // Create a router object.
  Router router = Router.router(vertx);

  // Rest of the method
}
```

正如你已经注意到的，我们在这里没有真正的后端；这只是一个内存中的地图。添加后端将在另一篇文章中讨论。

### 获取我们的阅读清单

够装修；让我们实现 REST API。我们将从`GET /api/articles`开始。它返回 JSON 数组中的文章列表。

在`start`方法中，将这一行添加到静态处理程序行的正下方:

```
router.get("/api/articles").handler(this::getAll);
```

这一行指示路由器通过调用 getAll 方法来处理`/api/articles`上的`GET`请求。我们可以内联处理程序代码，但是为了清楚起见，让我们创建另一个方法:

```
private void getAll(RoutingContext rc) {
  rc.response()
      .putHeader("content-type", 
         "application/json; charset=utf-8")
      .end(Json.encodePrettily(readingList.values()));
}
```

像每个路由处理器一样，我们的方法接收一个`RoutingContext`。我们通过设置`content-type`头和实际内容来填充响应。要创建实际的内容，不需要我们自己计算 JSON 字符串。Vert.x 提供了与 JSON 字符串相互映射的`Json`类对象。所以`Json.encodePrettily(readingList.values())`计算代表文章集的 JSON 字符串。

我们可以使用`Json.encodePrettily(readingList)`，但是为了使 JavaScript 代码更简单，我们只返回文章集，而不是包含`ID => Article`条目的对象。

有了这些，我们应该能够从 HTML 页面中检索文章集。让我们来试试:

```
mvn compile vertx:run
```

然后在浏览器中打开 HTML 页面到`http://localhost:8082/assets/index.html`，您应该会看到:

[![](img/e1b3bd69e0f1e91ff693af5cf5728b38.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/articles-list.png)

你可能会奇怪为什么端口是 8082？记住，在上一篇文章中，我们创建了`src/main/conf/my-application-conf.json`。默认情况下，Vert.x Maven 插件将该文件作为配置。

我肯定你很好奇，想实际看看我们的 REST API 返回了什么。让我们打开一个浏览器到`http://localhost:8082/api/articles`。您应该得到:

```
[ {
  "id" : 0,
  "title" : "Fallacies of distributed computing",
  "url" : "https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing"
}, {
  "id" : 1,
  "title" : "Reactive Manifesto",
  "url" : "https://www.reactivemanifesto.org/"
} ]
```

### 添加要阅读的内容

现在我们可以检索文章集，让我们创建一个新的。与前面的 REST API 端点不同，这个端点需要读取请求的主体。出于性能原因，应该显式启用它。不要害怕；这只是一个处理程序。

在 start 方法中，将这些行添加到以`getAll`结尾的行的正下方:

```
router.route("/api/articles*").handler(BodyHandler.create());
router.post("/api/articles").handler(this::addOne);
```

第一行允许读取`/api/articles`下所有路由的请求体。我们可以用`router.route().handler(BodyHandler.create())`在全球范围内启用它。

第二行将`/api/articles`上的 POST 请求映射到`addOne`方法。让我们创建这个方法:

```
private void addOne(RoutingContext rc) {
    Article article = rc.getBodyAsJson().mapTo(Article.class);
    readingList.put(article.getId(), article);
    rc.response()
        .setStatusCode(201)
        .putHeader("content-type", 
          "application/json; charset=utf-8")
        .end(Json.encodePrettily(article));
}
```

该方法首先从请求体创建一个`Article`实例。一旦创建完成，它会将文章添加到后端，并将创建的文章作为 JSON 返回。

让我们试试这个。如果您保持应用程序运行，只需点击`Add a new article`按钮。或者如果没有:`mvn compile vertx:run`。

输入数据，如:*用 Java 构建反应式微服务*作为标题，`https://developers.redhat.com/tions/building-reactive-microservices-in-java/`作为 url。点击`save`，文章应该会出现在列表中。

*地位 201？*
如您所见，我们已经将响应状态设置为`201`。它的意思是`CREATED`，通常用在创建实体的 REST API 中。默认情况下，vert.x web 将状态设置为`200`，即`OK`。

### 阅读完成

嗯，有时候你会花时间去读文章，所以我们应该可以把它从列表中删除。在`start`方法中，添加这一行:

```
router.delete("/api/articles/:id").handler(this::deleteOne);
```

在*路径*中，我们定义一个参数`:id`。因此，当处理匹配请求时，Vert.x 提取对应于参数的路径段，并让我们在 handler 方法中访问它。例如，`/api/articles/0`将`id`映射到`0`。

让我们看看如何在 handler 方法中使用该参数。如下创建`deleteOne`方法:

```
private void deleteOne(RoutingContext rc) {
    String id = rc.request().getParam("id");
    try {
        Integer idAsInteger = Integer.valueOf(id);
        readingList.remove(idAsInteger);
        rc.response().setStatusCode(204).end();
    } catch (NumberFormatException e) {
        rc.response().setStatusCode(400).end();
    }
}
```

使用`rc.request().getParam("id")`检索路径参数。在一个`try-catch`块中，我们试图将这个路径参数转换成整数。如果它失败了(用一个`NumberFormatException`，我们写一个`Bad Request - 400`响应。如果没问题，我们就把文章从后台删除。

*地位 204？*
如您所见，我们已经将响应状态设置为`204 - NO CONTENT`。对 HTTP 动词`DELETE`的响应通常没有内容。

### 其他方法

我们不会解释`getOne`和`updateOne`，因为它们的实现非常简单，而且非常相似。他们的实现可以在 GitHub 上找到。

## 并发

让我们谈一谈并发性。显然，使用内存后端并不适合生产环境，但它说明了 Vert.x 的一个关键特征。我们在这个后端上进行读写操作，而不使用任何同步结构。经验丰富的 Java 开发人员显然会对此感到恼火。

然而，Vert.x verticles 是单线程的。这意味着只有一个线程在访问它们，而且总是同一个线程。所以我们不需要同步，因为我们不能并发访问。太棒了，不是吗？但是我们如何处理并发的 HTTP 请求呢？这也很简单，每次都使用相同的线程。我们所做的一切都不会阻塞处理，对请求的响应也很快。因此，虽然我们不会在**相同的**时间处理另一个请求，但这并不意味着我们不能处理并发请求。他们只是在排队；但不会太久。如果您尝试执行并发请求(使用 Gatling 或`wrk`之类的工具),由于这种事件循环机制，您将实现非常好的响应时间。

### 摘要

是时候结束这篇帖子了。我们已经看到了 Vert.x Web 如何让您轻松实现 REST API，以及它如何服务于静态资源。比以前有点花哨，但仍然很容易。

在下一篇文章中，我们将使用 PostgreSQL 数据库作为后端。不要忘记代码可以在 Github 库中找到。

敬请期待，快乐编码！

*Last updated: April 7, 2022*