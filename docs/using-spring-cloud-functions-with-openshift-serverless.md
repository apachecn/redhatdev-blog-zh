# 通过 OpenShift 无服务器使用 Spring 云函数

> 原文：<https://developers.redhat.com/blog/2020/09/01/using-spring-cloud-functions-with-openshift-serverless>

当构建[无服务器应用](https://developers.redhat.com/topics/serverless-architecture)时， [Java 开发者](https://developers.redhat.com/topics/enterprise-java)又有了一个有趣的选择。您已经看到了如何使用 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 为 Red Hat OpenShift Serverless 构建和运行应用程序，但是在本文中，我们将讨论如何使用 Spring Cloud 函数并带您完成这些步骤。这些步骤类似于使用 [OpenShift 无服务器运行任何其他](https://openshift.com/serverless) [Spring Boot](https://developers.redhat.com/topics/spring-boot) 应用程序。构建一个开放的混合无服务器平台的好处之一是让开发者可以选择[编程语言](https://developers.redhat.com/blog/category/languages-compilers/)、[工具](https://developers.redhat.com/topics/developer-tools)、框架，以及跨任何环境运行无服务器应用的可移植性。除此之外，您希望确保开发人员体验和整体工作流是直观和实用的，这是您将在这里学到的。

如果您对观看本文中执行的步骤感兴趣，您可以在 YouTube 上观看视频。

https://www.youtube.com/watch?v=yXlTs0On3Ys

## 要求

*   【T0 度】T1
*   Java 开发工具包(JDK 8 版以上)
*   OpenShift 4.3+版
*   [OpenShift 无服务器](https://openshift.com/serverless) 1.7 以上
*   `curl`
*   [`kn`](https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-kn.html) (Knative Client)

## 生成 Spring 云函数项目

生成 Spring 项目的最简单的方法之一是使用`curl`来访问`start.spring.io`，这正是我们将如何开始我们的项目:

https://gist.github.com/markito/ab6184e4f18bca33f89df78721731411

这将在`my-function-project`文件夹中生成并下载一个项目。

## 实现你的第一个功能

我们将实现一个快速和肮脏的谷歌翻译包装。为了使用 Spring Cloud 函数创建函数，您需要一个带有`@Bean`注释的方法，该方法可以遵循来自`java.util.function`的任何函数接口，例如`Consumer`、`Supplier`或`Function`。关于 Spring Cloud 函数如何工作的更多细节，请阅读[文档](https://docs.spring.io/spring-cloud-function/docs/3.0.8.RELEASE/reference/html/spring-cloud-function.html)，但是对于这个例子，只需将下面的方法复制并粘贴到你的*DemoApplication.java*文件中:

https://gist.github.com/markito/e976443fd61a6d7d2278f2acfa7c18d3

### 在本地测试您的功能

由于 Spring Cloud 函数只是 Spring Boot 应用程序，所以您可以像任何其他 Java 应用程序一样使用 JUnit、Mockito 或任何您喜欢的东西来实现单元测试。您还可以使用`gradle bootRun`或`mvn spring-boot:run`在本地运行应用程序，这对于在容器中运行[之前迭代验证应用程序很有用。确保您已经修复了所有 Java 导入，然后使用以下命令在本地启动应用程序:](https://developers.redhat.com/topics/containers)

https://gist.github.com/markito/cf771a9941e7b8e3e187aaaa1eee55e7

每个函数都将映射到一个端点，可以按如下方式访问:

```
http://localhost:8080/<functionName>

```

具体来说，对于我们的 translate 函数，您可以使用以下代码访问端点:

https://gist.github.com/markito/5340d90f3288507d7c522733e0dd6338

我们发布一个单词或短语，它就会被翻译成西班牙语。您可以解析输出并正确格式化它，但是为了保持简短，我将把它留给读者作为练习。愿[杰克森](https://github.com/FasterXML/jackson)和[杰克森](https://www.json.org/json-en.html)成为你的朋友。

## 使用 Jib 构建容器

到目前为止，我们已经构建了一个 Spring 应用程序，并在本地执行它，但是是时候将应用程序容器化了。有许多方法可以执行这一步，但是我决定坚持使用 Java 社区使用的众所周知的工具，所以我将使用`[Jib](https://github.com/GoogleContainerTools/jib)`并将其作为插件添加到我的 Gradle 项目中。

编辑`build.gradle`文件并将这一行`id 'com.google.cloud.tools.jib' version "2.4.0"`添加到您的插件部分。它应该是这样的:

https://gist.github.com/markito/f6ee63153a25351d7fcccbb37885598b

将 Jib 作为项目的一部分，Gradle 可以构建您的容器并将其推送到您选择的容器注册中心。码头或[码头](https://hub.docker.com/)是众所周知的选择。

在 Gradle 项目上安装了 jib 插件后，使用下面的命令为项目构建一个容器:`gradle build jib --image=<your_container_registry>/demo-app:v1`

在我的机器上看起来是这样的:

https://gist.github.com/markito/9bf7aca34c8c1aabdd17bce7f8abde50

这为您的项目生成了一个容器，并自动将其推送到容器注册表中。如果你是第一次接触 [Linux](https://developers.redhat.com/topics/linux) 容器和容器注册表，请阅读[这篇文章](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/)以获得关于这个主题的实用介绍。

## 部署到 OpenShift 无服务器

随着容器的构建，您现在可以使用 Knative CLI 部署应用程序了，`kn`:

https://gist.github.com/markito/7f40db940f0edd2bd8e3898d71543f0a

OpenShift Serverless 将负责创建一个 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 部署、一个路由、SSL 配置(使用您的集群的配置)以及基于请求数量的自动伸缩配置。关于 OpenShift 无服务器的更多细节，请查看[产品文档](https://docs.openshift.com/container-platform/4.4/serverless/serverless-getting-started.html)。

上述代码将在六秒钟后自动缩小应用程序(到零)，而无需新的请求。

使用 OpenShift 中运行的服务的 URL 重试:

https://gist.github.com/markito/b1673b96043b3670c24f9a23a3666925

## 添加一个可以处理 JSON 文档的新函数

在创建 REST APIs 时，发送和接收 JSON 文档是非常常见的，您可以非常容易地添加一个接受 JSON 消息而不是字符串的函数。

### 数据模型

用以下内容创建一个新的`UserReview.java`文件:

https://gist.github.com/markito/df869ca8245f59ffde0175b0f679c476

我们将使用这个类来编组和解组由 API 发送或接收的 JSON 对象。用下面的内容创建一个`review.json`文件。这将是我们新的`translateReview`函数的输入。

https://gist.github.com/markito/31a7e912be19d7babf9b1aa719632b94

### 功能代码

这将非常类似于我们之前创建的函数，它只是另一个带有`@Bean`注释并使用我们的 POJO 类作为输入和输出的方法。

https://gist.github.com/markito/6cf0c421f3936ffb741ca61948666a98

### 发布 JSON 文件

在这里，我们将继续使用`curl`作为我们的客户端，并指定一个路径到上一步中创建的`review.json`文件。使用`gradle bootRun`再次启动应用程序，然后向 API 提交一个 json 文件:

https://gist.github.com/markito/f35a880180b46258b5050db381b20c6f

如您所见，该函数现在接受一个 JSON 文档，并将用户注释翻译成西班牙语。如果你喜欢的话，你可以尝试用一种声明性函数组合的方式来实现它，但是我现在保持简单。

### 构建您的新容器

步骤与之前相同，但是使用了一个`v2`标签，构建了一个新版本的容器:

```
$ gradle jib --image=<your_container_registry>/demo-app:v2
```

### 更新已部署的应用程序以包含新功能

现在，我们将使用 OpenShift Serverless 的另一个有趣功能，我们将部署一个新版本的应用程序，但使用不同的 URL，这样，该 API 的当前客户端甚至不会知道这个新功能，直到我们决定向它发送流量:

https://gist.github.com/markito/5e71653d3bda5f13b36151526a392cdc

这可以通过使用标签来实现。在这个例子中，我用一个将被附加到服务 URL 的值`preview`来标记一个特定的修订版本`@latest`。还要注意，我将流量 100%设置到先前的版本`translator-v1`，这意味着没有流量将被发送到正在部署的新版本。这也称为“黑暗启动”，在这种情况下，我的应用程序的新版本在生产环境中可用，但不一定会收到任何请求，除非有人知道使用哪个 URL。

验证完成后，您可以决定逐步使用[淡黄色或蓝/绿色部署](https://opensource.com/article/17/5/colorful-deployments)型号发送流量。在[https://learn . open shift . com/developing-on-open shift/server less/](https://learn.openshift.com/developing-on-openshift/serverless/)上有一个关于如何实现这些模型的分步实验

### 测试

现在您可以像以前一样执行相同的 curl 命令，但是这次添加了前缀`preview`:

https://gist.github.com/markito/efec228f47b7515d03d5475d8deb6851

## 结论

您可以使用 OpenShift Serverless 轻松构建和部署 Spring Cloud Functions 应用程序。对于 Java 开发人员来说，工作流感觉很自然，您甚至可以使用 Gradle(或 Maven)插件(如 Jib)来构建容器。

![A Spring Cloud Function in Red Hat OpenShift](img/598179e782a08e0a412879cba66defa1.png)

使用 OpenShift Serverless，您还可以轻松地部署应用程序的多个版本，并执行暗启动、蓝/绿或淡黄色部署。OpenShift 开发人员控制台使得可视化交通路线信息和应用程序的整体拓扑更加容易。

有关更多详情，[请查看 OpenShift 无服务器页面](https://openshift.com/serverless)。