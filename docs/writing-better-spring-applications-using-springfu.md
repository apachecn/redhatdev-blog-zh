# 使用 SpringFu 编写更好的 Spring 应用程序

> 原文：<https://developers.redhat.com/blog/2018/12/12/writing-better-spring-applications-using-springfu>

“真理只能在一个地方找到:代码”， [Robert C. Martin，*Clean Code:A Handbook of Agile Software crafts*。](https://www.goodreads.com/book/show/3735293-clean-code)

我们构建代码的方式直接影响到代码的可理解性。易于理解且没有或很少隐藏功能的代码更易于维护。这也让我们的程序员同事更容易追踪代码中的错误。这有助于我们避免文卡特的[耶稣驱动的开发](https://aidium.se/2015/06/tdd-with-venkat/)。

我编写 Spring 应用程序的方式包括大量使用 Spring [注释。](https://springframework.guru/spring-framework-annotations/)这种方法的问题是应用程序的部分流程由注释控制。我的代码的完整流程不在一个地方，也就是在我的代码里。我需要回头看文档来理解注释的行为。仅仅通过阅读代码，很难预测控制流。

幸运的是，Spring 有了一种新的编码方式，它被称为 Spring Functional 或 [SpringFu](https://github.com/spring-projects/spring-fu) 。在本文中，我将使用 [Kotlin](https://kotlinlang.org) 来展示您从 SpringFu 获得的一些好处。

## 春天来了

让我们从一个使用注释方法的简单的基于 Spring 的应用程序开始。我们遇到的第一个注释是`@SpringBootApplication`，它做了[许多](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-using-springbootapplication-annotation.html)的事情。当然，当您使用该注释时，代码中不会捕获这些信息，正如您在下面的“带注释的代码”一节中看到的那样。当我们在带有注释的代码中继续前进时， [@RestController](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html) 做了一系列事情，这使得代码在控制流方面的可读性变得更加复杂。

`@RequestMapping`、`@Autowired`、`@Component`都是我上面说的问题。Spring 必须使用反射、Kotlin [开放](https://kotlinlang.org/docs/reference/classes.html#inheritance)类，以及各种各样的设施来使这个代码(魔法)工作。如果我的目标是阅读代码——仅仅是代码——来理解控制流，那该怎么办？

减少反射的使用也有助于我们向基于 [GraalVM](https://www.graalvm.org) 的本地代码发展。首先，看看下面带注释的代码，看看是否能得到控制流。你可以在我的 GitHub [库](https://github.com/masoodfaisal/spring-app-no-annotations)获得完整的例子。

### 带注释的代码

```
@SpringBootApplication
class SpringWithAnnotationsApplication
  fun main(args: Array<String>) {
    runApplication<SpringWithAnnotationsApplication>(*args)
  }

@RestController
@RequestMapping("/events")
class EventHandler {
  @Autowired
  lateinit var eventService: EventService

  @GetMapping("/")
  fun getAllEvents () : List<Event> {
    return eventService.getAllEvents()
  }
}

@Component
class EventService {
  fun getAllEvents() : List<Event>{
    return listOf(
                  Event(name = "event1", description = "desc1"),
                  Event(name = "event2", description = "desc2")
                 )
  }
}

data class Event (val name: String, val description: String)
```

访问 Stack Overflow(有人说 Stack Overflow 是真正的技术领先)，你可以找到许多关于正确使用注释的问题。问题的数量和种类将让您了解代码可读性有多重要。

“程序必须是写给人们阅读的，顺便提一下，是写给机器执行的，”[艾贝尔森和苏斯曼](https://en.wikiquote.org/wiki/Programming_languages)。

### 没有注释的代码

现在让我们尝试使用 SpringFu 方法编写相同的代码。代码可读性更好，我可以通过阅读来了解整个流程。下面的代码流程如下。

*   我有一个`main`函数，它加载我的`appSimple` [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) 。
*   `AppSimple` DSL *导入*Spring bean(我已经将它加载到`eventBeans` DSL 中)并使用`eventRoutes` DSL 配置服务器应该做什么。

### 

```
//create an application
//remember no @SpringBootApplication
fun main(args: Array<String>) {
  appSimple.run()
}

//using the application dsl, i define what my service will listen to
val appSimple = application {
  //use the beans i define
  import(eventBeans)

  //http server listens for this endpoint
  server {
    import(::eventRoutes)
    codecs {
      jackson()
    }
  }
}

//define beans
val eventBeans = beans {
  bean<EventService>()
  bean<EventHandler>()
}

//define on what endpont i would be listening
//@RestController
//@RequestMapping("/events")
//@GetMapping("/")
fun eventRoutes(handler: EventHandler) = router {
  "/events".nest {
    GET("/", handler::generateResponse)
  }
}

//create the handler for http request
class EventHandler(private val eventService: EventService) {
  fun generateResponse(request: ServerRequest) = ServerResponse.ok().body(
    BodyInserters.fromObject(eventService.getAllEvents())
  )
}

//business logic
// no need to @Component
class EventService {
  fun getAllEvents(): List<Event> {
    return mutableListOf(
                         Event(name = "event1", description = "desc1"),
                         Event(name = "event2", description = "desc2")
    )
  }
}

data class Event(val name: String, val description: String)
```

## 不需要开班

当使用注释时，我们需要让所有的类[打开](https://kotlinlang.org/docs/reference/compiler-plugins.html)以便 Spring 工作。参见[代码](https://github.com/masoodfaisal/spring-app-no-annotations)下面的 [pom.xml](https://github.com/masoodfaisal/spring-app-no-annotations/blob/master/spring-with-annotations/pom.xml) 部分。有了 SpringFu，[最终类](https://docs.oracle.com/javase/tutorial/java/IandI/final.html)是可以接受的。注意下面展示自动打开所有类的 *kotlin-maven-plugin* 的用法。这是我的代码之外的一些额外的逻辑，使事情难以阅读。

```
<plugin>
  <artifactId>kotlin-maven-plugin</artifactId>
  <groupId>org.jetbrains.kotlin</groupId>
  <configuration>
    <args>
      <arg>-Xjsr305=strict</arg>
    </args>
    <compilerPlugins>
      <plugin>spring</plugin>
    </compilerPlugins>
  </configuration>
  <dependencies>
    <dependency>
      <groupId>org.jetbrains.kotlin</groupId>
      <artifactId>kotlin-maven-allopen</artifactId>
      <version>${kotlin.version}</version>
    </dependency>
  </dependencies>
</plugin>
```

## 结论

SpringFu 是一个令人兴奋的新项目，它使我们能够编写更干净、可读性更好的代码，并为基于 Spring 的应用程序提供更多的控制。它建立在您现有的 Java/Kotlin 知识的基础上，这使得学习它的任务变得更加容易。令人兴奋的目标之一是能够使用 [GraalVM](https://www.graalvm.org) 编写本地应用程序。请注意，有些 SpringFu 组件还没有准备好投入生产。

Kotlin 是一种令人兴奋的新编程语言，尤其是如果你有 Java 背景的话。你可以通过参加这个 [coursera 课程](https://www.coursera.org/learn/kotlin-for-java-developers)开始你的 Kotlin 之旅。

此外，您可能会发现这些其他 Spring 文章很有帮助。

*Last updated: January 29, 2019*