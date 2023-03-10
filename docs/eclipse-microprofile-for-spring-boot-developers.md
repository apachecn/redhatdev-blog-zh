# 面向 Spring Boot 开发者的 Eclipse MicroProfile

> 原文：<https://developers.redhat.com/blog/2018/11/21/eclipse-microprofile-for-spring-boot-developers>

到现在为止，您可能已经听说过 [Eclipse 微概要文件](http://microprofile.io/) (MP)。这是一个社区驱动的倡议，为企业 Java [微服务](https://developers.redhat.com/topics/microservices/)定义规范。MicroProfile 只有两年的历史，但是它已经发布了八个创新的规范，并且发展很快。它提供了度量、API 文档、健康检查、容错、分布式跟踪等等。有了它，您可以充分利用尖端的云原生技术，并以供应商中立的方式实现！

对于熟悉 Spring Boot 的开发人员，我们准备了这篇文章，它比较了使用 Spring Boot 和 MicroProfile 开发应用程序的基础。我们编写了两个应用程序，每个解决方案一个。在这篇文章中，我们将通过他们之间的差异。你可以在 [GitHub](https://github.com/michalszynkiewicz/from-spring-to-microprofile) 上找到两个项目的源代码。

对于 MicroProfile 应用程序，我们使用 [Thorntail](https://developers.redhat.com/blog/2018/10/17/announcing-thorntail-2-2-general-availability/) (以前称为 Wildfly Swarm)，但除了设置部分，Open Liberty、Payara、TomEE 或任何其他实现看起来都完全一样。

在整篇文章中，我们假设您了解 Spring Boot，我们将重点关注 MicroProfile 中的不同之处。

## 设置项目

我们使用 Maven 设置了这两个应用程序。

使用 Thorntail，项目的设置与 Spring Boot 非常相似。第一个区别是在项目包装上。当我们对一个 Spring Boot 应用程序使用`jar`(默认打包)时，我们需要将其设置为`war`来打包一个 Thorntail 应用程序。

我们的 Spring Boot 应用程序使用一个名为`spring-boot-dependencies`的 BOM 文件。Thorntail 也提供 BOM 文件。我们选择了`bom`，它列出了所有稳定的、经过良好测试的 Thorntail 元素。例如，如果你喜欢实验，你可以用`bom-all`来代替。

Spring Boot 和索恩泰尔都使用 Maven 插件将用户的类、资源和所选解决方案的所有部分打包成一个大罐子。对于 Thorntail 来说，这个插件叫做`thorntail-maven-plugin`。下面的清单显示了它的声明以及 Thorntail BOM 的声明。

```
<project ...>
 ...
 <packaging>war</packaging>

<dependencyManagement>
 <dependencies>
   <dependency>
     <groupId>io.thorntail</groupId>
     <artifactId>bom</artifactId>
     <version>${version.thorntail}</version>
     <type>pom</type>
     <scope>import</scope>
   </dependency>
 </dependencies>
</dependencyManagement>
...
<build>
 <plugins>
   <plugin>
     <groupId>io.thorntail</groupId>
     <artifactId>thorntail-maven-plugin</artifactId>
     <version>${version.thorntail}</version>
     <executions>
       <execution>
         <goals>
           <goal>package</goal>
         </goals>
       </execution>
     </executions>
   </plugin>
 </plugins>
</build>

```

定义了基础知识之后，我们可以选择我们需要的特性。

对于 Spring Boot，我们选择了`spring-boot-starter-web`，我们简单地为 Thorntail 添加了对`microprofile`的依赖。

```
<dependency>
 <groupId>io.thorntail</groupId>
 <artifactId>microprofile</artifactId>
</dependency>

```

你可以看看 GitHub 项目中最后的`pom.xml`文件[。](https://github.com/michalszynkiewicz/from-spring-to-microprofile/blob/master/microprofile/pom.xml)

在设置项目时还有一个不同之处:静态资源应该放置的位置。我们的 Spring Boot 应用程序将静态资源保存在`src/main/resources/static`目录中，而 Thorntail 应用程序要求将它们放在`src/main/webapp`中。

## 公开一个 REST 端点

从 Spring 应用程序中公开 REST API 最惯用的方式是使用 Spring MVC。MicroProfile 利用 JAX 遥感器实现了这一目的。从一个到另一个的翻译非常简单。两者都处理注释。

首先，对于 JAX-RS，需要一个`Application`级。它扩展了`javax.ws.rs.core.Application`，可以为所有 JAX-RS 端点提供全局路径前缀。我们的情况是:

```
@ApplicationPath("/api")
public class ApplicationConfig extends Application {
}

```

接下来就是翻译资源类注释的问题了。

下面的 Spring MVC 代码:

```
@RestController
@RequestMapping(value = "/api/greeting", produces = MediaType.APPLICATION_JSON_VALUE)
public class GreetingController {

   @GetMapping
   public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
      ...
   }

```

转换成以下微文件代码:

```
@Path("/greeting") // 1
@Produces(MediaType.APPLICATION_JSON) // 2
public class GreetingResource {

   @GET // 3
   public Response greeting(@QueryParam("name") @DefaultValue("World") String name) {  // 4
      ...
   }

```

分解一下:

1.  JAX-RS 端点类应该用`@Path`来注释。该注释也是为端点提供任何前缀的地方。
2.  如果我们想要指定响应的内容类型，必须使用单独的注释`@Produces`。它有一个名为`@Consumes`的对应物来指定接受的请求实体类型。
3.  此外，处理请求的方法需要用`@GET`、`@POST`、`@DELETE`、`@PUT`或`@HEAD`进行注释，这取决于它们要处理的 HTTP 方法。
4.  我们的端点使用一个查询参数。为了让它工作，我们在 Spring MVC 中使用了一个`@RequestParam`注释。对于 MicroProfile，要使用的注释是`@QueryParam`。为了提供默认值，我们添加了一个`@DefaultValue`注释。

其他可用注释见`javax.ws.rs`包。

提到`@Context`注释也很重要。有了它，您可以向方法或端点注入诸如 HTTP 头或安全上下文之类的值。

## 依赖注入

Spring 的核心是它的依赖注入。使用 MicroProfile，您可以使用 CDI 来代替。

这主要意味着使用`@Inject`注释代替 Spring 的`@Autowired`注释:

```
   @Inject
   private GreetingGenerator generator;

```

类似于 Spring，CDI 有一个 bean 的概念，这些 bean 生活在一定的范围内。用于控制 Spring beans 范围的`@Scope`注释被翻译成`@ApplicationScoped`、`@RequestScoped`等中的一个。CDI 注释。

这里有一个来自`GreetingGenerator`类的例子:

```
@ApplicationScoped
public class GreetingGenerator {
 …
}

```

查看`javax.enterprise.context`包中所有可用选项的列表。

翻译很简单。唯一的问题是默认范围是不同的。而在 Spring 中，默认作用域是 *singleton* ，在 CDI 中是`@Dependent`——一个对应于 Spring 的*原型*的作用域。

## 配置

在我们的 Spring 应用程序中，我们用`@Value`注释注入配置值。

使用微配置文件配置，我们可以通过以下方式达到同样的目的:

```
@Inject
@ConfigProperty(name = "greeting.message")
private String message;

```

上述消息字段的值将取自`META-INF/microprofile-config.properties`。它可以被环境变量或系统属性覆盖。MicroProfile Config 还提供了一个简单的机制来定义一个定制的`ConfigSource`，一个额外的配置值来源。

## 执行

让我们都试试吧！

要获得代码，克隆 GitHub 存储库:

```
git clone https://github.com/michalszynkiewicz/from-spring-to-microprofile/

```

存储库在两个独立的目录中包含两个项目。要构建它们中的任何一个，请导航到适当的目录并运行以下命令:

```
mvn clean package

```

Spring Boot 和索恩泰尔都创造了一个超级罐子。要运行 Thorntail，进入目标目录并执行以下命令:

```
java -jar microprofile-from-spring-1.0-SNAPSHOT-thorntail.jar

```

现在，当您在浏览器中访问 http://localhost:8080 时，您应该看到一个由所选应用程序显示的 web 页面。可以通过浏览器试用应用或者直接访问 REST API http://localhost:8080/API/greeting？把你的名字写在这里。

## 不仅仅是一个对手

虽然这两个示例应用程序的代码非常相似，但 MicroProfile one 有更多的功能。

如所配置的，微剖析应用程序另外暴露:

*   REST 端点的 OpenAPI 文档，位于 http://localhost:8080/openapi
*   应用程序指标，包括线程数量、堆使用情况等。在 http://localhost:8080/metrics 上
*   健康检查，可与 Kubernetes 一起使用，位于 http://localhost:8080/health check

此外，在不修改配置的情况下，它可以利用其他微文件规范，比如容错或类型安全的 REST 客户端。

## 进一步阅读

如果你想了解更多，你可以下载一个免费的电子书: *[用 Enterprise Java 构建微服务:Eclipse MicroProfile](http://bit.ly/MP-ebook)* 实用指南。

**Thorntail 和 Eclipse Microprofile 资源:**

*   詹姆斯·福克纳的文章 *[宣布 Thorntail 2.2 正式上市](https://developers.redhat.com/blog/2018/10/17/announcing-thorntail-2-2-general-availability/)* 提供了 Thorntail 的概述，以及如何开始使用，包括在在线环境中试用，无需下载。
*   *[Eclipse MicroProfile 和 Red Hat 更新:Thorntail 和 small rye](https://developers.redhat.com/blog/2018/08/23/eclipse-microprofile-and-red-hat-update-thorntail-and-smallrye/)*
*   [*Eclipse 1.3 版本中的微文件状态*](https://developers.redhat.com/blog/2018/05/07/microprofile-status-version-1-3/) 提供了微文件中 API 规范的概述
*   [*用 Eclipse Microprofile 1.2 进行云原生开发*](https://developers.redhat.com/blog/2018/01/29/microprofile-1-2-cloud-native-development/)
*   Thorntail 项目网站: [thorntail.io](https://thorntail.io/)

*Last updated: January 29, 2019*