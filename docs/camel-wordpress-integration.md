# 使用 Apache Camel 自动生成新闻并发布到 WordPress

> 原文：<https://developers.redhat.com/blog/2018/08/27/camel-wordpress-integration>

随着 [Apache Camel 2.21](http://camel.apache.org/camel-2210-release.html) 的发布，一些新的组件被添加到项目中， [Camel WordPress](https://github.com/apache/camel/blob/master/components/camel-wordpress/src/main/docs/wordpress-component.adoc) 就是其中之一。骆驼是[红帽导火索](https://developers.redhat.com/products/fuse/overview/)的上游社区项目之一。在本文中，我们将看到如何使用这个新组件来发布基于[足球统计 API](https://www.football-data.org/) 的自动生成的新闻帖子。这个例子使用了 statistics API，基于一个[自然语言生成(NLG)库](https://github.com/simplenlg/simplenlg)生成文本，然后将其发布到 WordPress 博客。

WordPress 是创建网站最常用的开源工具之一。超过 30%的网站是建立在 WordPress 之上的。除了创建网站、博客和应用程序，WordPress 还利用了一个由热情的社区维护的巨大插件库。甚至有插件可以把 WordPress 网站变成一个[电子商务平台](https://woocommerce.com/)。

从版本 4.7 开始，WordPress 公开了一个 [REST API](https://developer.wordpress.org/rest-api/) ，能够与它的资源交互，例如用户、类别、页面、文章和自定义类型。现在，第三方有可能与 WordPress 平台集成，并利用他们的资源执行几乎任何事情。

一些公司使用 WordPress 实现内部网站、博客和项目网站。将这样的平台与另一家公司的组件整合——比如 CRM、ERP、LDAP 和日历服务——将为基于 WordPress 的项目增加额外的价值。Camel WordPress 可以帮助轻松集成这些组件。要开始使用这个新组件，没有什么比演示更好的了。

## 演示

对于这个演示，我们将实现一个场景，其中 Apache Camel 将基于足球统计 API 生成描述足球比赛结果的新闻。然后这条新闻将会被发布在我们的足球新闻博客上。

这个演示的灵感来自于[新闻写作机器人](https://www.wired.com/2017/02/robots-wrote-this-story/)，在未来，它们可以根据一套规则、NLG 和人工智能轻松地写几段文字。当然，对于这个简单的演示，我们不会创建任何过于花哨的内容。我们有一个预格式化的模板集，其中包含一场足球比赛的可能结果，准备从 API 接收比赛安排。这些模板的灵感来自若昂·艾利斯的[论文，名为“体育新闻的自动生成”，由波尔图大学(葡萄牙)出版](https://sigarra.up.pt/feup/pt/pub_geral.pub_view?pi_pub_base_id=139549)。

请看下图，了解演示架构和路由流程。

有两个 REST 端点公开了这个演示的用例。第一个是“summary”，它只是将足球比赛的细节转换成一条新闻。第二个是“send”，将生成的新闻发布到我们的博客。花点时间搞清楚流程。

[![](img/4badc9dfea4ca66b548f245851f00fc0.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/08/flow2.png)

敏锐的读者会注意到，我们在两个不同的用例中重用了一些路由。Camel 是一个强大的框架，由于它的输入/输出过程设计，使得代码重用更加容易。在开始为你的路线编码之前，花点时间计划你要做什么，并通过将路线分成更小的部分来简化路线。这样，您可以获得更好的架构和干净的代码。

REST 端点非常简单，因为我们实现了 Camel REST 功能来公开我们的路线。如果你不熟悉这个特性，可以看一下 [Camel 文档](http://camel.apache.org/rest-dsl.html)。

## 路线

让我们深入了解 Get Fixture 细节并转换为新闻路线。

Get Fixture Details 路由调用了[足球数据 API](https://www.football-data.org/) 的 REST 端点。API 所有者非常友好地免费提供了他的 API 的基本用法。[我们在这次演示中使用了免费层](https://www.football-data.org/client/register)。如果你要运行这个演示，我建议你也这样做，因为你需要一个 API 密匙。

该路由将匹配的 fixture ID 作为参数，调用第三方 API，并将其 JSON 结果转换为我们的内部域模型(`Statistics`):

```
from("direct:get-fixture-detail")
  .routeId("get-fixture-details")
  .setHeader("X-Auth-Token", 
    constant(config.getFootballApiToken()))
  .toF("rest:get:%s?host=%s&synchronous=true", 
    config.getFootballApiFixturePath(), 
    config.getFootballApiHost())
  .unmarshal().json(JsonLibrary.Jackson, Statistics.class);
```

有了 API 的结果，就该把这些数据转换成文本了。对于这个任务，我们使用简单的 NLG 库。这个库促进了英语习语中自然语言[的生成。举个例子，让我们创造一个简单的句子，比如*玛丽追猴子*:](https://en.wikipedia.org/wiki/Natural_language)

```
SPhraseSpec p = nlgFactory.createClause();
p.setSubject("Mary");
p.setVerb("chase");
p.setObject("the monkey");
String output2 = realiser.realiseSentence(p);
System.out.println(output2);
```

请注意，我们只将不定式形式的动词传递给库，而将语法规则留给引擎。我们甚至可以用一个简单的论点把这句话变成过去式:

```
p.setFeature(Feature.TENSE, Tense.PAST);
```

*“玛丽追猴子”*将是输出。

这只是冰山一角；你可以利用 NLG 图书馆做更多的事情。深入研究 NLG 超出了本文的范围，但是如果你有兴趣[在 Github 中有一个很好的教程解释如何使用它](https://github.com/simplenlg/simplenlg/wiki/Section-0-%E2%80%93-SimpleNLG-Tutorial)。

负责将比赛数据转换成文本的路由称为 Convert to News，这是一个简单的`bean`调用:

```
from("direct:convert-nlg")
  .routeId("convert-nlg")
  .bean(ContentFactory.class, "generate");
```

域数据到文本的转换发生在类`ContentFactory`内部。这个工厂负责协调`nlg`包内的文本生成。下图用类图说明了这个包内部的关系。在现实场景中，这种转换可能是另一个微服务的责任；这样，Camel 将只处理集成部分。

[![](img/d7e3e707173b5a4714dc1bf493ed897e.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/08/nlg-package.png)

`ContentFactory`将摘要生成委托给`IntroTemplate`类，它将创建我们新闻的第一段。出于演示的目的，这是一个只有一句话的段落。通过在这个包中创建更多的模板类，您可以很容易地添加更多的内容，例如，一个“最佳球员比赛”段落描述了一个球员在比赛中的表现。

如果我们假设德国队以 7 比 1 的比分击败巴西队，这个引擎将生成如下结果:

7 月 8 日，德国队造访巴西，以 1 比 7 的比分带回家一场大胜。

生成匹配摘要后，路由以名为`StatisticsSummary`的域模型的形式输出它。这个域模型包含关于比赛摘要、日期和球队名称的数据。这个结构将是最后一个路由的输入，它将把这个统计域转换成一个 WordPress 帖子，并将其发布到我们的博客上。

最后，我们到达发送到 Wordpress 的路径。该路径将帖子的发布委托给 Camel WordPress 组件，使用 [Camel type converter](http://camel.apache.org/type-converter.html) 特性将比赛摘要转换成 WordPress 帖子:

```
from("direct:post-news-summary")
.routeId("post-news-summary")
.convertBodyTo(Post.class)
.to("wordpress:post");
```

要使用 Camel type converter 特性，只需将文件 [TypeConverter](https://github.com/ricardozanini/camel-example-wordpress/blob/master/src/main/resources/META-INF/services/org/apache/camel/TypeConverter) 添加到路径`services/org/apache/camel`中，并使用转换器类的完全限定名。让我们来看看这个类:

```
@Converter
public final class StatisticsToPostConverter {

  private static final int DEFAULT_AUTHOR_ID = 1;

  private static final Logger LOGGER = 
    LoggerFactory.getLogger(StatisticsToPostConverter.class);

  @Converter
  public static Post toPost(StatisticsSummary statisticsSummary, 
                            Exchange exchange) {
    final Post post = new Post();
    final Content postContent = 
      new Content(statisticsSummary.getSummary());
    postContent.setRaw(statisticsSummary.getSummary());
    final Content titleContent = 
      new Content(String.format("%s X %s Results", 
             statisticsSummary.getFixture().getHomeTeamName(), 
             statisticsSummary.getFixture().getAwayTeamName()));
    titleContent.setRaw(titleContent.getRendered());
    post.setContent(postContent);
    post.setFormat(Format.standard);
    post.setStatus(PublishableStatus.publish);
    post.setTitle(titleContent);
    post.setAuthor(DEFAULT_AUTHOR_ID);
    LOGGER.debug("Converted StatisticsSummary {} to Post {}", 
      statisticsSummary, post);
    return post;
  }
}
```

这个转换器的职责是创建一个新的`Post`对象，并根据 WordPress 规则为一篇新发布的文章填充它。

前几行只是设置了帖子的原始内容和格式。帖子的发布状态也可以设置为**草稿**或**私人**。草稿状态会将文章保存在 WordPress 数据库中，以备后用。在私人状态下，只有管理员才能看到帖子。检查 WordPress 文档中的其他状态。

创建新帖子时，作者 ID 是另一个重要的属性。如果你不确定，可以通过 Camel WordPress 插件的作者操作或者查看 WordPress 用户的仪表板来获取。

另一个需要检查的重要事情是 WordPress 的配置和连接。为了使 Camel WordPress 插件工作，必须安装插件[基本认证](https://github.com/WP-API/Basic-Auth)。这样，Camel 可以使用 HTTP 基本认证连接到 WordPress 并执行写操作。

未来版本的 Camel WordPress [将使用基于令牌的认证](https://issues.apache.org/jira/browse/CAMEL-12201)，这是一种在非 TLS 连接中比 HTTP Basic 更安全可靠的方法。值得注意的是，永远不要使用普通的 HTTP 连接通过 HTTP Basic 进行身份验证，因为凭据是以 base64 编码的字符串形式传递的，攻击者可以很容易地窃取这些凭据。

## Camel WordPress 配置

要让 Camel WordPress 工作，需要一些配置。这种配置可以在[路线定义](https://github.com/ricardozanini/camel-example-wordpress/blob/master/src/main/java/sample/camel/wordpress/ExampleCamelWordpressRoute.java)中完成。在那里，我们传递能够创建新帖子的 WordPress URL 和用户凭证:

```
final WordpressComponentConfiguration configuration = 
  new WordpressComponentConfiguration();
final WordpressComponent component = new WordpressComponent();
configuration.setUrl(config.getWordpressUrl());
configuration.setPassword(config.getWordpressPassword());
configuration.setUser(config.getWordpressUser());
component.setConfiguration(configuration);
getContext().addComponent("wordpress", component);
```

所有这些配置都使用后来注入到 Camel 上下文中的 Spring Boot 属性进行了具体化:

```
camel.springboot.name=CamelSampleWordpress
camel.component.servlet.mapping.context-path=/api/*

logging.level.org.apache.http=DEBUG
logging.level.org.apache.http.wire=ERROR
logging.level.org.restlet=DEBUG
logging.level.org.apache.camel=DEBUG

football.api.fixture.path=v1/fixtures/{fixtureId}
football.api.host=https://api.football-data.org
football.api.token=${FOOTBALL_API_TOKEN:<your_api_token>}

wordpress.url=${WORDPRESS_URL:http://localhost/wp-json/}
wordpress.user=${WORDPRESS_USER:user}
wordpress.password=${WORDPRESS_PASS:pass}
```

这些属性是在`application.properties`文件中设置的，该文件稍后将在类`ExampleCamelWordpressRouteConfig`中可用，以便跨路线使用。

## 运行演示

为了让演示运行起来[,我们有一个 Red Hat OpenShift 模板](https://github.com/ricardozanini/camel-example-wordpress/blob/master/openshift/camel-wordpress-sample-template.yaml),它创建了一个由 MySQL 数据库和这个 Camel 演示微服务支持的 WordPress 博客。访问“摘要”端点，查看 NLG 引擎的工作情况:

```
curl http://<camel-wp-example-host>/api/match/158186/summary
{"fixture":{"date":"2017-05-14T19:00:00.000+0000",
"status":"FINISHED" (...)
```

要将这个摘要发送到 WordPress 博客，向“发送”端点发出一个简单的 GET 请求:

```
curl http://<camel-wp-example-host>/api/match/158186/send
{"id":6,"author":1,"date":"2018-08-20T18:05:37.000+0000",
"modified":"2018-08-20T18:05:37.000+0000",
"slug":"avai-sc-x-ec-vitoria-results-2" (...)
```

为了简单起见，在这个演示中，我们使用了 GET 动词，但是在实际场景中，最好使用 POST 动词来创建新内容。

最后，我们应该看到我们的新闻发布到 WordPress 首页:

![](img/edbc389c89b5306612d375724a77d889.png)

## 摘要

在本文中，我们看到了如何使用 Apache Camel 自动生成 NLG 的新闻帖子，并通过检索统计数据的 soccer 公共 API 将它们发布到 WordPress 博客上。Camel WordPress 组件的行为类似于脸书、Twitter 和其他与社交媒体集成以生成新内容的组件。除了创建帖子之外，该组件还有作者操作，可以用来跨数据库集成用户数据。

我希望这个例子可以帮助你使用 WordPress 编写新的集成和用例。[完整源代码托管在 GitHub](https://github.com/ricardozanini/camel-example-wordpress) (欢迎投稿！).请继续关注，因为我们计划在 Camel WordPress 组件中添加更多的操作。

*Last updated: February 23, 2022*