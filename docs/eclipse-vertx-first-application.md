# Eclipse Vert.x 简介——我的第一个 Vert.x 应用程序

> 原文：<https://developers.redhat.com/blog/2018/03/13/eclipse-vertx-first-application>

比方说，你听到有人说 [Eclipse Vert.x](http://vertx.io/) 很牛逼。好极了，但是你可能想自己试试。下一个逻辑问题是“我从哪里开始？”。这篇文章是一个很好的起点。它展示了:如何构建一个非常简单的 Vert.x 应用程序(没什么花哨的)，如何测试，以及如何打包和执行。基本上，在构建自己的突破性应用程序之前，您需要知道的一切。

本文中开发的代码可以在 GitHub 上获得。这是“Vert.x 系列介绍”的一部分。这篇文章的代码位于`post-1`目录下的[https://github . com/red hat-developer/introduction-to-eclipse-vertx](https://github.com/redhat-developer/introduction-to-eclipse-vertx)存储库中。

## 开始吧！

首先，让我们创建一个项目。在这篇文章中，我们使用了 [Apache Maven](https://maven.apache.org/) ，但是你也可以使用 [Gradle](https://gradle.org/) 或者你喜欢的构建过程工具。您可以使用 Maven jar 原型来创建结构，但是基本上您只需要一个目录，其中包含:

*   一个`src/main/java`目录
*   一个`src/test/java`目录
*   一个`pom.xml`文件

所以，你会得到这样的结果:

```
.
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   └── test
│       └── java 
```

让我们用以下内容创建`pom.xml`文件:

```
4.0.0

  io.vertx.intro
  my-first-app
  1.0-SNAPSHOT

    3.5.0
    1.0.13

      io.vertx
      vertx-core
      ${vertx.version}

        maven-compiler-plugin
        3.7.0

          1.8
          1.8

        io.fabric8
        vertx-maven-plugin
        ${vmp.version}

            vmp

              initialize
              package

          true
```

这个`pom.xml`文件非常简单:

*   它声明了对“vertx-core”的依赖，Vert.x 版本被声明为属性。
*   它将“maven 编译器插件”配置为使用 Java 8。
*   它声明了“vertx-maven-plugin ”;我们一会儿将回到这一点。

Vert.x 至少需要 Java 8，所以不要试图在 Java 6 或 7 的 JVM 上运行 Vert.x 应用；没用的。

`vertx-maven-plugin`是一个可选插件，它打包了你的应用并提供额外的功能(文档在这里是)。否则你可以使用`maven-shade-plugin`或任何其他打包插件。`vertx-maven-plugin`很方便，因为它打包应用程序时不需要任何配置。它还提供了一个*重新部署*特性，在您更新代码或资源时重新启动应用程序。

## 我们来编码吧！

好了，现在我们已经做好了`pom.xml`文件。让我们做一些真正的编码。用以下内容创建`src/main/java/io/vertx/intro/first/MyFirstVerticle.java`文件:

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
                 .end("
<h1>Hello from my first Vert.x application</h1>

"))
            .listen(8080, result -> {
                if (result.succeeded()) {
                    fut.complete();
                } else {
                    fut.fail(result.cause());
                }
            });
    }
}
```

这其实是我们*并不看中的*应用。该类扩展了`AbstractVerticle`。在 Vert.x 世界中，一个*垂直线*是一个组件。通过扩展`AbstractVerticle`，我们的类可以访问`vertx`字段，以及部署 verticle 的`vertx`实例。

部署 verticle 时会调用`start`方法。我们还可以实现一个`stop`方法(在取消部署 verticle 时调用),但是在这种情况下，Vert.x 会为我们处理垃圾。`start`方法接收一个`Future`对象，让我们在启动序列完成时通知 Vert.x，或者在发生任何不好的情况时报告一个错误。Vert.x 的一个特点是它的异步和非阻塞特性。当我们的 verticle 将要部署时，它不会等到`start`方法已经完成。因此，`Future`参数对于完成通知非常重要。注意，您也可以实现一个没有`Future`参数的`start`方法版本。在这种情况下，当`start`方法返回时，Vert.x 考虑部署的 verticle。

`start`方法创建一个 HTTP 服务器，并将一个*请求处理程序*附加到它上面。请求处理程序是一个函数(这里是 lambda ),它被传递到`requestHandler`方法中，并且在服务器每次收到请求时被调用。在这段代码中，我们只需回复 *Hello* (我没有告诉你任何花哨的内容)。最后，服务器被绑定到`8080`端口。由于这可能会失败(因为端口可能已经被使用)，我们传递另一个 lambda 表达式，调用结果(因此能够检查连接是否成功)。如上所述，如果成功，它会调用`fut.complete`，或者调用`fut.fail`来报告错误。

在尝试应用程序之前，编辑`pom.xml`文件并在属性中添加`vertx.verticle`条目:

```
3.5.0
    1.0.13
    <!-- line to add: -->
    io.vertx.intro.first.MyFirstVerticle
```

该属性指示 Vert.x 在启动时部署该类。

让我们尝试使用以下代码来编译应用程序:

```
mvn compile vertx:run
```

应用程序启动；打开浏览器到 [http://localhost:8080](http://localhost:8080) ，你应该会看到 *Hello* 消息。如果更改代码中的消息并保存文件，应用程序将使用更新后的消息重新启动。

点击`CTRL+C`停止应用程序。

申请到此为止。

## 让我们测试一下

嗯，开发一个应用程序是好事，但是我们不能太小心，所以让我们测试它。测试使用 JUnit 和`vertx-unit`——一个随 Vert.x 提供的框架，使 Vert.x 应用程序的测试更加自然。

打开`pom.xml`文件，添加以下两个依赖项:

```
junit
  junit
  4.12
  test

  io.vertx
  vertx-unit
  ${vertx.version}
  test
```

现在用以下内容创建`src/test/java/io/vertx/intro/first/MyFirstVerticleTest.java`:

```
package io.vertx.intro.first;

import io.vertx.core.Vertx;
import io.vertx.ext.unit.Async;
import io.vertx.ext.unit.TestContext;
import io.vertx.ext.unit.junit.VertxUnitRunner;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(VertxUnitRunner.class)
public class MyFirstVerticleTest {

    private Vertx vertx;

    @Before
    public void setUp(TestContext context) {
        vertx = Vertx.vertx();
        vertx.deployVerticle(MyFirstVerticle.class.getName(),
            context.asyncAssertSuccess());
    }

    @After
    public void tearDown(TestContext context) {
        vertx.close(context.asyncAssertSuccess());
    }

    @Test
    public void testMyApplication(TestContext context) {
        final Async async = context.async();

        vertx.createHttpClient().getNow(8080, "localhost", "/",
            response ->
                response.handler(body -> {
                    context.assertTrue(body.toString().contains("Hello"));
                    async.complete();
                }));
    }
}
```

这是我们垂直领域的 JUnit 测试案例。测试使用了`vertx-unit`，所以我们配置了一个定制的运行器(带有`@RunWith`注释)。`vertx-unit`使测试异步交互变得容易，这是 Vert.x 应用程序的基础。

在`setUp`方法(在每次测试之前调用)中，我们创建了一个`vertx`的实例，并部署了我们的 verticle。您可能已经注意到，与传统的 JUnit `@Before`方法不同，它接收一个`TestContext`对象。这个对象让我们控制测试的异步方面。例如，当我们部署 verticle 时，它是异步启动的，大多数 Vert.x 交互也是如此。在它正确启动之前，我们无法检查任何东西。因此，作为`deployVerticle`方法的第二个参数，我们传递一个结果处理程序:`context.asyncAssertSuccess()`。如果垂直线没有正确开始，则测试失败。此外，它会一直等待，直到 verticle 完成它的开始序列。记住，在我们的 verticle 中，我们称之为`fut.complete()`。所以它一直等到这个方法被调用，在失败的情况下，测试失败。

嗯，`tearDown`(每次测试后调用)方法很简单，只需关闭我们创建的`vertx`实例。

现在让我们来看看我们的应用程序的测试:`testMyApplication`方法。该测试向我们的应用程序发出一个请求，并检查结果。发出请求和接收响应是异步的。所以我们需要一种方法来控制它。像`setUp`和`tearDown`方法一样，测试方法接收一个`TestContext`。从这个对象中，我们创建了一个异步句柄(`async`)，让我们在测试完成时通知测试框架(使用`async.complete()`)。

因此，一旦创建了异步句柄，我们就创建一个 HTTP 客户端，并发出一个由我们的应用程序使用`getNow()`方法处理的 HTTP 请求(`getNow`只是`get(...).end()`的一个快捷方式)。响应由处理程序处理。在这个函数中，我们通过向 handler 方法传递另一个函数来检索响应体。`body`参数是响应体(作为一个`buffer`对象)。我们检查主体是否包含`"Hello"`，并宣布测试完成。

让我们花点时间来提一下这些断言。与传统的 JUnit 测试不同，它使用`context.assert`....事实上，如果断言失败，它会立即中断测试。所以使用这些断言很重要，因为 Vert.x 应用程序和测试是异步的。然而，Vert.x Unit 提供了钩子让你使用 Hamcrest 或 AssertJ，如这个[例子](https://github.com/vert-x3/vertx-examples/blob/master/unit-examples/src/test/java/io/vertx/example/unit/test/JUnitAndHamcrestTest.java)和另一个[例子](https://github.com/vert-x3/vertx-examples/blob/master/unit-examples/src/test/java/io/vertx/example/unit/test/JUnitAndAssertJTest.java)所示。

我们的测试可以从 IDE 中运行，或者使用 Maven:

```
mvn clean test
```

## 包装

所以，我们总结一下。我们有一个申请和一个测试。现在让我们打包应用程序。在本文中，我们将应用程序打包在一个*大罐子*中。fat jar 是一个独立的可执行 jar 文件，包含运行应用程序所需的所有依赖项。这是打包 Vert.x 应用程序的一种非常方便的方式，因为它只有一个文件。这也使得它们易于执行。

要创建一个 fat jar，只需运行:

```
mvn clean package
```

`vertx-maven-plugin`负责打包并创建`target/my-first-app-1.0-SNAPSHOT.jar`文件，嵌入我们的应用程序以及所有的依赖项(包括 Vert.x 本身)。检查大小:6MB 左右；这包括运行应用程序的一切。

## 执行我们的应用程序

嗯，有一个 jar 很好，但是我们希望看到我们的应用程序运行！如上所述，由于 fat jar 打包，运行 Vert.x 应用程序很容易:

```
java -jar target/my-first-app-1.0-SNAPSHOT.jar
```

然后，打开浏览器到 [http://localhost:8080](http://localhost:8080) 。

要停止应用程序，请点击`CTRL+C`。

## 结论

这个 Vert.x *速成班*展示了:如何使用 Vert.x 开发一个简单的应用程序；如何测试、打包和运行它。所以你现在在 Vert.x 的基础上构建一个惊人的系统的道路上有了一个很好的开始。下次我们将看到如何[配置我们的应用](https://developers.redhat.com/blog/2018/03/22/eclipse-vert-x-application-configuration/)。不要忘记这个博客文章系列的代码可以在[这个资源库](https://github.com/redhat-developer/introduction-to-eclipse-vertx)中找到。

快乐编码&敬请关注！

*Last updated: April 22, 2022*