# 用 Rest DSL 编写您的第一个 Camel Spring Boot 项目

> 原文：<https://developers.redhat.com/blog/2018/03/26/camel-spring-boot-rest-dsl>

Rest 服务对于系统间的通信变得越来越流行。现在红帽支持使用[红帽 JBoss Fuse](https://developers.redhat.com/products/fuse/overview/) 与[阿帕奇骆驼 Spring Boot](http://camel.apache.org/spring-boot.html) ，学习如何开始使用 [Rest DSL](http://camel.apache.org/rest-dsl.html) 和 Spring Boot。这些方向将使用 camel-servlet 组件，尽管可以使用各种组件。

与所有 Spring Boot 项目一样，您首先需要一个类来启动应用程序:

```
@SpringBootApplication
@Configuration
@ComponentScan("com.sample.camel")
public class SampleCamelApplication {
  /**
  * A main method to start this application.
  */
  public static void main(String[] args) {
    SpringApplication.run(SampleCamelApplication.class, args);
  }

  @Bean
  public ServletRegistrationBean camelServletRegistrationBean() {
    ServletRegistrationBean registration = new ServletRegistrationBean(new CamelHttpTransportServlet(), "/camel/*");
    registration.setName("CamelServlet");
    return registration;
  }
 }
```

接下来你可以写你的骆驼路线。此示例扩展了 RouteBuilder 类:

```
@Component
public class SampleCamelRouter extends RouteBuilder {
  @Override
  public void configure() throws Exception {
    restConfiguration()
      .component("servlet")
      .bindingMode(RestBindingMode.json);

    rest().get("/hello")
      .to("direct:hello");

    from("direct:hello")
      .log(LoggingLevel.INFO, "Hello World")
      .transform().simple("Hello World");
   }
}
```

现在您可以使用`mvn spring-boot:run`从命令行运行您的示例。然后你可以点击下面的网址来查看你的服务是否正常工作:`http://localhost:8080/camel/hello`

在[红帽峰会](https://www.redhat.com/en/summit/2018)上，与 Claus Ibsen 一起来到我的[工作室，了解关于其余 DSL 的更多信息，如何部署到](https://agenda.summit.redhat.com/SessionDetail.aspx?id=154680)[红帽 OpenShift](https://developers.redhat.com/products/openshift/overview/) 的这条(及更多)路由，以及如何使用 [3scale by Red Hat](https://www.redhat.com/en/technologies/jboss-middleware/3scale) 来管理您的 API。

*Last updated: January 29, 2019*