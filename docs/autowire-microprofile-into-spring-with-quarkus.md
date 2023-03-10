# 用夸克将微轮廓自动连接到 spring

> 原文：<https://developers.redhat.com/blog/2019/10/02/autowire-microprofile-into-spring-with-quarkus>

在开发 [Java](https://developers.redhat.com/topics/enterprise-java/) [微服务](https://developers.redhat.com/topics/microservices/)时，Eclipse MicroProfile 和 [Spring Boot](https://developers.redhat.com/topics/spring-boot/) 通常被认为是独立且不同的 API。开发人员通过利用他们日常使用的 API 来默认他们的精神肌肉记忆。学习新的框架和运行时可能是一项重大的时间投资。本文旨在通过使 Spring 开发者能够利用他们已经知道的 Spring APIs，同时受益于 Quarkus 提供的重要新功能，来简化对 Spring 开发者的一些流行的 MicroProfile APIs 的介绍。

更具体地说，本文涵盖了 Quarkus 支持的 Spring APIs 的范围和细节，这样 Spring 开发人员就可以掌握他们可以利用 MicroProfile APIs 构建的基础。然后，本文介绍了 MicroProfile APIs，Spring 开发人员会发现这些 API 有助于微服务的开发。只涵盖了微文件的一个子集。

为什么是夸库斯？一个原因是实时编码，任何改变都会自动重新加载，无论是 MicroProfile、Spring 还是其他任何 Java API。只需运行`mvn quarkus:dev`。就是这样。第二个令人信服的原因是，[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)的 Person 服务使用 GraalVM 的本机映像将 Spring、MicroProfile 和 JPA APIs 编译成本机二进制文件，它在 0.055 秒内启动，并在到达应用程序 RESTful 端点后使用大约 90MB 的 RAM (RSS)。运行`mvn package -Pnative`编译成本地二进制文件。就是这样。

本文不会进行详细的比较，但是会让 Spring 开发人员了解 Spring 和 MicroProfile APIs 如何与 Quarkus 一起使用。

## 容器和 Kubernetes

为了使本文简短(er ),本文将只从高层次上介绍 Kubernetes 支持，但是简要讨论一下是很重要的。Quarkus 的关键价值主张之一是“Kubernetes-native Java”，其目标是最小化内存占用并减少启动时间。内存占用的减少有助于提高相同硬件上的应用密度，从而降低总体成本。

Quarkus 还[支持 Kubernetes 资源的自动生成](https://quarkus.io/guides/kubernetes-resources)，并且[指南](https://quarkus.io/guides/)也可以部署到 Kubernetes 和[红帽 OpenShift](https://developers.redhat.com/openshift/) 上。此外，会自动生成 Dockerfile.jvm (JVM 打包)和 Dockerfile.native(本机二进制打包)来创建容器。

最后，鉴于 Quarkus 认为 Kubernetes 是一个目标部署环境，当 Kubernetes 固有的功能可用时，它放弃使用 Java 框架。表 1 简要地映射了 Spring 开发人员通常使用的 Java 框架和 Kubernetes 的内置功能。

**表 1: Java 框架到 Kubernetes 的映射**

服务，复制控制器(“服务器端”)

| **能力** | **传统 Spring Boot** | **Kubernetes** |
| 服务发现 | 尤里卡 | 域名服务器(Domain Name Server) |
| 配置 | 春季云配置 | 配置映射/机密 |
| 负载平衡 | 功能区(“客户端”) | 服务，复制控制器(“服务器端”) |

## 编译和运行示例代码

本文附有一个[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)，它在同一个项目中同时使用了 Spring 和 MicroProfile APIs，甚至是同一个 Java 类。代码可以从命令行编译和运行。请务必阅读 README.md 以获取说明。

## Spring 框架 API

### 依赖注入

Quarkus 支持许多[上下文和依赖注入(CDI)API](https://quarkus.io/guides/cdi-reference)和 Spring 依赖注入(Spring DI)API。MicroProfile、 [Java EE 和 Jakarta EE](https://developers.redhat.com/blog/2019/09/12/jakarta-ee-8-the-new-era-of-java-ee-explained/) 开发人员将非常熟悉 CDI。另一方面，Spring 开发者可以使用 Spring DI API 的 *Quarkus 扩展*来实现 Spring DI 兼容性。表 2 给出了一个受支持的 Spring DI APIs 示例。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)同时利用了 CDI 和 Spring 依赖注入，Quarkus [Spring DI 指南](https://quarkus.io/guides/spring-di-guide)通过其他示例进行了更详细的介绍。

**表 2:受支持的 Spring DI APIs 示例**

| **春迪** 支持的功能 | **例题** |
| 构造函数注入 | 

```
public PersonSpringController(
   PersonSpringRepository personRepository,  // injected      
   PersonSpringMPService personService) {    // injected
      this.personRepository = personRepository;
      this.personService = personService;
}
```

 |
| 现场注入
@自动连线
@值 | 

```
@Autowired
@RestClient
SalutationRestClient salutationRestClient;

@Value("${fallbackSalutation}")
String fallbackSalutation;
```

 |
| @ Bean
@配置 | 

```
@Configuration
public class AppConfiguration {
   @Bean(name = "capitalizeFunction")
   public StringFunction capitalizer() {
      return String::toUpperCase;
   }
}
```

 |
| @组件 | 

```
@Component("noopFunction")
public class NoOpSingleStringFunction implements StringFunction {
   @Override
   public String apply(String s) {
      return s;
   }
}
```

 |
| @服务 | 

```
@Service
public class MessageProducer {
   @Value("${greeting.message}")
   String message;

   public String getPrefix() {
      return message;
   }
}
```

 |

### Web 框架

MicroProfile 开发人员会对 Quarkus 支持 JAX-RS、MicroProfile Rest 客户端、JSON-P 和 JSON-B 作为核心 web 编程模型感到满意。Spring 开发者可能会惊讶于 Quarkus 最近增加了 Spring Web API 支持，特别是围绕 Spring REST 相关的 API。与 Spring DI 一样，Spring Web API 支持的目标是让 Spring 开发人员在一起使用 Spring Web API 和 MicroProfile APIs 时有宾至如归的感觉。表 3 给出了一个受支持的 Spring Web APIs 示例。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)利用了 Spring Web 和 MicroProfile Rest 客户端 API，Quarkus [Spring Web Guide](https://quarkus.io/guides/spring-web-guide) 通过其他示例进行了更详细的介绍。

**表 3:受支持的 Spring Web APIs 示例**

| **春网** **支持的功能** | **例题** |
| @ rest controller
@ request mapping | 

```
@RestController
@RequestMapping("/person")
public class PersonSpringController {
   ...
   ...
   ...
}
```

 |
| @ get mapping
@ post mapping
@ put mapping
@ delete mapping
@ patch mapping
@ request param
@ request header
@ matrix variable
@ path variable
@ CookieValue
@ request body
@ response status
@ exception handler
@ RestControllerAdvice(partial) | 

```
@GetMapping(path = "/greet/{id}",
   produces = "text/plain")
   public String greetPerson(
   @PathVariable(name = "id") long id) {
   ...
   ...
   ...
}
```

 |

### 春季数据 JPA

使用 Hibernate ORM，MicroProfile 开发人员会对 Quarkus JPA 支持感到满意。春天的开发者们，不要害怕！Quarkus 支持常用的 Spring 数据 JPA 注释和类型。表 4 涵盖了一个受支持的 Spring 数据 JPA APIs 示例。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile) 利用了 Spring 数据 JPA 存储库 API，Quarkus [Spring 数据 JPA Guide](https://quarkus.io/guides/spring-data-jpa-guide) 通过其他示例进行了更详细的介绍。

**表 4:受支持的 Spring 数据 JPA APIs 示例**

| **Spring Data JPA** **支持的功能** | **例题** |
| CrudRepository | 

```
public interface PersonRepository
         extends JpaRepository,
                 PersonFragment {
   ...
}
```

 |
| RepositoryJpaRepository分页和分类存储库 | 

```
public class PersonRepository extends 

    Repository {

    Person save(Person entity);

    Optional findById(Person entity);
}
```

 |
| 储存库碎片 | 

```
public interface PersonRepository
         extends JpaRepository,
                 PersonFragment {
   ...
}
```

 |
| Derived query methods | 

```
public interface PersonRepository extends CrudRepository {

    List findByName(String name);

    Person findByNameBySsn(String ssn);

    Optional 
       findByNameBySsnIgnoreCase(String ssn);

    Boolean existsBookByYearOfBirthBetween(
            Integer start, Integer end);
}
```

 |
| 用户自定义查询 | 

```
public interface MovieRepository
         extends CrudRepository {

    Movie findFirstByOrderByDurationDesc();

    @Query("select m from Movie m where m.rating = ?1")
    Iterator findByRating(String rating);

    @Query("from Movie where title = ?1")
    Movie findByTitle(String title);
}
```

 |

## 微配置文件 API

### 容错

容错模式对于防止级联故障和创建可靠的微服务架构至关重要。多年来，断路一直是 Spring 开发人员的“首选”容错模式。然而，Hystrix 处于维护模式。微文件容错正在积极开发中，开发人员已经在生产中使用它很多年了。Quarkus 建议使用 MicroProfile 容错 API 来提高服务可靠性。表 5 给出了一个 MicroProfile 容错 API 的例子。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)使用 MicroProfile 容错 API，特别是@Timeout 和@Fallback。Quarkus [容错指南](https://quarkus.io/guides/fault-tolerance-guide)通过更多的例子进行了更详细的介绍。

**表 MicroProfile 容错 API 示例**

| **微文件容错** **特性** | **描述** | **例题** |
| @异步 | 在单独的线程上执行逻辑 | 

```
@Asynchronous
@Retry
public Future<String> getSalutation() {
   ...
   return future;
}
```

 |
| @隔板 | 限制并发请求的数量 | 

```
@Bulkhead(5)
public void fiveConcurrent() {
   makeRemoteCall(); //...
}
```

 |
| @断路器 | 优雅地处理故障和故障恢复 | 

```
@CircuitBreaker(delay=500   *// milliseconds*
   failureRatio = .75,
   requestVolumeThreshold = 20,
   successThreshold = 5)
@Fallback(fallbackMethod = "fallback")
public String getSalutation() {
   makeRemoteCall(); *//...*
}
```

 |
| @回退 | 失败时的替代逻辑 | 

```
@Timeout(500) *// milliseconds*
@Fallback(fallbackMethod = "fallback")
public String getSalutation() {
   makeRemoteCall(); *//...*
}

public String fallback() {
   return "hello";
}

```

 |
| @重试 | 重试请求 | 

```
@Retry(maxRetries=3)
public String getSalutation() {
   makeRemoteCall(); //...
}
```

 |
| @超时 | 假设失败前的等待时间 | 

```
@Timeout(value = 500 )   *// milliseconds*
@Fallback(fallbackMethod = "fallback")
public String getSalutation() {
   makeRemoteCall(); //...
}
```

 |

### 服务健康

像 Kubernetes 这样的平台利用探针来检查容器的健康状况。Spring 开发人员利用定制的健康指示器和 Spring Boot 执行器向底层平台公开服务的健康状况。有了 Quarkus，Spring 开发人员可以利用 MicroProfile Health 来公开服务的健康状况。提供了默认的活性检查，开发人员也可以提供定制的活性和就绪性检查。表 6 给出了一个 MicroProfile 健康 API 的例子。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile) 使用 MicroProfile Health 来展示应用程序的就绪状态。夸尔库斯[健康指南](https://quarkus.io/guides/health-guide)通过额外的例子进行了更详细的介绍。

**表 MicroProfile Health APIs 示例**

| **微档案健康** **特色** | **描述** | **例题** |
| @活跃度 | Platform will reboot unhealthy containerized applications.端点:
主机:8080/健康/直播 | 

```
@Liveness
public class MyHC implements HealthCheck {
  public HealthCheckResponse call() {

   ...
   return HealthCheckResponse
     .named("myHCProbe")
     .status(ready ? true:false)
     .withData("mydata", data)
     .build();  
}
```

 |
| @准备就绪 | Platform will not direct traffic to containerized applications that are not ready.端点:
主机:8080/健康/就绪 | 

```
@Readiness
public class MyHC implements HealthCheck {
  public HealthCheckResponse call() {

   ...
   return HealthCheckResponse
     .named("myHCProbe")
     .status(live ? true:false)
     .withData("mydata", data)
     .build();  
}
```

 |

### 韵律学

应用程序出于操作原因(如性能 SLA)和非操作原因(如业务 SLA)公开指标。Spring 开发人员通常利用 Spring Boot 执行器和千分尺来暴露度量。Quarkus 利用微概要度量来公开基础(JVM &操作系统)、供应商(Quarkus)和应用程序度量。MicroProfile Metrics 要求实现支持 JSON 和 OpenMetrics (Prometheus)输出格式。表 7 包含了一个 MicroProfile Metrics APIs 的例子。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)使用 MicroProfile 度量来公开应用程序度量。Quarkus [指标指南](https://quarkus.io/guides/metrics-guide)通过更多的例子进行了更详细的介绍。

**表 MicroProfile Metrics APIs 示例**

| **微剖析指标** **特征** | **描述** | **例题** |
| @数过了 | 表示对注释对象的调用进行计数的计数器。 | 

```
@Counted(name = "fallbackCounter", 
  displayName = "Fallback Counter", 
  description = "Fallback Counter")
public String salutationFallback() {
   return fallbackSalutation;
}
```

 |
| @并发量规 | 表示对注释对象的并行调用进行计数的度量。 | 

```
@ConcurrentGuage(
  name = "fallbackConcurrentGauge", 
  displayName="Fallback Concurrent", 
  description="Fallback Concurrent")
public String salutationFallback() {
   return fallbackSalutation;
}
```

 |
| @仪表 | 表示一个量规，它对注释对象的
值进行采样。 | 

```
@Metered(name = "FallbackGauge",
   displayName="Fallback Gauge",
   description="Fallback frequency")
public String salutationFallback() {
   return fallbackSalutation;
}
```

 |
| @计量 | 表示跟踪被注释对象的调用频率的计量器。 | 

```
@Metered(name = "MeteredFallback",
   displayName="Metered Fallback",
   description="Fallback frequency")
public String salutationFallback() {
   return fallbackSalutation;
}
```

 |
| @公制 | 当请求注入或产生一个度量到
时，包含元数据
信息的注释。 | 

```
@Metric
@Metered(name = "MeteredFallback",
   displayName="Metered Fallback",
   description="Fallback frequency")
public String salutationFallback() {
   return fallbackSalutation;
}
```

 |
| @定时 | 表示一个计时器，用于跟踪注释对象的持续时间。 | 

```
@Timed(name = "TimedFallback",
   displayName="Timed Fallback",
   description="Fallback delay")
public String salutationFallback() {
   return fallbackSalutation;
}
```

 |
| **指标终点** |
| 应用指标 | http://localhost:8080/metrics/application |
| 基本指标 | http://localhost:8080/metrics/base |
| 供应商指标 | http://本地主机:8080/metrics/vendor |
| 所有指标 | http://本地主机:8080/metrics |

### 微文件 Rest 客户端

微服务通常公开 RESTful 端点，需要客户端 API 来消费 RESTful 端点。Spring 开发人员通常使用 RestTemplate 来消费 RESTful 端点。Quarkus 支持 MicroProfile Rest 客户端 API 来做同样的事情。表 8 展示了一个 MicroProfile Rest 客户端 API 的示例。

[示例项目](https://github.com/jclingan/quarkus-spring-microprofile)使用 MicroProfile Rest 客户端来消费 RESTful 端点。Quarkus [Rest 客户端指南](https://quarkus.io/guides/rest-client-guide)提供了更多细节和示例。

**表 MicroProfile Rest 客户端 API 示例**

| **微档案****休息客户端** **功能** | **描述** | **例题** |
| @RegisterRestClient | 将类型化的 Java 接口注册为 REST 客户机 | 

```
@RegisterRestClient
@Path("/")
public interface MyRestClient {
    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String getSalutation();
}
```

 |
| @RestClient | 修饰类型化 REST 客户端接口的实例注入 | 

```
@Autowired // or @Inject
@RestClient
MyRestClient restClient;
```

 |
| 祈祷 | 调用 REST 端点 | 

```
System.out.println(
   restClient.getSalutation());
```

 |
| MP rest/URL | 指定 rest 端点 | 

```
application.properties:
org.example.MyRestClient/mp-rest/url=
   http://localhost:8081/myendpoint
```

 |

## 摘要

本文主要为 Spring 开发人员提供了一个概述，介绍了如何将 Spring APIs 和 MicroProfile APIs 与 Quarkus 一起使用。Spring 开发人员现在可以使用他们所知道和喜爱的一些 API，结合 MicroProfile APIs，对 Java 微服务进行实时编码，然后编译成节省 100 MB RAM 的本机二进制代码，而启动只需几毫秒。

**注意:**[quar kus 指南](https://quarkus.io/guides/)提供了更多关于 Spring 和 MicroProfile API 支持的细节，等等。

*Last updated: July 1, 2020*