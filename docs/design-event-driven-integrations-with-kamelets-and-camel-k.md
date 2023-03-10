# 使用 Kamelets 和 Camel K 设计事件驱动的集成

> 原文：<https://developers.redhat.com/blog/2021/04/02/design-event-driven-integrations-with-kamelets-and-camel-k>

[Kamelet](https://camel.apache.org/components/latest/kamelet-component.html) 是由 [Camel K](/topics/camel-k) 在 2020 年底推出的概念，旨在简化复杂系统集成的设计。如果你熟悉 [Camel](https://camel.apache.org/) 生态系统，你就会知道 Camel 为集成现有系统提供了许多[组件](https://camel.apache.org/components/latest/)。然而，集成的粒度与其底层组件相关。与单独使用 Kamelets 相比，使用 Kamelets 可以在更高的抽象层次上进行推理。

A *Kamelet* 是指定集成流程的文档。Kamelet 使用 Camel 组件和企业集成模式(EIP)来描述系统的行为。您可以在基于 [Kubernetes](/topics/kubernetes) 集群的任何集成中重用 Kamelet 抽象。您可以使用任何 Camel 的领域特定语言(DSL)来编写 Kamelet，但是最自然的选择是 YAML DSL。YAML DSL 旨在指定输入和输出格式，因此任何集成开发人员都预先知道预期的数据类型。

一个 Kamelet 也可以作为事件的源或汇，使 kame let 成为一个[事件驱动架构](/topics/event-driven)的构建模块。在本文的后面，我将介绍源和接收器事件的概念，以及如何通过 Kamelets 在事件驱动的架构中使用它们。

## 捕捉系统复杂行为

在前 Kamelet 时代，您通常会通过推理底层组件来设计企业集成。集成将由队列、API 和数据库组成，它们粘合在一起以产生或消费事件。理想情况下，您将使用 Camel EIPs 和组件来开发集成，并使用 Camel K 在 Kubernetes 上运行它。

随着 Kamelets 的出现，您可以将设计阶段视为一个*声明阶段*，在这里您可以在 YAML 文档中描述流程。你只需要描述你的系统规格。大多数情况下，您将定义在运行时提供的变量，就在运行集成之前。最后，您将使用可视化工具来帮助实现建议和自动完成等功能。一旦 Kamelet 准备就绪，您就可以在 Kubernetes 集群上安装它的定义，并在真正的集成中使用它。

在这个阶段，你仍然没有执行任何事情。要查看 Kamelet 的运行情况，必须在集成中使用它，可能需要为预期变量指定具体的值。我觉得这是 Kamelet 最显著的特点。它将集成开发分为两个阶段:设计(抽象)和执行(具体)。

使用这种方法，您可以为您的集成开发人员提供一组预构建的高级抽象，用于他们的具体集成。例如，您可以创建一个 Kamelet，其作用域是公开某个企业域的事件。您可以将每个 Kamelet 看作是具有某种“业务”复杂性的任何东西的封装。您可以在任何改进的集成中轻松使用这些构建块。

基本上，我们可以将 Kamelet 视为一个复杂系统的精华视图，它展示了一组有限的信息。集成在其执行流程中使用这些信息。

## 骆驼组合中的 Kamelets

在真正的集成中使用 Kamelets 非常简单。你可以把 Kamelets 看作是一个新的 Camel 组件。每个 Kamelet 可以被设计成信源(Camel DSL 中从开始的*)或信宿(Camel DSL 中从*到*)。(我们将很快更多地讨论将 Kamelets 设计为源和汇。)*

一个 Camel K 操作员可以发现在幕后处理所有魔术的`kamelet`组件。因此，只要您运行集成，Kamelet 就会立即开始工作。

### kamenets 在行动

一个演示通常胜过千言万语，所以我们来看一个真实的例子。首先，我们将定义一个 Kamelet。对于这个例子，我们要品尝一种新鲜啤酒:

```
apiVersion: camel.apache.org/v1alpha1
kind: Kamelet
metadata:
  name: beer-source
  labels:
    camel.apache.org/kamelet.type: "source"
spec:
  definition:
    title: "Beer Source"
    description: "Retrieve a random beer from catalog"
    properties:
      period:
        title: Period
        description: The interval between two events
        type: integer
        default: 1000
  types:
    out:
      mediaType: application/json
  flow:
    from:
      uri: timer:tick
      parameters:
        period: "#property:period"
      steps:
      - to: "https://random-data-api.com/api/beer/random_beer"
      - to: "kamelet:sink"

```

根据您想要设计的抽象的复杂性,`flow`部分可能会更加详细。我们姑且称这个文件为`beer-source.kamelet.yaml`。我们现在可以在我们的 Kubernetes 集群上创建这个 Kamelet:

```
$ kubectl apply -f beer-source.kamelet.yaml

```

在本例中，我们使用了 Kubernetes 命令行界面(CLI ),因为我们只是声明了 Kamelet 的类型。使用[红帽 OpenShift](/products/openshift/overview) 的`oc`也可以。要查看集群上可用 Kamelets 的列表，让我们运行:

```
$ kubectl get kamelets

NAME           PHASE
beer-source    Ready

```

我们将在实际集成中使用它，用下面的内容替换前面的`flow`部分:

```
- from:
    uri: "kamelet:beer-source?period=5000"
    steps:
      - log: "${body}"

```

现在，我们有了一个名为`kamelet`的新组件。我们正在从`beer-source`创建一个新的集成，这是我们提供的 Kamelet 名称。我们还设置了一个变量`period`，我们之前已经在 Kamelet 中声明过了。

### 运行 kamelot

骆驼 K 是跑 Kamelet 最简单的方法。如果你还没有安装 Camel K，按照[的指示安装它](https://camel.apache.org/camel-k/latest/installation/installation.html)。准备好之后，将集成包在一个文件中(比如，`beers.yaml`)并运行:

```
$ kamel run beers.yaml --dev

```

一旦你运行集成，你将开始享受每五秒钟一次的新鲜啤酒(就在屏幕上)。

## 附加集成特性

既然你已经看了演示，让我们回到理论上来。Kamelets 支持类型规范和可重用性，这对于复杂的集成来说都是有用的特性。

### 类型规范

使用 Kamelets 的一个重要方面是*类型规范*。在 Kamelet 中，您可以声明 Kamelet 将作为输入使用和作为输出产生的数据类型。如果集成开发人员不想处理 Kamelet 隐藏的复杂逻辑，这个规范非常有用。类型信息以 JSON 模式的形式提供，因此开发人员可以很容易地阅读它并理解它所公开的信息。

让我们设想一个 Kamelet，它被设计用来从产品目录中提取价格。价格是通过在由各种端点和数据库组成的系统中过滤、拆分和转换数据来计算的。在一天结束时，开发人员想知道预期的数据格式。有了名称、描述和数据类型，开发人员就对 Kamelet 有了一个大致的了解。

### 复用性

我提到过，Kamelet 安装在 Kubernetes 集群上，可以在任何 Camel K 集成中使用。安装过程是 Kubernetes 通过定制资源端点的扩展。

这是让 Kamelets 成为集群的一等公民的好方法，让它们被其他应用程序发现和使用。拥有各种各样的 Kamelets，组织中的其他人可以重用，这意味着如果已经有了一些东西，您就不需要重新发明轮子。

将变量定义为可以改变的数据的抽象占位符的可能性是可重用性的关键。如果您能够定义一个通用的流程，用参数描述一个集成或者一个集成的一部分，那么您将拥有一个 Kamelet 以备可能的重用。

## 在事件驱动架构中使用 Kamelets

因为 Kamelets 是定义输入和输出的高级概念，所以可以把它们看作事件驱动架构的构建块。前面我提到过，您可以创建一个 Kamelet 作为源或汇。在事件驱动的架构中，*源*发出事件，接收器可以使用这些事件。因此，你可以把一个系统想象成*产生*事件(一个源)或者*消耗*事件(一个汇)。这个想法是提炼一个更复杂系统的观点。

### 在 Kamelet 绑定中声明高级流

Camel K 工程师将这些概念整合在一起，包括一个负责将事件路由到相关方的中间层。这是在一个 *Kamelet 绑定*中以声明方式完成的。

Kamelet 绑定允许您告诉 Kamelet 源将事件发送到目的地，目的地可以由一个 [Knative](/topics/serverless-architecture) 通道、一个 [Kafka](/topics/kafka-kubernetes) 主题或一个通用的 URI 来表示。同样的机制让 Kamelet 接收器使用来自目的地的事件。因此，您可以使用 Kamelet 绑定来定义事件驱动架构的高层流程，如图 1 所示。最终用户完全看不到路由逻辑。

[![Kamelets Event Driven Architecture](img/e26a5b342904efdd9dee8eb3b131a793.png "Kamelets Event Driven Architecture")](/sites/default/files/blog/2021/01/eda_jabberwocky.png)Kamelets Event Driven Architecture

Figure 1: Kamelet's event-driven architecture.

集成开发人员可以通过将这些事件绑定到 Kamelet 接收器来创建基于相关事件的工作流。想象一个网上商店。订单系统发出可以被运输系统、库存系统或用户偏好系统捕获的事件。三个接收器独立工作，依靠这个框架。他们每个人都关注事件并对事件做出反应。

订单系统可以用*订单源 Kamelet* 来表示，这隐藏了其内部的复杂性。然后，创建一个 Kamelet 绑定，开始将事件发送到 Knative、Kafka 或 URI 通道。在生产者方面，开发人员可以设计一个解耦的*运输接收器 Kamelet* ，并通过 Kamelet 绑定将其绑定到*订单事件通道*。因此，发送接收器将开始接收事件并执行编码到其中的业务逻辑。

### kamelot 绑定正在运行

我们用之前的 Kamelet 把事件(随机啤酒)发射到一个卡夫卡的话题里。为了让这个例子工作，您可能需要一个正在运行的 Kafka 集群。如果你没有 Kafka 集群，你可以用 [Strimzi](https://strimzi.io/) 设置一个。准备就绪后，创建一个 Kafka 主题作为您的目标频道:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: beer-events
  labels:
    strimzi.io/cluster: my-cluster
spec:
  partitions: 1
  replicas: 1
  config:
    retention.ms: 7200000
    segment.bytes: 1073741824

```

我们把这个文件叫做`beer-destination.yaml`。现在，在集群上创建 Kubernetes 主题:

```
$ kubectl apply -f kafka-topic.yaml

```

一旦准备好，就声明一个`KameletBinding`,它的目标是将生成器生成的任何事件转发到目的地:

```
apiVersion: camel.apache.org/v1alpha1
kind: KameletBinding
metadata:
  name: beer-event-source
spec:
  source:
    ref:
      kind: Kamelet
      apiVersion: camel.apache.org/v1alpha1
      name: beer-source
    properties:
      period: 5000
  sink:
    ref:
      kind: KafkaTopic
      apiVersion: kafka.strimzi.io/v1beta1
      name: beer-events

```

调用这个文件`beer-source.binding.yaml`并使用它在集群上创建资源:

```
$ kubectl apply -f beer-source.binding.yaml

```

### 创建汇流

让我们暂停一下。这份文件的结构很简单。我们声明一个源和一个接收器。换句话说，我们告诉系统我们希望某些事件在哪里结束。到目前为止，我们有一个源，以及一个收集其事件并将其转发到目的地的机制。为了继续，我们需要一个消费者(*又名*，一个接收器)。

让我们创建一个`log-sink`。它的目标是消费任何通用事件并在屏幕上打印出来:

```
apiVersion: camel.apache.org/v1alpha1
kind: Kamelet
metadata:
  name: log-sink
  labels:
    camel.apache.org/kamelet.type: "sink"
spec:
  definition:
    title: "Log Sink"
    description: "Consume events from a channel"
  flow:
    from:
      uri: kamelet:source
      steps:
      - to: "log:sink"

```

这是一个简单的记录器，但是你可以根据你的用例需要设计一个复杂的接收器流。让我们称之为`log-sink.kamelet.yaml`并在我们的集群上创建记录器:

```
$ kubectl apply -f log-sink.kamelet.yaml

```

### 创建 kamelot 绑定

现在，我们创建一个`KameletBinding`来将事件与我们刚刚创建的记录器相匹配:

```
apiVersion: camel.apache.org/v1alpha1
kind: KameletBinding
metadata:
  name: log-event-sink
spec:
  source:
    ref:
      kind: KafkaTopic
      apiVersion: kafka.strimzi.io/v1beta1
      name: beer-events
  sink:
    ref:
      kind: Kamelet
      apiVersion: camel.apache.org/v1alpha1
      name: log-sink

```

注意，我们刚刚将以前的绑定接收器(`beer-event`)交换为源，并将接收器声明为新的 log Kamelet。让我们调用文件`log-sink.binding.yaml`并在我们的集群上创建它:

```
$ kubectl apply -f log-sink.binding.yaml

```

### 检查运行中的积分

在幕后，Camel K [操作符](/topics/kubernetes/operators)负责将这些 Kamelet 绑定转换成运行中的集成。我们可以看到，通过执行:

```
$ kamel get

NAME           PHASE    KIT
beer-event-source    Running    kit-c040jsfrtu6patascmu0
log-event-sink       Running    kit-c040r3frtu6patascmv0

```

所以，让我们最后看一下日志，看看水槽的运行情况:

```
$ kamel logs log-event-sink
…
{"id":9103,"uid":"19361426-23dd-4051-95ba-753d7f0ebe7b","brand":"Becks","name":"Alpha King Pale Ale","style":"German Wheat And Rye Beer","hop":"Mt. Rainier","yeast":"1187 - Ringwood Ale","malts":"Vienna","ibu":"39 IBU","alcohol":"3.8%","blg":"11.1°Blg"}]
{"id":313,"uid":"0c64734b-2903-4a70-b0a1-bac5e10fea75","brand":"Rolling Rock","name":"Chimay Grande Réserve","style":"Dark Lager","hop":"Northern Brewer","yeast":"2565 - Kölsch","malts":"Chocolate malt","ibu":"86 IBU","alcohol":"4.8%","blg":"19.1°Blg"}]

```

## 结论

多亏了 Kamelet 和 kame let 绑定机制，我们成功地创建了一个简单的事件驱动架构；与生产者分离的消费者。最棒的是，我们做任何事情都只需声明事件的起源(`beer-source`)和目的地(`log-sink`)。

*Last updated: March 31, 2021*