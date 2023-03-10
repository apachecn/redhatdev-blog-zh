# 使用 Eclipse MicroProfile 介绍微服务的可观察性

> 原文：<https://developers.redhat.com/blog/2019/10/01/introduction-to-microservices-observability-with-eclipse-microprofile>

微服务提供了一种现代的开发方法，它与云环境兼容，并使我们能够创建云原生应用程序。通过[微服务](https://developers.redhat.com/topics/microservices/)，我们提升了弹性、容错能力和规模；然而，微服务方法也提出了不同于单一应用程序的挑战，因为它的分布式性质。

其中一个挑战涉及到监控和日志记录，这自然就引出了可观察性的概念。在本文中，我们将看看 [Eclipse MicroProfile](https://microprofile.io/) 如何帮助您在微服务中实现可观测性。

## 什么是可观测性？

可观性的概念来源于控制理论，也就是一种数学理论。

> *形式上，一个系统被称为是**可观测的**，如果对于[状态和控制向量](https://en.wikipedia.org/wiki/State_space_representation#Linear_systems)(后者是可以选择值的变量)的任何可能序列，当前状态(底层动态演化变量的值)可以在有限时间内仅使用输出来确定。* — [维基百科](https://en.wikipedia.org/wiki/Observability)

一些开发人员将微服务架构中的可观察性定义为度量、日志和跟踪工具的集合，但我认为可观察性是一个更一般的概念。度量、日志和跟踪是提供可观察性的简单方法。

对我来说，可观察性是一个系统以快速简单的方式暴露其状态的精确信息的能力。与监控不同，可观察性是关于系统的。当我们谈论监控时，重点是用于监控系统的工具，这些工具可能很容易监控，也可能很难监控。当我们谈论可观察性时，焦点是系统本身，以及以更容易和更快的方式提供这些信息的需要。

## 微轮廓的可观察性

为了支持简单的可观察性，MicroProfile 有一些规范，允许开发人员在微服务中实现可观察性。MicroProfile 对此有三个主要规范:Microprofile OpenTracing、MicroProfile Metrics 和 micro profile health check——所有这些我们都将在这里看到。

![Screenshot from 2019-02-27 16-44-31](img/71ddf8ba5e74de62b5da70157da2746f.png)

### 微文件打开跟踪

[MicroProfile OpenTracing](https://microprofile.io/project/eclipse/microprofile-opentracing) 规范允许我们使用分布式跟踪，使用 OpenTracing API 来跟踪跨服务的请求流。这个规范与 Zipkin 和 Jaeger 兼容，它允许我们使用这些工具来显示关于分布式跟踪的信息。下面是一个如何使用 MicroProfile OpenTracing 的例子。

```
@Path("subjects")
@Traced
public  SubjectEndpoint { 
```

### 微轮廓度量

[MicroProfile Metrics](https://microprofile.io/project/eclipse/microprofile-metrics) 是一个规范，它允许我们公开关于我们的应用程序的度量信息。有了这个，我们可以更快更容易地公开精确的指标。下面是一个如何使用 MicroProfile 度量的例子。

```
@Counted
public CounterBean() {
}
```

### 微文件健康检查

[MicroProfile health check](https://microprofile.io/project/eclipse/microprofile-health)规范允许我们暴露应用程序在我们的环境中是运行还是关闭。它对问题“ 我的应用程序还能正常运行吗？”作出布尔响应(是或否)? ”。下面是如何使用 MicroProfile HealthCheck 的示例。

```
@Health
@ApplicationScoped
public class ApplicationHealthCheck implements HealthCheck {

 @Override
 public HealthCheckResponse call() {
 return HealthCheckResponse
 .named("application-check").up()
 .withData("CPUAvailable", Runtime.getRuntime().availableProcessors())
 .withData( "MemoryFree", Runtime.getRuntime().freeMemory())
 .withData("TotalMemory", Runtime.getRuntime().totalMemory())
 .build();
 }
} 
```

## 结论

Eclipse MicroProfile 为微服务挑战提供了几种解决方案，包括各种规范来提高我们微服务的可观察性。有了这些规范，我们可以将 MicroProfile 与 Jaeger、Zipkin、Prometheus 和其他工具配合使用，以提高可观测性和监控能力。我将在接下来的文章中提供这些规范的更多细节以及如何使用它们。

*Last updated: July 1, 2020*