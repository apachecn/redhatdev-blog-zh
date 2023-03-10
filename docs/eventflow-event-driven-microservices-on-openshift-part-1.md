# Event flow:open shift 上事件驱动的微服务(第 1 部分)

> 原文：<https://developers.redhat.com/blog/2018/10/15/eventflow-event-driven-microservices-on-openshift-part-1>

这篇文章是三篇相关文章中的第一篇，描述了我们创建的名为 EventFlow 的轻量级云原生分布式[微服务](https://developers.redhat.com/topics/microservices/)框架。EventFlow 可以用来开发能够处理[云事件](https://cloudevents.io)的流媒体应用，这是一种基于数据格式的标准化努力，用于交换云平台生成的事件信息。

EventFlow 平台是专门针对[Kubernetes](https://developers.redhat.com/topics/kubernetes/)/[open shift](http://openshift.com/)平台而创建的，它将事件处理应用程序建模为一个连接的流或组件流。可以通过使用简单的 SDK 库来促进这些组件的开发，或者可以将它们创建为 Docker 映像，可以使用环境变量对其进行配置，以附加到 Kafka 主题并直接处理事件数据。

## 背景

事件处理是一种方法，用于对潜在的实时事件和数据流进行推理，并根据它们的绝对值和时间特性生成结论。事件流可以来自许多来源；它们可以由组织的某些部分生成(例如，订单模式、销售电话。等等。)，它们可以从外部来源(例如，新闻条目的出现、股票价格等)聚集。)，从传感器收集(物联网应用程序有可能生成大量的流数据)，甚至由云托管平台内发生的变化发出。

尽管能够对信息流做出实时反应很有吸引力，但创建和部署事件驱动系统的实际过程是复杂的。除了管理事件处理的实际业务逻辑之外，在部署此逻辑以生成一个正常运行的系统时，还需要考虑其他事项:

*   代码必须以适合在容器环境中部署的形式构建和打包。这主要是通过使用 Maven 等工具和相关插件(通常是 [fabric8 插件](https://maven.fabric8.io/))来处理的。然而，安装和连接底层消息中间件的责任仍然是开发人员的责任。
*   修改和扩展应用程序的过程需要小心管理。添加额外的计算资源来缓解瓶颈需要将新的资源投入运行并集成到正在运行的系统中，而不干扰现有的操作。同样，移除多余的资源需要在不影响应用程序整体语义的情况下完成。如果附加资源位于远程云平台中(例如，云爆发操作)，这一点尤为重要。

考虑到与有效处理事件数据相关联的好处和挑战，大量平台被创建就不足为奇了。

重要的是要认识到，我们并没有试图重新创建像 Apache Kafka Streams API 这样的库所提供的功能。相反，我们创建了一个框架，用于将流组件(其中一些组件本身可能包含 Apache Kafka Streams API 代码)连接到一个连贯的数据流中，该数据流可以在容器平台中部署和管理。

此外，这篇文章中描述的框架是专门为在容器平台内大规模运行而设计的，并且是使用完全云原生的方法编写的。这种方法有许多明显的优点:

*   容器平台非常适合处理不同级别的负载，因为这种架构使得响应不同级别的需求而扩展或缩减应用程序的各个部分变得非常容易。
*   通过采用云原生优先的方法，我们可以利用容器平台已经提供的各种管理、数据表示和监控工具。
*   像 OpenShift 这样的平台在实现语言方面所提供的固有灵活性意味着应用程序可以使用各种不同的工具包和语言来构建，并且仍然可以在相同的事件流上运行。

在本文中，我们将重点关注处理 [CloudEvents](https://cloudevents.io/) ，它提供了一种以独立于平台的方式描述事件数据的标准机制。CloudEvents 规范旨在使编写生产和消费基于事件的服务的可移植应用程序变得更加容易。该规范提供了一个基本级别的元数据和有效载荷，元数据可能是事件感兴趣的内容，有效载荷可以是任意结构。CloudEvents 不同于任何特定序列化格式的绑定，但是事件流平台使用它们的 [JSON 表示](https://github.com/project-streamzi/jcloudevents)。尽管我们在这篇文章中关注的是 CloudEvents，但 EventFlow 平台并不特定于它们。它可以用于在处理器之间传输任何其他类型的“可序列化”数据。

## EventFlow 架构

出于 EventFlow 平台的目的，应用程序被建模为一组相连的处理器(P [1] ，P [2，]和 P [3] )。事件数据沿着逻辑连接在这些处理器之间流动(C [1] 和 C [2] ): [![Diagram showing how event data flows between processors along logical connections](img/0f63d9d2acf12b3ade47afb54ebb7c7f.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/Blog-Post_-CloudEvent-Flow-1.png)

为了参与这样的流程，开发人员编写的任何代码都需要符合以下三个类别之一:

因为流中的组件被部署为 OpenShift pods，所以我们可以通过增加或减少每个组件的部署 pod 的数量来支持不同级别的消息吞吐量。例如，一个包含计算开销很大的步骤的流(下图中的 P [2] )可以部署额外的瓶颈处理器副本来处理吞吐量。(EventFlow 支持在本地云环境和/或远程云环境中部署的副本):

[![Diagram of extra replicas of a bottleneck processor deployed to cope with the throughput](img/d8d29c0da7f7201ffcfc2358a0e3801b.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/Blog-Post_-CloudEvent-Flow_Replicas.png)

为了将处理器连接在一起并管理副本之间的事件分布，我们使用部署在容器平台内的 Apache Kafka(由 [Red Hat AMQ 流](https://developers.redhat.com/products/amq/overview/)产品提供)。对于第一个图中显示的例子，两个处理器间连接(C [1] 和 C [2] )使用两个独立的 Kafka 主题来表示。

当在容器平台中部署流时，流中的组件表示如下:

这些概念如下所示:

[![Diagram of the components within a flow](img/c6e618fad9365d9b2c07b4a23f5e46e7.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/Blog-Post_-CloudEvent-Flow_Full.png)

## 开发、设计、部署和管理流程

鉴于上面给出的高级概述，利用事件流平台需要开发人员考虑一系列任务，从开发事件处理代码到管理部署的副本和到远程云的连接。

### 开发:用于开发处理器的 Maven 原型

Maven 原型可以用来为开发人员创建事件流可以包含的处理器提供脚手架。event-flow SDK 提供了一些接口，开发人员只需很少的努力就可以使用这些接口来生成和消费 CloudEvents。例如，下面的代码将记录它接收到的事件。事件流运行时负责用输入/输出连接细节和开发人员需要的任何设置来配置处理器。

```
@CloudEventComponent
public class EchoingProcessor {

   Logger logger = Logger.getLogger(DataLogger.class.getName());

   @CloudEventProducer(name = "OUTPUT_DATA")
   CloudEventProducerTarget target;

   @CloudEventConsumer(name = "INPUT_DATA")
   public void onCloudEvent(CloudEvent evt){

      if(evt.getData().isPresent()){
         logger.info(evt.getData().get().toString());
      }
      target.send(evt);
   }
}
```

定制处理器的开发将在本系列的第 3 部分中深入讨论。第 3 部分还将展示如何用其他语言开发处理器。

### 设计:事件流管理器 API 和 UI

流在内部表示为 k8s 定制资源。虽然这些可以通过标准的 k8s API 创建，并且我们提供了使用 CRD 的 Java 类，但是这对大多数开发人员来说不是很方便。为了让开发人员更容易地创建和部署流，我们开发了一个基于 web 的初始 UI，用于图形化地创建流。

使用该工具，开发人员可以选择红帽 AMQ 流中已经存在的输入主题，并将它们连接到已经部署到 OpenShift 中的处理器。管理器 UI 允许开发人员设置处理器参数和非功能性设置，例如每个处理器的副本数量。很可能在将来，当前的原型用户界面将被其他更加一致和用户友好的工具所取代。

### 部署:流运算符

流操作符部署在 OpenShift 中，负责流的部署和配置。当新的流定制资源被部署到平台中时或者当现有资源被更新时，操作员被通知。操作员检查流 CR(自定义资源)并生成一组可以部署到 OpenShift 中的资源(部署、配置映射和 KafkaTopics)。在新的流部署中，这些资源是在平台中创建的，当流到达输入主题时，它将开始处理数据。在重新配置现有流程的情况下，操作员将只更新已经改变的组件，而保持未改变的组件“原样”这种改变可能是增加一个处理器的副本的数量，或者编辑流定义以包括额外的处理器。

### 连接:利用红帽 AMQ 流

EventFlow 平台使用红帽 AMQ 流作为微服务之间的通信机制。这允许平台根据需要动态创建主题，并且具有一些关于部署和重新配置的便利属性。

流处理器的部署可以按照任何顺序进行，这取决于诸如是否有任何图像被本地缓存以及每个容器的启动开销之类的因素。因为红帽 AMQ 流将在处理器之间缓冲消息，我们能够以任何顺序实例化流，并且确信一旦上游组件启动，消息将开始流动。

第二，构成红帽 AMQ 流基础的阿帕奇卡夫卡的特性之一是，它有无限期存储历史信息的潜力。这个特性意味着，如果一个新的处理器被添加到一个正在运行的流中，就有可能通过它和所有下游处理器“重放”旧消息。这导致重新配置的流的行为就像它最初被部署一样。

## 额外资源

以下是一些可能有帮助的卡夫卡文章:

*   [在 OpenShift 上使用 Apache Kafka 进行智能电表数据处理](https://developers.redhat.com/blog/2018/07/16/smart-meter-streams-kafka-openshift/)
*   [介绍 Kafka-CDI 库](https://developers.redhat.com/blog/2018/05/31/introducing-the-kafka-cdi-library/)
*   [宣布 AMQ 流:OpenShift 上的阿帕奇卡夫卡](https://developers.redhat.com/blog/2018/05/07/announcing-amq-streams-apache-kafka-on-openshift/)

## 结论

这篇文章展示了一个框架，它能够使用一组组件来处理事件流，这些组件通过安装由 Red Hat AMQ 流产品提供的 Apache Kafka 来连接。这种事件处理方法允许开发人员创建简单的组件，这些组件可以使用图形编辑器或通过流定义文档连接在一起。这意味着可以相对容易地创建复杂的事件处理管道，然后根据不同的需求水平进行扩展，或者根据需要分布在多个云提供商上。

本系列的后续文章将描述事件流的创建和部署过程，还将介绍软件开发环境，该环境使开发人员能够创建自己的处理组件，并使用该框架将它们链接在一起。

*Last updated: September 3, 2019*