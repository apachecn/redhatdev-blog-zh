# 开源分布式跟踪环境指南

> 原文：<https://developers.redhat.com/blog/2019/05/01/a-guide-to-the-open-source-distributed-tracing-landscape>

开始使用分布式跟踪可能是一项艰巨的任务。有许多新的术语、框架和工具具有明显重叠的功能，很容易迷失或偏离方向。本指南将通过描述和分类最流行的工具来帮助您浏览开源分布式跟踪的前景。

尽管跟踪和分析是密切相关的学科，但分布式跟踪通常被理解为一种技术，用于将有关不同工作单元(通常在不同的进程或主机中执行)的信息联系在一起，以便了解整个事件链。在现代应用程序中，这意味着分布式跟踪可以用来讲述 HTTP 请求的故事，因为它穿越了无数的微服务。

这里列出的大多数工具都可以归类为工具库、跟踪器、分析工具(后端+ UI)或者它们的任意组合。文章[“跟踪、追踪和追踪的区别”](https://medium.com/opentracing/the-difference-between-tracing-tracing-and-tracing-84b49b2d54ea)是描述分布式追踪的这三个方面的一个很好的资源。

出于本指南的目的，我们将把*检测*定义为用来告诉记录什么的库，*跟踪器*定义为知道如何记录和提交这些数据的库，*分析工具*定义为接收跟踪信息的后端。在现实世界中，这些类别是可变的，仪器和示踪剂之间的区别并不总是很清楚。类似地，术语“分析工具”可能过于宽泛，因为一些工具专注于探索踪迹，而其他工具则是完整的可观察性平台。

本指南仅列出了开源项目，但还有其他几个值得一试的供应商和解决方案，如 AWS X-Ray、Datadog、Google Stackdriver、Instana、LightStep 等。

### 阿帕奇空中漫步

| 使用仪器 | 追踪者 | 分析工具 |
| 一千 | 一千 | -好的 |

[Apache SkyWalking](https://skywalking.apache.org/) 最初于 2015 年开发，作为一个了解分布式系统的培训项目。此后，它在中国流行起来，目标是成为一个完整的应用程序性能监控平台(APM ),重点关注通过代理的自动检测以及与现有跟踪器(如 Zipkin 和 Jaeger)的集成，或者与基础设施组件(如服务网格)的集成。最近，[将](https://blogs.apache.org/foundation/entry/the-apache-software-foundation-announces50)提升为阿帕奇基金会的顶级项目。

### 阿帕奇(孵化中)齐普金

| 使用仪器 | 追踪者 | 分析工具 |
| -好的 | -好的 | -好的 |

Apache(孵化中)Zipkin 最初由 Twitter 开发，[于 2012 年开源](https://blog.twitter.com/engineering/en_us/a/2012/distributed-systems-tracing-with-zipkin.html)。它是最成熟的开源追踪系统之一，并且启发了几乎所有的现代分布式追踪工具。这是一个完整的跟踪解决方案，包括工具库、跟踪器和分析工具。称为 [B3](https://github.com/apache/incubator-zipkin-b3-propagation) 的传播格式是分布式跟踪中的通用语言，其[数据格式](https://github.com/apache/incubator-zipkin-api/blob/master/zipkin2-api.yaml)也是如此，其他工具本身支持这种格式，例如生产端的[特使代理](https://www.envoyproxy.io/)和消费端的其他跟踪解决方案。Zipkin 的优势之一是高质量框架检测库的数量。

### 干草堆

| 使用仪器 | 追踪者 | 分析工具 |
| 一千 | -好的 | -好的 |

Haystack 是一个跟踪系统，具有类似 APM 的功能，例如异常检测和趋势可视化。该架构最初由 Expedia 开发，明确关注高可用性。Haystack 利用 OpenTracing 作为其主要的工具库，像 [Pitchfork](https://github.com/HotelsDotCom/pitchfork) 这样的附加组件可以用于接收其他格式的数据。

### 贼鸥

| 使用仪器 | 追踪者 | 分析工具 |
| 一千 | -好的 | -好的 |

[Jaeger](https://www.jaegertracing.io/) 最初在[优步开发，2017 年](https://eng.uber.com/distributed-tracing/)开源，不久后转移到[云原生计算基金会(CNCF)](https://www.cncf.io/) 。Dapper 和 Zipkin 的灵感可以在 Jaeger 的原始架构、数据模型和命名法中看到，但它已经超越了这些。对于仪器部分，Jaeger 利用了 OpenTracing API，它从一开始就是一等公民。该分析工具非常轻量级，非常适合开发目的和高弹性环境(例如，多租户 Kubernetes 集群)，并且是像 [Istio](https://istio.io/) 这样的工具的默认跟踪器。

### OpenCensus

| 使用仪器 | 追踪者 | 分析工具 |
| -好的 | -好的 | 一千 |

OpenCensus 最初是由 Google 基于其内部追踪平台开发的，它既是一个追踪器，也是一个工具库。它的 tracer 可以连接到“出口商”，将数据发送到 Jaeger、Zipkin 和 Haystack 等开源分析工具，以及该地区的供应商，如 Instana 和 Google Stackdriver。除了 tracer 之外，OpenCensus 代理也可以用作进程外导出器，允许被检测的应用程序完全独立于数据最终到达的分析工具。追踪是 OpenCensus 的一个方面，而指标则是整个画面的补充。它在框架插装库方面还不够丰富，但是一旦与 OpenTracing 项目的合并完成，这种情况可能会改变。

### OpenTracing

| 使用仪器 | 追踪者 | 分析工具 |
| -好的 | 一千 | 一千 |

如果在分布式跟踪的工具方面有接近标准的东西，那就是 [OpenTracing](https://opentracing.io/) 。这个在 CNCF[举办的项目是由在各种场景下实现分布式跟踪系统的人发起的:作为供应商、用户或内部实现的开发者。在项目的一方面，有许多框架工具库，例如 JAX-RS、Spring Boot 或 JDBC 的工具库。另一方面，几个 tracers 完全支持 OpenTracing API，包括 Jaeger 和 Haystack，以及该领域的知名供应商，如 Instana，LightStep，Datadog 和 New Relic。Zipkin 也有兼容的实现。](https://www.cncf.io/)

### 公开追踪+公开普查

| 使用仪器 | 追踪者 | 分析工具 |
| 是吗？-你好 | 是吗？-你好 | 一千 |

最近宣布，OpenTracing 将与 OpenCensus 合并。虽然还不清楚未来的工具会是什么样子，甚至不知道它将如何命名，但这肯定是值得关注的事情。一个试探性的[路线图](https://medium.com/opentracing/a-roadmap-to-convergence-b074e5815289)已经发布，同时还有一些代码方面的具体建议，展示了这个新工具将遵循的方向。

### 精确的

| 使用仪器 | 追踪者 | 分析工具 |
| -好的 | -好的 | -好的 |

Pinpoint 最初于 2012 年在 [Naver](https://www.navercorp.com/en) 开发，2015 年开源。它包含 APM 功能，以网络拓扑、JVM 遥测图和跟踪视图为特色。插装只能通过代理来完成，并且可以通过插件来扩展。这种方法的好处是插装不需要修改代码；但缺点是，它缺乏对显式工具的支持。Pinpoint 与基于 PHP 和 JVM 的应用程序一起工作，在这些应用程序中，它对框架和库有广泛的支持。

### 耶稣基督

| 使用仪器 | 追踪者 | 分析工具 |
| -好的 | -好的 | 一千 |

这个项目是由 Stripe 发起的，它被描述为可观测性数据的管道。它偏离了本指南中的几乎所有其他工具，因为它非常固执地认为可观测性应该是什么:跨度。它带有一组本地代理(称为“接收器”)，能够接收 spans，从中提取或聚合数据，并将结果发送到 Kafka 等外部系统。为了更好地实现这一点，Veneur 自带数据格式， [SSF](https://github.com/stripe/veneur/tree/master/ssf) 。指标可以嵌入到跨度中，也可以基于“常规”跨度数据进行合成/汇总。

### 衣冠楚楚的

Dapper 分布式追踪解决方案起源于 Google，在 2010 年的一篇论文中有所描述。它是这里列出的大多数工具的共同祖先，包括 Zipkin、Jaeger、Haystack、OpenTracing 和 OpenCensus。虽然 Dapper 并不是一个可以下载和安装的解决方案，但这篇文章仍然是现代分布式跟踪解决方案中使用的原语的很好参考，也是一些设计决策背后的推理。

### W3C 跟踪上下文

当前分布式跟踪生态系统的一个大问题是使用不同跟踪器的应用程序之间的互操作性。为了解决这个问题，[分布式跟踪工作组](https://www.w3.org/2018/distributed-tracing/)在万维网联盟(W3C)成立，致力于传播格式的[跟踪上下文](https://www.w3.org/TR/trace-context/)建议。

### 所有项目概述

| 项目 | 使用仪器 | 追踪者 | 分析工具 |
| 阿帕奇空中漫步 | 一千 | 一千 | -好的 |
| 阿帕奇(孵化中)齐普金 | -好的 | -好的 | -好的 |
| 干草堆 | 一千 | -好的 | -好的 |
| 贼鸥 | 一千 | -好的 | -好的 |
| OpenCensus | -好的 | -好的 | 一千 |
| OpenTracing | -好的 | -好的 | 一千 |
| 公开追踪+公开普查 | 是吗？-你好 | 是吗？-你好 | 一千 |
| 精确的 | -好的 | -好的 | -好的 |
| 耶稣基督 | -好的 | -好的 | 一千 |

*Jura ci paix o krhling 将介绍[“我的微服务在做什么？”](https://summit.redhat.com/conference/sessions)Red Hat Summit，5 月 9 日，星期四，上午 11:00-11:45。本次演讲将探讨微服务架构带来的一些挑战，包括可观察性问题，在这种情况下，很难知道存在哪些服务，它们如何相互关联，以及每个服务的重要性。*

如果您还没有注册，请访问[红帽峰会](https://www.redhat.com/en/summit/2019/?intcmp=701f20000012i8UAAQ)进行注册。波士顿见！

*Last updated: January 14, 2022*