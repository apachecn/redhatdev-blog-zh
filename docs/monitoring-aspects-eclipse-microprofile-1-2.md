# Eclipse MicroProfile 1.2 的监控方面

> 原文：<https://developers.redhat.com/blog/2017/10/17/monitoring-aspects-eclipse-microprofile-1-2>

Eclipse MicroProfile (MP)旨在通过开发符合 MP 的供应商随后实施的通用标准，将微服务引入企业 Java[1]。这不仅适用于开发人员 API，也适用于运行、配置和管理服务器的接口。

更经典的规范常常遗漏了许多特定于供应商的细节——尤其是在应用程序和服务器的设置和运行方面。对于 Java 企业版，有像 JMX 和 JSR-77 这样的标准，但是这些标准大部分时间没有被使用，或者没有指定从管理站的访问。这使得在实践中监控应用程序的健康或从一个应用服务器供应商移植应用程序变得更加困难。

MicroProfile 社区已经决定，运行遥测和健康检查等应用程序的各个方面不应该是特定于供应商的，而应该是基本规范的一部分。

我现在将展示 MicroProfile 1.2 版本中包含的两个方面的监控。更多方面，如分布式跟踪，可能会在后续版本中出现。

## 健康检查

健康检查回答了二进制问题“我的应用程序运行良好还是应该重启？”对于操作来说，这一直是一个重要的问题，重启一个有内存泄漏的应用程序是很常见的。最近几天，健康检查变得与 Kubernetes 这样的调度程序更加相关，健康检查是这些调度程序的核心概念。Kubernetes 和 OpenShift 会定期检查正在运行的容器的健康状况。如果容器报告不健康(或者根本没有响应)，Kubernetes 将终止容器并启动一个新的实例。

### 通过 Java API 描述健康状态

健康检查有两个访问点，http 端和 Java API 端。让我们先来看看 Java 端，以揭示系统健康状况:

```
@Health
@ApplicationScoped
public class HealthDemo implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        HealthCheckResponseBuilder alive = HealthCheckResponse.named("alive");
        // add other info
        return alive.up().build();
    }
}
```

为了公开数据，您必须实现`HealthCheck`接口。在已实现的`call()`方法中，检索带有检查名称的`HealthCheckResponseBuilder`，并提供状态(up/down)。也可以向`HealthCheckResponseBuilder`提供更多参数。然后在 rest 接口上公开它们。

有可能提供不止一个这样的健康检查提供者。所有这些检查的结果将汇总形成最终结果。

### 通过 http 获取健康状态

系统可以通过在`/health`端点上的 http GET 操作来查询健康数据。如果响应代码在 200-399 的范围内，Kubernetes 将认为应用程序是健康的。

因此，为了报告一个健康的系统状态，MP-Health 实现将用“200 OK”和一个有效负载来响应，该有效负载将结果指定为 UP 和执行的检查列表。除非安装了特定的运行状况检查，否则检查数组将为空。

```
$ curl http://localhost:8080/health
{
"outcome": "UP",
"checks": [
    {
        "name": "alive",
        "state": "UP"
    }
  ]
}
```

各个检查在“检查”部分提供其状态。当配置了多个检查并且总体结果为 DOWN 时，这很有帮助。个人检查有助于查明不健康的原因。Java 代码中给`HealthCheckResponseBuilder`的附加信息被传递给单独的检查结果。

## 遥测又名微轮廓度量

遥测揭示了正在运行的服务器的指标，如 CPU 和内存使用情况、线程数等。然后，这些数据通常会被输入到图表系统中，以便随时查看指标，或者用于容量规划目的。

Java 虚拟机有办法通过 MBeans 和 MBeanServer 长期公开数据。从 Java SE 6 开始，甚至有一个(基于 RMI 的)远程协议为所有虚拟机定义了如何从远程进程访问 MBean 服务器。

处理这个协议很困难，并且不适合今天基于 http 的交互。另一个棘手的问题是，许多现有的服务器在不同的名称下有不同的属性。因此，对不同类型的服务器进行监控并不容易。

MicroProfile 创建了一个监控规范，它通过一个基于 http 的 API(供监控代理访问)和一个 Java API(允许导出特定于应用程序的指标)来解决上述两点。

规范中有三个度量范围:

*   基础:这些是指标，主要是 JVM 统计数据，每个兼容的供应商都必须支持。
*   供应商:不可移植的可选供应商特定指标。
*   应用程序:来自已部署应用程序的可选指标。我将为下面的人展示 Java-API。

### 通过 http 检索遥测数据

让我们看一下监控代理从服务器检索数据的方式。

