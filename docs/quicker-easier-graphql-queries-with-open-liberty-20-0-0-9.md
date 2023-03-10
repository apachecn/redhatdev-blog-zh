# 开放自由 20.0.0.9，更快、更简单的 GraphQL 查询

> 原文：<https://developers.redhat.com/blog/2020/09/29/quicker-easier-graphql-queries-with-open-liberty-20-0-0-9>

open Liberty 20.0.0.9 允许开发人员试验类型安全的 SmallRye GraphQL 客户端 API，并使用内置的 GraphQL 用户界面(UI)更轻松地编写和运行 GraphQL 查询和变体。本文介绍了 Open Liberty 20.0.0.9 中的新特性和更新:

*   [尝试第三方 GraphQL 客户端 API。](#GraphQLAPIs)
*   [使用内置的 GraphiQL UI 进行更快的查询和转换。](#GraphiQL)
*   请给我们您的反馈！

## 使用 Open Liberty 20.0.0.9 运行您的应用

如果您使用的是 [Maven](https://openliberty.io//guides/maven-intro.html) ，请使用这些坐标更新到 Open Liberty 的最新版本:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.9</version>
    <type>zip</type>
</dependency>

```

For [Gradle](https://openliberty.io//guides/gradle-intro.html) , enter:

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.9,)'
}

```

如果您使用 Docker，它是:

```
FROM open-liberty

```

## 试用第三方 GraphQL 客户端 API

MicroProfile GraphQL 在 Open Liberty 上才推出几个月，就已经红极一时了。也就是说，我们有一些方法可以改进它，使它更完整。我们想要改进的一个特性是 GraphQL 客户端 API。虽然正式的客户端 API 要到 MicroProfile GraphQL 的下一个版本才会发布，但是您现在可以开始尝试使用类型安全的 SmallRye GraphQL 客户端 API。

SmallRye 是 MicroProfile GraphQL 的底层实现。通过为您的应用程序添加“第三方”API 类型可见性，您可以访问其超乎规范的特性:

```
<application name="MyGraphQLApp" location="MyGraphQLApp.war">
    <classloader apiTypeVisibility="+third-party"/>
</application>

```

此更新允许您像类型安全客户端一样访问 SmallRye GraphQL APIs。请注意，这些 API 在未来的版本中可能会发生变化，因为 SmallRye 在不断发展。欲了解更多信息，请访问 [SmallRye GraphQL 项目](https://github.com/smallrye/smallrye-graphql)。对于开放自由 20.0.0.9，我们使用的是 SmallRye GraphQL 1.0.7。

### 远程方法的类型安全调用

SmallRye GraphQL 客户端 API 与 [MicroProfile Rest 客户端](https://developers.redhat.com/cheat-sheets/microprofile-rest-client)非常相似，后者使用一个接口以类型安全的方式调用远程方法。例如，假设我们想要一个可以调用给定位置的所有超级英雄的查询的客户端。我们将创建这样一个查询界面:

```
@GraphQlClientApi
interface SuperHeroesApi {
    List allHeroesIn(String location);
}

```

其中`SuperHero`在客户端看起来是这样的:

```
class SuperHero {
    private String name;
    private List superPowers;
}

```

在服务器端，`SuperHero`实体可能包含几十个字段，但是如果我们只对英雄的名字和超能力感兴趣，那么我们在客户端类中只需要这两个字段。现在，我们可以用如下代码调用查询:

```
SuperHeroesApi api = GraphQlClientBuilder.newBuilder().build(SuperHeroesApi.class);
List heroesOfNewYork = api.allHeroesIn("NYC");

```

记住这个客户端 API 不是官方的，但是官方的 MicroProfile GraphQL 1.1 API 将基于它。就当这是预演。

## 使用内置的 GraphiQL UI 进行更快的查询和转换

Open Liberty now 有一个内置的 [GraphiQL](https://github.com/graphql/graphiql/blob/main/packages/graphiql/README.md) 用户界面，如图 1 所示。新的基于 web 的 UI 允许您实时编写和执行 GraphQL 查询和变化，并具有高级编辑功能，如命令完成、查询历史、模式自省等。

[![Web UI open to edit a query, with hover-over text displayed.](img/f7facc690f87ad327e84d2e646be66d4.png "Screenshot 2020-09-22 at 10.50.00")](/sites/default/files/blog/2020/09/Screenshot-2020-09-22-at-10.50.00.png)

Figure 1: Work faster with the GraphiQL web UI.

要启用 UI，您必须首先[编写并部署一个微文件 GraphQL 应用程序](https://openliberty.io/blog/2020/06/10/microprofile-graphql-open-liberty.html)。然后把这一行加到你的`server.xml`:

```
<variable name="io.openliberty.enableGraphQLUI" value="true" />

```

您可以使用 web 浏览器访问 UI，只需打开 GraphQL 应用程序的上下文根并添加`/graphql-ui`。例如，假设我们使用默认端口(9080)，我们的应用程序被命名为`myGraphQLApp`。在这种情况下，我们将在`http://localhost:9080/myGraphQLApp/graphql-ui`访问用户界面。

此变通办法已在问题#13201 中[解决。](https://github.com/OpenLiberty/open-liberty/issues/13201)

## 我们需要您的反馈

作为一个开源团队，我们喜欢收到来自 Open Liberty 用户的反馈。最近的一个例子是这个评论，摘自 Open Liberty [#13036](https://github.com/OpenLiberty/open-liberty/issues/13036) ):“你好，我正在 Open Liberty 上使用 microprofile-graphql，除了通过 microprofile config 的异常白名单机制之外，一切都很好……”

我们的 MicroProfile GraphQL 特性才推出几个月，所以知道用户正在采用它真是太好了。我们也很高兴你们中的一些人已经在探索异常处理和类似特性的“黑暗角落”。

虽然我们不喜欢发现我们让一个错误从裂缝中溜走，但当它们出现时，我们渴望修复它们。如果您发现了一个问题，或者想提出一个改进建议，让您对 Open Liberty 的体验更好，请告诉我们。你可以通过[在 GitHub](https://github.com/OpenLiberty/open-liberty/issues) 上发布一个问题或者在 Twitter 上联系我们 [@OpenLibertyIO](https://twitter.com/OpenLibertyIO) 。我们也可以使用 [Gitter](https://gitter.im/OpenLiberty/help) 和 [Open Liberty 开发者体验](https://gitter.im/OpenLiberty/developer-experience)页面进行在线聊天。

## 现在尝试在红帽运行时打开自由 20.0.0.8

Open Liberty 是 Red Hat Runtimes 产品的一部分，可供 [Red Hat Runtimes 用户](https://access.redhat.com/products/red-hat-runtimes)使用。要了解更多关于将 Open Liberty 应用部署到 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 的信息，请参阅我们的 *[Open Liberty 指南:将微服务部署到 OpenShift](https://openliberty.io/guides/cloud-openshift.html)* 。

*Last updated: September 25, 2020*