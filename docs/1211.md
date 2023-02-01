# Eclipse Vert.x 应用程序配置(Vert.x 介绍的第 2 部分)

> 原文：<https://developers.redhat.com/blog/2018/03/22/eclipse-vert-x-application-configuration>

在我之前的文章[Eclipse Vert.x 简介](https://developers.redhat.com/blog/2018/03/13/eclipse-vertx-first-application/)中，我们开发了一个非常简单的 vert . x 应用程序，并了解了如何测试、打包和执行这个应用程序。那很好，不是吗？嗯，这只是开始。在这篇文章中，我们将增强我们的应用程序以支持外部配置，并学习如何处理不同的配置源。

提醒您一下，我们有一个应用程序在端口`8080`上启动一个 HTTP 服务器，并以礼貌的“Hello”消息回复所有 HTTP 请求。上一篇文章的代码可以在[https://github . com/red hat-developer/introduction-to-eclipse-vertx](https://github.com/redhat-developer/introduction-to-eclipse-vertx)的`post-1`目录中找到。这篇文章中开发的代码在`post-2`目录中。

## 那么，我们为什么需要配置呢？

这个问题问得好。应用程序现在可以工作了，但是假设您想将它部署在一台已经占用了端口`8080`的机器上。我们需要在应用程序代码和测试中改变端口，仅仅是为了这台机器。那就太可悲了。幸运的是，Vert.x 应用程序是可配置的。

有几种方法可以配置 Vert.x 应用程序:

1.  使用简单的 JSON 文件。
2.  使用垂直 x 配置。

在这两种情况下，应用程序代码都将配置作为`JsonObject`来操作。

当使用简单的 JSON 文件时，verticle 接收配置。可以将配置传递给命令行或使用 API。让我们看一看。

## 不再有“8080”了

第一步是修改`io.vertx.intro.first.MyFirstVerticle`类，使其不绑定到端口`8080`，而是从配置中读取它:

```
package io.vertx.intro.first;

import io.vertx.core.AbstractVerticle;
import io.vertx.core.Future;

public class MyFirstVerticle extends AbstractVerticle {

  @Override
  public void start(Future fut) {
    vertx
      .createHttpServer()
      .requestHandler(r ->
        r.response()
            .end("</pre>
<h1>Hello from my first " + "Vert.x application</h1>
<pre>
"))
      .listen(config()
        .getInteger("HTTP_PORT", 8080), 
      result -> {
        if (result.succeeded()) {
            fut.complete();
        } else {
            fut.fail(result.cause());
        }
      });
    }
}
```

所以，和之前版本唯一的区别就是`config().getInteger("HTTP_PORT", 8080)`。这里，我们的代码现在请求配置并检查是否设置了`HTTP_PORT`属性。如果没有，港口`8080`被用作后备。检索到的配置是一个`JsonObject`。

由于我们默认使用端口`8080`,您仍然可以打包我们的应用程序并像以前一样运行它:

```
mvn clean package
java -jar target/my-first-app-1.0-SNAPSHOT.jar
```

简单吧？

## 基于 API 的配置-用于测试的随机端口

既然应用程序是可配置的，让我们试着提供一个配置。在我们的测试中，我们将配置我们的应用程序来使用端口`8081`。以前，我们通过以下方式部署我们的垂直市场:

```
vertx.deployVerticle(MyFirstVerticle.class.getName(), 
  context.asyncAssertSuccess());
```

现在让我们传递一些部署选项:

```
private Vertx vertx;
// New field storing the port.
private int port = 8081;

@Before
public void setUp(TestContext context) {
  vertx = Vertx.vertx();
  // Create deployment options with the chosen port
  DeploymentOptions options = new DeploymentOptions()
      .setConfig(new JsonObject().put("HTTP_PORT", port));
  // Deploy the verticle with the deployment options
  vertx.deployVerticle(MyFirstVerticle.class.getName(), 
    options, context.asyncAssertSuccess());
}
```

`DeploymentOptions`对象让我们定制各种参数。特别是，它让我们在使用`config()`方法时注入由 verticle 检索的`JsonObject`。

显然，连接到服务器的测试需要稍加修改才能使用正确的端口(端口是一个字段):

```
vertx.createHttpClient().getNow(port, "localhost", 
  "/", response -> {
    response.handler(body -> {
      context.assertTrue(body.toString()
        .contains("Hello"));
      async.complete();
    });
});
```

这并没有真正解决我们的问题。当端口`8081`也被使用时会发生什么？现在让我们随机选择一个端口:

```
// Pick an available and random
ServerSocket socket = new ServerSocket(0);
port = socket.getLocalPort();
socket.close();

DeploymentOptions options = new DeploymentOptions()
            .setConfig(new JsonObject()
              .put("HTTP_PORT", port));
vertx.deployVerticle(MyFirstVerticle.class.getName(), 
     options, context.asyncAssertSuccess());
```

所以，想法很简单。我们打开一个服务器套接字，它将随机选择一个端口(这就是为什么我们将`0`放在`ServerSocket`参数中)。我们检索使用的端口并关闭套接字。请注意，这种方法并不完美，如果选择的端口被用在`close`方法和我们的 HTTP 服务器之间，这种方法可能会失败。然而，在大多数情况下，它应该工作得很好。

有了这些，我们的测试现在使用一个随机端口。通过以下方式执行它们:

```
mvn clean test
```

## 外部配置-让我们在另一个端口上运行

好吧，随机端口不是我们在生产中想要的。如果您告诉运营团队您的应用程序正在选择一个随机端口，您能想象他们的表情吗？这实际上可能很有趣，但我们永远不应该和行动小组发生冲突。

对于应用程序的实际执行，让我们将配置传递到一个外部文件中。配置存储在一个 JSON 文件中。

用以下内容创建`src/main/conf/my-application-conf.json`:

```
{
  "HTTP_PORT" : 8082
}
```

现在，要使用此配置，只需使用以下命令启动您的应用程序:

```
java -jar target/my-first-app-1.0-SNAPSHOT.jar \
  -conf src/main/conf/my-application-conf.json
```

在 http://localhost:8082 上打开浏览器，就到了！

这是怎么回事？我们的 fat jar 正在使用`Launcher`类(由 Vert.x 提供)来启动我们的应用程序。这个类在部署我们的 verticle 时读取`-conf`参数并创建相应的部署选项。

## 12 因素应用和其他配置存储

虽然将配置存储在 JSON 文件中非常方便，但它并不总是符合要求。例如，如果你遵循 [12 因子应用](https://12factor.net/config)原则，它建议应用程序读取*环境变量*作为配置。领事或者金库储存*机密*怎么办？为了处理所有这些情况，Vert.x 提供了一个方便的模块:`vertx-config`。在本节中，我们将改变从环境变量、系统属性以及最后提供的配置文件中检索 HTTP 端口的方式。

首先，将以下依赖项添加到您的`pom.xml`文件中:

```
io.vertx
  vertx-config
  ${vertx.version}
```

在 verticle 类中，将`start`方法的内容更新为:

```
@Override
public void start(Future fut) {
  ConfigRetriever retriever = ConfigRetriever.create(vertx);
  retriever.getConfig(
      config -> {
        if (config.failed()) {
            fut.fail(config.cause());
        } else {
            vertx
                .createHttpServer()
                .requestHandler(r ->
                    r.response().end(
                      "</pre>
<h1>Hello from my first " + "Vert.x application</h1>
<pre>
"))
                .listen(config
                   .result().getInteger("HTTP_PORT", 8080),
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
}
```

`vertx-config`模块提供了`ConfigRetriever`。这个对象负责检索不同的配置块并计算最终的配置。因为这个过程是异步的，所以结果被传递给一个处理程序，该处理程序执行启动逻辑的其余部分。

准备就绪后，现在可以从 3 个不同的位置选择端口:

*   配置文件，如前所述(使用`-conf`)。
*   系统属性。例如，用`-DHTTP_PORT=8081`启动应用程序以使用端口 8081。
*   环境属性。例如，使用以下命令启动应用程序:

```
export HTTP_PORT=8081
java -jar target/my-first-app-1.0-SNAPSHOT.jar
```

`vertx-config`提出了更多的功能和配置存储。查看它的[文档](http://vertx.io/docs/vertx-config/java/)。

## 结论

在开发了您的第一个 Vert.x 应用程序之后，我们已经看到了这个应用程序是如何配置的。这并没有增加我们应用程序的复杂性。在下一篇文章中，我们将会看到如何使用`vertx-web`来开发一个服务于静态页面和 REST API 的小应用。有点花哨，但仍然很简单。

如果你渴望看到更多，请访问 Eclipse Vert.x 网站。如果你现在想在 OpenShift 上部署一个 Vert.x 应用，可以查看[http://launch . open shift . io](https://launch.openshift.io)。

祝您编码愉快，敬请关注我的下一篇 Vert.x 文章！

*Last updated: April 17, 2018*