默认情况下，MicroProfile Metrics 以两种格式公开数据。如果没有明确要求格式，则使用[普罗米修斯文本格式](https://prometheus.io/docs/instrumenting/exposition_formats/#text-format-details)。该规范还定义了一个 JSON 编码，可以通过传递“应用程序/json”的媒体类型来请求它。

```
$ curl -Haccept:application/json http://localhost:8080/metrics/base
{
  "classloader.totalLoadedClass.count" : 12304,
  "cpu.systemLoadAverage" : 2.029296875,
  "thread.count" : 53,
  "classloader.currentLoadedClass.count" : 12262,
  "jvm.uptime" : 6878170,
  "gc.PS MarkSweep.count" : 3,
  "memory.committedHeap" : 1095237632,
  "thread.max.count" : 66,
  "gc.PS Scavenge.count" : 11,
  "cpu.availableProcessors" : 4,
  "thread.daemon.count" : 11,
  "classloader.totalUnloadedClass.count" : 42,
  "memory.maxHeap" : 3817865216,
  "memory.usedHeap" : 427363088,
  "gc.PS MarkSweep.time" : 322,
  "gc.PS Scavenge.time" : 244
}
```

在前面的例子中，我们只检索 JSON 格式的 *base* 范围中的指标。接下来，我们以 Prometheus 格式公开所有范围的指标。我已经对输出进行了裁剪，只显示了每个范围的一个指标。

```
$curl http://localhost:8080/metrics
# TYPE application:de_bsd_swarmdemo_rest_hello_world_endpoint_a_counter counter
application:de_bsd_swarmdemo_rest_hello_world_endpoint_a_counter{tier="integration"} 52.0
# TYPE base:classloader_total_loaded_class_count counter
base:classloader_total_loaded_class_count{tier="integration"} 12304.0
# TYPE vendor:memory_pool_metaspace_usage_max gauge
vendor:memory_pool_metaspace_usage_max_bytes{tier="integration"} 6.47796E7
```

### Java API

现在让我们来看看 Java-API。在这个例子中，我使用了一个 JAX-RS 端点。DropWizard 指标的用户可能会发现以下一些熟悉的内容。该 API 是有意模仿 DropWizard 标准的。在 CDI 的帮助下，它已经得到了增强，可以用来做繁重的工作。

```
@ApplicationScoped
@Path("/hello")
public class HelloWorldEndpoint {

    @Inject
    Counter aCounter;

    @GET
    @Produces("text/plain")
    @Counted(description = "Counting of the Hello call", absolute = true)
    @Timed(name="helloTime", description = "Timing of the Hello call", absolute = true)
    @Metered(absolute = true, name = "helloMeter")
    public Response doGet() {
        aCounter.inc();
        return Response.ok("Hello from WildFly Swarm! " 
                               + aCounter.getCount())
               .build();
    }
}
```

通过 CDI 的魔力，可以使用和公开指标。在大多数情况下，只需在您想要公开的方法或字段上提供来自包`org.eclipse.microprofile.metrics.annotation`的注释`@Counted`、`@Timed`等中的一个，实现就会为您完成其余的工作。

在这个例子中有一个计数器( *aCounter* )，它只是在`@Inject`(第 6 行)的帮助下定义的。

第 14 行中的计数器显式增加。它的值在第 16 行被检索，并包含在我们的 JAX-RS 端点的 REST 响应中。

如果您查看注释，您会发现可以按照上面的描述提供额外的元数据。强烈建议提供元数据，以便操作员更容易理解某个指标的含义。

如果您没有在批注上提供显式名称，则实现会根据批注的项计算基本名称。在`@Counted` inline 10 的情况下，得到的度量名是 *doGet* ，这是方法名。因为参数*绝对*是`true`，所以完全限定的类名不在前面。

## 贡献的

MicroProfile 规范遵循一种敏捷的*足够好的*反馈周期方法。一旦发布了一个规范，它将被包含在后续的 MicroProfile 伞形发布中。规范的新版本可能会破坏与以前版本的兼容性。

如果你对规范的未来版本感兴趣，你可以加入 [MicroProfile Google Group](https://groups.google.com/forum/#!forum/microprofile) 。

## 参考

普罗米修斯文本格式[https://Prometheus . io/docs/instrumenting/exposition _ formats/# text-format-details](https://prometheus.io/docs/instrumenting/exposition_formats/#text-format-details)

微档案健康规范库[https://github.com/eclipse/microprofile-health](https://github.com/eclipse/microprofile-health)

微文件度量规范库[https://github.com/eclipse/microprofile-metrics](https://github.com/eclipse/microprofile-metrics)

[1][https://developers . red hat . com/blog/2016/06/27/micro profile-collaboration-to-bring-micro service-to-enterprise-Java/](https://developers.redhat.com/blog/2016/06/27/microprofile-collaborating-to-bring-microservices-to-enterprise-java/)

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: October 16, 2017*