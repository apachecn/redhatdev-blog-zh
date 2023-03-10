# 为工作选择正确的异步消息传递基础设施

> 原文：<https://developers.redhat.com/blog/2020/07/31/choosing-the-right-asynchronous-messaging-infrastructure-for-the-job>

术语*异步*的意思是“不同时发生”在分布式系统和消息传递的上下文中，这个术语意味着请求处理将在任意时间点发生。与同步交互相比，异步交互有很多优势，但是它们也带来了新的挑战。在本文中，我们将关注为您的[事件驱动的](https://developers.redhat.com/topics/event-driven/)系统选择异步消息传递基础设施的具体考虑事项。

我们将从基于业务价值和所交付消息的语义类型的异步交互风格之间的细微差异开始。考虑这些差异有助于我们识别消息传递模式，我们可以用它来确定我们需要的消息传递系统的类型。

**注意**:在 Red Hat，我们热爱任何开源技术，所以我使用了三个开源消息框架——作为我的例子， [Apache Qpid](https://qpid.apache.org) 、 [Apache ActiveMQ Artemis](https://activemq.apache.org/components/artemis/) 和 [Apache Kafka](https://kafka.apache.org) 。[红帽 AMQ](https://developers.redhat.com/products/amq/overview) 是我们灵活的信息平台，包括所有三个框架，让您轻松选择适合您需求的工具。

## 不同消息类型的商业价值

并非所有的信息都是平等的。有些只在短期内有效和有价值，以后就过时了。有些直到被消费掉之前都是有价值的，不管时间过去了多少。并且一些消息对于重复消费是有效和有用的。通过考虑消息相对于时间和消耗率的有效性和价值，我们可以将服务之间的交互风格分为三类，如图 1 所示。

[![Figure 1: Understanding the business value of different message types.](img/7e0127586ba44934f3e93d91a2d8138c.png "Figure 1: Understanding the business value of different message types.")](/sites/default/files/blog/2020/07/business_value.png)Figure 1: Understanding the business value of different message types.Figure 1: Understanding the business value of different message types.">

让我们考虑一下这些类别。

### 不稳定的

易变消息传递系统中的消息是短暂的，并且消息的价值是有时间限制的:它们现在是有价值的，但很快就不再有价值了。存储很快就没用的事件是没有意义的。对于这种类型的事件，易失性消息传递系统以尽可能低的延迟获得最佳性能，因为它跳过了对磁盘的写入。在这种情况下，消息传递系统知道消费者，并将事件传播给发布时在线的所有消费者。当消费者与系统断开连接时，消息传递系统会忘记他们。这种类型的系统对于其处理大量具有低延迟交互需求的动态客户端的能力至关重要，例如物联网(IoT)设备。

### 持久耐用

这种更传统类型的消息系统知道注册的消费者，并持久地存储消息，直到每个注册的消费者都阅读了它们。这种类型的系统适用于发布事件时消费者可能断开连接的情况。系统保存消息，直到每个消费者都重新连接并消费了相关事件。一旦事件被完全消费，消息代理就会丢弃消息。目标是在服务之间提供可靠的消息传递，并提供强大的订购和交付保证。

### 可重放的

这里，消息传递系统不知道消费者或事件注册。它只是存储事件，并在给定的时间内将它们发布到流中，或者直到达到最大容量。在这种类型的系统中，现有的或新的消费者可以随时加入，连接和消费事件，甚至从头重放流。消费者可以根据需要在流中来回移动。这种类型的消息传递系统的驱动力是可伸缩性，以及重放消息的能力。

## 消息语义

除了信息的技术特征之外，区分我们使用的语言——即语义方面——和交互的意图也是至关重要的。有些信息是针对特定消费者的，需要具体的行动。一些查询系统的最新状态而不需要状态改变，一些通知世界关于在源系统中已经发生的改变。从消息传递语义的角度来看，有三种消息类型，如图 2 所示。

[![A graphic showing three message types based on semantics: Command, query, and event.](img/df8757539bc11ea4c541fb3798041e47.png "message_semantics")](/sites/default/files/blog/2020/06/message_semantics.png)message_semantics

Figure 2: Understanding message types based on semantics.

让我们考虑一下这些消息类型。

### 命令

一个*命令*是一个动作请求，通常会导致一个已知目标系统的状态改变。通常，响应表明操作已经完成，甚至可能有与响应相关联的结果。当需要响应时，命令通常通过同步协议(如 HTTP)来实现。还可以在异步消息传递系统上应用请求-响应或触发-遗忘命令风格。基于命令的异步消息要求源系统和目标系统之间以命令语义的形式进行耦合。

### 询问

一个*查询*就像一个命令，但是它是一个只读的交互，不会导致状态改变。本质上，查询需要响应。这种消息类型的同步实现很常见，但是消息系统上的异步和非阻塞实现也很常见。对于长时间运行的操作，即使是“一劳永逸”的交互也很常见，在这种情况下，响应被写到不同的位置。

### 事件

一个*事件*是一个通知，告知某些事情已经改变。系统发送事件通知，通知其他系统其域中的更改。事件不同于命令，因为事件发出系统通常不期待响应。除了异步之外，事件消息不针对特定的接收者，这使得进一步的解耦成为可能。与其他异步交互类似，事件被实现为队列中的消息，这通常被称为*流*。

**注意**:参见 Martin Fowler 的演讲“[事件驱动架构的多种含义](https://www.youtube.com/watch?v=STKCRSUsyP0)”，深入了解消息系统中不同类型的事件。

## 选择消息传递系统

亚伯拉罕·马斯洛定义的 [*工具法则*](https://en.wikipedia.org/wiki/Law_of_the_instrument) 方法说，“如果你唯一的工具是一把锤子，把一切都当成钉子。”按照这种方法，您当然可以使用经典的消息代理，如 [Apache ActiveMQ Artemis](https://activemq.apache.org/components/artemis/) 来实现本文中描述的不同交互风格。这项技术为许多人所熟悉，这将使它从一开始就易于使用。另一方面，用传统的消息代理开发用例，比如可回放的消息传递，将是一个挑战。

另一个极端是，你可以尝试在每个消息场景中使用类似于 [Apache Kafka](https://kafka.apache.org/) 的东西。Kafka 将需要更多的硬件资源和人力来管理，但是需要可重放消息传递或极端可伸缩性的用例将被涵盖。

虽然上述两种方法在某些情况下都很好，但是当您有大量具有不同消息传递需求的服务时，为正确的工作使用正确的工具是更好的选择。映射前面描述的消息传递模式是决定您需要什么消息传递基础设施的有用工具。在图 3 中，我将三种不同类型的 message broker 系统的特征映射到我们讨论过的不同消息传递场景和消息类型。

[![A chart for comparing messaging systems based business needs and message semantics.](img/1cc63b5ff79c9930297b49797b2c7289.png "messaging_infrastructures")](/sites/default/files/blog/2020/06/messaging_infrastructures-1.png)messaging_infrastructures

Figure 3: Mapping messaging characteristics to messaging infrastructures.

图 3 中的三个框架对于不同类型的消息传递场景和需求都是有效的。正如我在本文开头提到的， [Red Hat AMQ](https://www.redhat.com/en/technologies/jboss-middleware/amq) 打包了所有这三个工具——Apache Qpid、Apache ActiveMQ Artemis 和 Apache Kafka——以便您可以为正确的工作选择正确的工具。

## 结论

当根据您的需求选择合适的事件消息传递基础设施时，有许多方面需要考虑。我希望本文中介绍的考虑因素和映射工具能够帮助您更进一步做出决定。

*Last updated: July 28, 2020*