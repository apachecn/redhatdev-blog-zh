# 开放自由 20.0.0.6 对 GraphQL 的支持

> 原文：<https://developers.redhat.com/blog/2020/06/17/support-for-graphql-with-open-liberty-20-0-0-6>

开放自由 20.0.0.6 版本带来了新的特性、更新和错误修复。本文介绍了 Open Liberty 20.0.0.6 中的新特性，包括支持开发“代码优先”的 GraphQL 应用程序，从 Maven 存储库中提供特性，以及使用服务器配置来控制应用程序的启动。

## 开放自由 20.0.0.6 有什么新内容

open Liberty 20.0.0.6 包括以下功能更新，我将在接下来的部分中讨论:

*   [使用开放自由的 GraphQL】](#GQL)
*   [从 Maven 知识库中提供特性](#MVN)
*   [管理员控制应用启动顺序](#ORDER)
*   [Open Liberty Grafana 仪表盘中新的 REST 可视化功能](#GRA)
*   [网络应用启动变化](#STA)

**注**:访问 [Open Liberty 的 GitHub 库](https://github.com/OpenLiberty/open-liberty/issues?q=label%3Arelease%3A20006+label%3A%22release+bug%22+)了解 Open Liberty 20.0.0.6 的漏洞修复。

## 使用 Open Liberty 20.0.0.6 运行您的应用

使用以下命令获取或更新以使用 [Maven](https://openliberty.io/guides/maven-intro.html) 打开自由 20.0.0.6:

```
  <dependency>
      <groupId>io.openliberty</groupId>
      <artifactId>openliberty-runtime</artifactId>
      <version>20.0.0.6</version>
      <type>zip</type>
  </dependency>

```

如果您使用的是 [Gradle](https://openliberty.io/guides/gradle-intro.html) ，只需输入:

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.6,)'
}

```

以下是 Docker 的命令:

```
FROM open-liberty

```

参见[打开 Liberty 下载页面](https://openliberty.io/downloads/)获取 Jakarta EE 8、Web Profile 8 和 MicroProfile 3 的最新版本。

## 通过 Open Liberty 使用 GraphQL

Open Liberty 对 [MicroProfile GraphQL](https://github.com/eclipse/microprofile-graphql) 的支持让开发人员编写“代码优先”的 GraphQL 应用程序，让客户控制他们接收的数据。您可以使用像`@Query`和`@Mutation`这样的注释将普通的旧 Java 对象(POJOs)转换成基于 HTTP 的 GraphQL 端点。当查询或变异方法返回一个现有的实体对象时，客户端可以指定它感兴趣的字段——这减少了网络带宽和客户端处理。

这里有一个例子:

```
@GraphQLApi
public class MovieService {
    AtomicInteger nextId = new AtomicInteger();
    Map<Integer, Movie> movieDB = new HashMap();

    @Query("movieById")
    public Movie getMovieByID(int id) throws UnknownMovieException {
        return Optional.ofNullable(movieDB.get(id)).orElseThrow(UnknownMovieException::new);
    }

    @Query("allMoviesDirectedBy")
    public List<Movie> getAllMoviesWithDirector(String directorName) {
        return movieDB.values().stream()
                               .filter(m -> m.getDirector().equals(directorName))
                               .collect(Collectors.toList());
    }

    @Mutation("newMovie")
    public int createNewMovie(@Name("movie") Movie movie) {
        int id = nextId.incrementAndGet();
        movie.setId(id);
        movieDB.put(id, movie);
        return id;
    }
}

```

这段代码用两个查询(`movieById`和`allMoviesDirectedBy`)和一个变异(`newMovie`)创建了一个 GraphQL 应用程序。如果客户端执行以下查询:

```
query {
  allMoviesDirectedBy(directorName: "Roland Emmerich") {
    id, title, actors
  }
}

```

结果将是:

```
{
  "data": {
    "allMoviesDirectedBy": [
      {
        "id": 1,
        "title": "Independence Day",
        "actors": [
          "Will Smith",
          "Bill Pullman",
          "Jeff Goldblum",
          ...
        ]
      },
      ...
    ]
  }
}

```

Open Liberty 的 GraphQL APIs 是在 MicroProfile 社区中开发的，拥有广泛的行业支持。该实现基于 SmallRye GraphQL。Liberty 的 GraphQL 特性超越了 MicroProfile 规范，增加了对收集指标、检查授权、记录查询和变异方法的请求和响应的支持。

有关更多细节和示例应用程序，请参见 Open Liberty 中的这个[micro profile graph QL 功能的简单演示。加入 GraphQL 采用者不断增长的](https://github.com/OpenLiberty/sample-mp-graphql)[版图](https://landscape.graphql.org/)，今天就编写你的第一个 GraphQL 应用程序吧！

## 从 Maven 存储库中提供特性

开发人员现在可以使用方便的命令行工具将 Maven 中央存储库或内部 Maven 存储库(如 Artifactory 或 Nexus 上的存储库)中的功能直接安装到 Open Liberty 运行时上。表 1 详细说明了如何使用`wlp/bin/featureUtility`命令来查找、安装和获取 Maven 存储库中的资产信息。

#### 表 1。打开 Liberty 命令从 Maven 存储库中安装特性

| **命令** | **描述** |
| `featureUtility help installFeature` | 显示`installFeature`动作的帮助信息。 |
| `featureUtility installFeature mpHealth-2.2`或`featureUtility installFeature io.openliberty.features:mpHealth-2.2` | 从 Maven Central 安装 MicroProfile Health 2.2 特性。 |
| `featureUtility installServerFeatures myserver` | 为`myserver`服务器安装服务器功能。 |
| `featureUtility installFeature mpHealth-2.2 --noCache` | 安装 MicroProfile Health 2.2 特性，但不将该特性缓存到本地 Maven 存储库中。 |
| `featureUtility installServerFeatures myserver --noCache` | 为`myserver`服务器安装服务器特性，而不将特性缓存到本地 Maven 存储库中。 |
| `featureUtility installFeature adminCenter-1.0 --acceptLicense` | 从 Maven Central 安装管理中心功能。 |
| `featureUtility installServerFeatures defaultServer --verbose` | 为启用调试的`myserver`服务器安装功能。 |
| `featureUtility viewSettings` | 查看您的`featureUtility.properties`文件的模板。 |
| `featureUtility find mpHealth-2.2` | 从 Maven Central 和所有已配置的 Maven 资源库中搜索 MicroProfile Health 2.2 特性。 |
| `featureUtility find` | 从 Maven Central 和所有已配置的 Maven 资源库中搜索所有可用的特性。 |

## 管理员控制应用程序的启动顺序

默认情况下，应用程序并行启动，并可以随机顺序完成启动。在 20.0.0.6 版本中，Open Liberty 允许管理员在一个或多个应用程序启动之前阻止一个应用程序启动。

能够定义应用程序启动的顺序是至关重要的，因为应用程序通常是相互依赖的。例如，一台 Open Liberty 服务器可能包含一个提供用户界面的前端应用程序和一个访问数据库的后端应用程序。在后端启动之前让前端应用程序可用可能会导致错误。有了这个特性，您可以阻止前端应用程序启动，直到后端准备就绪。

您只需要在配置中定义应用程序依赖关系，使用`application`元素上的`startAfter`属性。在此示例中，您将为后端应用程序添加一个以逗号分隔的 ID 值列表，后端应用程序必须在前端应用程序运行之前启动:

```
 <webApplication id="frontend" location="myFrontend.war" startAfter="backend1, backend2"/>
 <enterpriseApplication id="backend1" location="myBackend.ear"/>
 <enterpriseApplication id="backend2" location="myUtilities.ear"/>

```

## 开放式 Liberty Grafana 仪表板中的 REST 可视化

Grafana 仪表板提供了各种各样的微概要指标数据的时间序列可视化，如 CPU、REST、servlet、连接池和垃圾收集指标。Grafana 仪表板由 Prometheus 数据源提供支持，该数据源被配置为从一个或多个 Open Liberty 服务器的`/metrics`端点接收数据。因此，用户可以在 Grafana 仪表板上近乎实时地查看指标。

随着`mpMetrics-2.3`的发布及其 JAX-遥感指标的增加，我们为 Open Liberty Grafana 仪表盘引入了一组新的可视化工具。你可以在标签为 **REST** 的新标签下找到这些。我们还添加了悬停描述，提供了每个可视化及其功能的简要总结。这些更新适用于[OKD 3.11](https://docs.okd.io/3.11/welcome/index.html)([红帽 OpenShift](https://developers.redhat.com/openshift) 的上游分发)、[红帽 OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview) (OCP)，以及独立的 Open Liberty 实例。

如果您的开发环境中还没有安装 Grafana 和 Prometheus，请参阅 Kabanero 指南中的 [Red Hat OpenShift 容器平台 4.3](https://kabanero.io/guides/app-monitoring-ocp4.2/) 。要开始使用 Open Liberty 的 Grafana 仪表板，请参见我的博客文章介绍 [Open Liberty 对 MicroProfile 3.3](https://openliberty.io/blog/2020/04/09/microprofile-3-3-open-liberty-20004.html#gra) 的支持。

**注意**:你可以在[开放自由运营商库](https://github.com/OpenLiberty/open-liberty-operator/tree/master/deploy/dashboards/metrics)中找到 OKD 或 OCP 开放自由的 Grafana 仪表盘。

## Web 应用程序启动更改

从 Open Liberty 20.0.0.6 开始，web 应用程序只有在对`ServletContainerInitializers`和`ServletContextListeners`的调用完成后才被认为已经启动。此更新有效地将更多的应用程序初始化过程转移到服务器启动过程中。因此，应用程序和服务器似乎需要更长时间才能启动。这一更改不会影响应用程序开始处理请求所需的时间；它只是将它移动到在打开端口之前运行。此外，您现在可以配置`server.xml`，以便`ServletContextListener`中的故障会导致应用程序启动失败。只需添加以下内容:

```
<webContainer stopAppStartUponListenerException="true"/>

```

**注意**:点击了解更多关于应用属性的[。](https://openliberty.io/docs/ref/config/#application.html)

## 结论

开放自由 20.0.0.6 可以通过 Maven、Gradle、Docker 获得，也可以下载存档。今天就获取它，开始使用我在本文中描述的新特性。

*Last updated: June 25, 2020*