# WildFly Swarm 中的微轮廓容错

> 原文：<https://developers.redhat.com/blog/2018/03/08/microprofile-fault-tolerance-wildfly-swarm>

每个开发人员的目标都是尽可能构建最具弹性的应用程序。由于微服务的分布式性质，弹性和优雅地处理故障是强制性的。Java 生态系统有一些不错的容错框架，比如 [Hystrix](https://github.com/Netflix/Hystrix) 或者 [Failsafe](https://github.com/jhalterman/failsafe) 。然而，这些都没有提供标准的 API，所以使用它们意味着您的应用程序将与该框架紧密耦合。MicroProfile 规范的主要动机是提供标准的 API 来消除紧密耦合并提高部署灵活性。本文将描述微文件容错规范的主要特性，然后展示它是如何在 [WildFly Swarm](http://wildfly-swarm.io/) 中实现的，Red Hat 微文件实现。

## 关于 Eclipse 微概要文件容错

[Eclipse micro profile Fault Tolerance](https://github.com/eclipse/microprofile-fault-tolerance)是 [Eclipse MicroProfile](https://microprofile.io) 规范之一，它提供了一种标准且简单的方法来为您的微服务或其他 Java EE 开发增加弹性。
注:参见 Jeff Mesnil 关于[用 MicroProfile 开发云原生应用](https://developers.redhat.com/blog/2018/03/05/cloud-native-microprofile-config-healthcheck-openshift/)的文章

像大多数微概要规范一样，容错基于上下文和依赖注入(CDI ),更准确地说是基于 CDI 拦截器实现。它还依赖微配置文件配置规范来允许容错策略的外部配置。

规范的主要思想是将业务逻辑从容错样板代码中分离出来。为了实现这一点，规范定义了拦截器绑定注释，以便在方法执行或类上应用容错策略(在这种情况下，所有的类方法都有相同的策略)。

容错规范中包含的策略如下:

*   **超时:**应用了`@Timeout`标注。它为当前操作添加了一个超时。
*   **重试:**应用`@Retry`标注。它添加了重试行为，并允许对当前操作进行配置。
*   **回退:**应用了`@Fallback`标注。它定义了当前操作失败时要执行的代码。
*   **舱壁:**应用`@Bulkhead`标注。它隔离当前操作中的故障，以保留其他操作的执行。
*   **断路器:**应用`@CircuitBreaker`标注。它提供了自动快速失败执行，以防止系统过载。
*   **异步:**应用了`@Asynchronous`标注。它使当前操作异步(即代码将被异步调用)。

应用这些策略中的一个或多个就像在您想要启用这些策略的方法(或类)上添加所需的注释一样简单。因此，使用容错相当简单。但是这种简单性并不妨碍灵活性，因为每个策略都有配置参数。

现在，以下供应商正在提供该规范的实现。

*   野花成群中的红帽
*   WebSphere Liberty 中的 IBM
*   Payara 服务器中的 Payara
*   吊床和托米的阿帕奇安全装置
*   KumuluzEE 框架的 KumuluzEE

在这篇文章中，我们将关注[野生蜂群](http://wildfly-swarm.io/)的实现和使用。

## 在野生蜂群中使用微配置文件容错

WildFly Swarm 中包含的 [![](img/486531bc9d2928bee28589770a47719b.png)](https://developers.redhat.com/blog/wp-content/uploads/2016/06/swarm_logo_final.png) MicroProfile 容错实现基于网飞开发的 [Hystrix](https://github.com/Netflix/Hystrix) 框架。

要将容错添加到 WildFly Swarm 项目中，您只需将容错部分添加到项目中，如下所示:

```
<dependency>
   <groupId>org.wildfly.swarm</groupId>
   <artifactId>microprofile-fault-tolerance</artifactId>
</dependency>
```

无需添加其他部分，容错部分会将其所有依赖项添加到部署中。

## 微配置文件容错在起作用

正如我们在上面看到的，使用容错非常简单。spec API 提供了一组注释，您必须对类或方法应用这些注释来实施容错策略。也就是说，您必须记住，这些注释是拦截器绑定，因此只能在 CDI bean 上使用，所以在对它们或它们的方法应用容错注释之前，要小心地将您的类定义为 CDI bean。

在下面几节中，您将找到每个容错注释的使用示例。

### @异步

使操作异步非常简单:

```
@Asynchronous
public Future<Connection> service() throws InterruptedException {
    Connection conn = new Connection() {
      {
        Thread.sleep(1000);
      }
    @Override
    public String getData() {
      return "service DATA";
    }
  };
  return CompletableFuture.completedFuture(conn);
}
```

唯一的约束是让`@Asynchronous`方法返回一个`Future`，否则实现会抛出一个异常。

### @重试

如果操作失败，您可以应用重试策略再次调用该操作。`@Retry`注释可以在类或方法级别使用，如下所示:

```
@Retry(maxRetries = 5, maxDuration= 1000, retryOn = {IOException.class})
public void operationToRetry() {
    ...
}
```

在上例中，操作最多只能在`IOException`重试 5 次。如果所有重试的总持续时间超过 1000 毫秒，操作将被中止。

### @回退

此批注只能应用于方法，批注类会产生意外的结果:

```
@Retry(maxRetries = 2)
@Fallback(StringFallbackHandler.class)
public String shouldFallback() {
	 ...
}
```

达到重试次数后调用 Fallback 方法。在上面的示例中，如果出现错误，该方法将重试两次，然后回退将用于调用另一段代码。

回退代码可以由实现 FallbackHandler 接口的类定义(参见上面的代码)，或者由当前 bean 中的方法定义。

### @超时

这个注释可以应用在类或方法上，以确保操作不会永远持续下去。

```
@Timeout(200) 
public void operationCouldTimeout() {
    ...
}
```

在上例中，如果持续时间超过 200 毫秒，操作将会停止。

### @断路器

注释可以应用于类或方法。Martin Fowler 引入了断路器模式，通过在功能障碍的情况下使操作快速失败来保护操作的执行。

```
@CircuitBreaker(requestVolumeThreshold = 4, failureRatio=0.75, delay = 1000)
public void operationCouldBeShortCircuited(){
  ...
}
```

在上面的示例中，该方法应用了断路器策略。如果在 4 次连续调用的滚动窗口中出现 3 次(4 x 0.75)故障，电路将被断开。电路将保持开路 1000 毫秒，然后回到半开状态。成功调用后，电路将再次闭合。

### @隔板

这个注释也可以应用在类或方法上，以加强隔离策略。此模式隔离当前操作中的故障，以保留其他操作的执行。该实现通过限制给定方法的并发调用数量来实现这一点。

```
@Bulkhead(4)
public void bulkheadedOperation() {
       ...
}
```

在上面的代码中，这个方法只支持同时调用 4 次。
隔板也可与`@Asynchronous`一起使用，以限制异步操作中的线程数量。

## 使用 MP 配置配置容错

正如我们在前面几节中看到的，容错策略是通过使用注释来应用的。对于大多数用例来说，这已经足够了，但是对于其他用例来说，这种方法可能并不令人满意，因为配置是在源代码级别完成的。

这就是为什么可以使用微文件配置来覆盖微文件容错注释的参数。

注释参数可以通过 **<类名> / <方法名> / <注释> / <参数>** 的命名约定中的配置属性进行覆盖。

要在`MyService`类的`doSomething`方法上覆盖`@Retry`的`maxDuration`,设置 config 属性如下:

`com.redhat.microservice.MyService/doSomething/Retry/maxDuration=3000`

如果特定注释的参数需要为特定类配置相同的值，请使用 config 属性: **<类名> / <注释> / <参数>** 进行配置。

例如，使用下面的 config 属性将类`MyService`上指定的`@Retry`的所有`maxRetries`覆盖为 100。

`com.redhat.microservice.MyService/Retry/maxRetries=100`

有时，需要为整个微服务(即部署中所有出现的注释)配置相同的参数值。

在这种情况下，配置属性 **<注释> / <参数>** 会覆盖指定注释的相应参数值。例如，为了将所有`@Retry`的所有`maxRetries`覆盖为 30，请指定以下配置属性:

`Retry/maxRetries=30`

## 结论

这篇文章只是对规范的一个概述。要了解它的一切，你可以在这里找到 MicroProfile 容错 1.0 规范文档[。了解该规范如何工作的另一种方法是检查它的](https://developers.redhat.com/blog/wp-content/uploads/2017/11/microprofile-fault-tolerance-spec.pdf) [TCK](https://github.com/eclipse/microprofile-fault-tolerance/tree/master/tck) (技术兼容性工具包)。

该规范还很年轻，但它已经有了 5 个实现。如果你想参与到这个规范中，你可以从完成一个关于 MicroProfile 容错报告的[问题](https://github.com/eclipse/microprofile-fault-tolerance/issues)开始，或者只是开始着手其中的一个。

您还可以帮助我们增强 WildFly Swarm repo 中的[容错实现](https://github.com/wildfly-swarm/wildfly-swarm/tree/master/fractions/microprofile/microprofile-fault-tolerance)(或任何其他微配置文件实现)，或者报告我们[吉拉服务器上的一个 bug。](https://issues.jboss.org/issues/?jql=project%20%3D%20SWARM%20AND%20status%20in%20(Open%2C%20Reopened))

*Last updated: October 17, 2018*