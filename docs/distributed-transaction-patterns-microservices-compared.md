# 微服务的分布式事务模式比较

> 原文：<https://developers.redhat.com/articles/2021/09/21/distributed-transaction-patterns-microservices-compared>

作为 Red Hat 的一名咨询架构师，我有幸参与了大量的客户项目。每个客户都有自己的挑战，但我发现了一些共性。大多数客户想知道的一件事是如何协调对多个记录系统的写入。回答这个问题通常需要对双重写入、分布式事务、现代替代方案以及每种方法可能出现的故障场景和缺点进行长时间的解释。通常，这是客户意识到[将单片应用拆分成微服务](/articles/2021/06/14/application-modernization-patterns-apache-kafka-debezium-and-kubernetes#)是一个漫长而复杂的过程，并且通常需要权衡的时刻。

本文没有深入讨论事务，而是总结了协调对多个资源的写操作的主要方法和模式。我知道你可能对这些方法中的一种或多种有好的或坏的经历。但是在实践中，在正确的环境和约束下，所有这些方法都可以很好地工作。技术负责人负责选择适合其环境的最佳方法。

**注意**:如果你对双重写入感兴趣，请观看我的 Red Hat Summit 2021 会议，我在会上深入讨论了[双重写入挑战](https://events.summit.redhat.com/widget/redhat/sum21/sessioncatalog/session/1607126048915001oL4X)。你也可以[浏览一下我演讲中的幻灯片](https://www.slideshare.net/bibryam/dual-write-strategies-for-microservices)。目前，我参与了 Apache Kafka 的 Red Hat OpenShift Streams，这是一个完全托管的 Apache Kafka 服务。启动不到一分钟，试用期内完全免费。[试一试](https://console.redhat.com/beta/application-services/streams/kafkas)用您的早期反馈帮助我们塑造它。如果你对这篇文章有任何问题或评论，请在 Twitter [@bibryam](https://twitter.com/bibryam) 上联系我，让我们开始吧。

## 双重写入问题

您可能有双重写入问题的唯一标志是需要可预见地写入多个记录系统。这种需求可能并不明显，在分布式系统设计过程中，它会以不同的方式表达自己。例如:

*   您已经为每项工作选择了最好的工具，现在您必须更新 NoSQL 数据库、搜索索引和缓存，作为单个业务事务的一部分。
*   您设计的服务必须更新其数据库，并向另一个服务发送有关更改的通知。
*   您有跨越多个服务边界的业务事务。
*   您可能必须将服务操作实现为幂等的，因为消费者必须重试失败的调用。

对于本文，我们将使用一个示例场景来评估在分布式事务中处理双重写入的各种方法。我们的场景是一个客户端应用程序，它在一个变化的操作上调用一个微服务。服务 A 必须更新它的数据库，但是它还必须在一个写操作中调用服务 B，如图 1 所示。数据库的实际类型，即服务到服务的交互协议，与我们的讨论无关，因为问题是一样的。

[![The dual write problem in microservices.](img/d838382ff4c8c8f6de6b910fe21b5470.png)](/sites/default/files/1.png)Figure 1: The dual write problem in microservices.

一个小而关键的澄清解释了为什么这个问题没有简单的解决方案。如果服务 A 写入其数据库，然后向服务 B 的队列发送通知(让我们称之为*本地提交然后发布方法*)，应用程序仍然有可能无法可靠地工作。当服务 A 写入其数据库，然后将消息发送到队列时，在提交到数据库之后和第二次操作之前，应用程序很可能会崩溃，这将使系统处于不一致的状态。如果消息是在写入数据库之前发送的(我们称这种方法为*发布然后本地提交*)，那么在服务 A 将更改提交到数据库之前，服务 B 可能会收到事件，从而导致数据库写入失败或时间问题。无论哪种情况，这个场景都涉及到对数据库和队列的双重写入，这是我们将要探讨的核心问题。在接下来的几节中，我将讨论目前针对这一始终存在的挑战可用的各种实现方法。

## 模块化整体结构

将您的应用程序开发成一个模块化的整体可能看起来像是一个黑客或者是架构发展的倒退，但是我已经看到它在实践中工作得很好。它不是微服务模式，而是微服务规则的一个例外，可以谨慎地与微服务结合。当强大的写入一致性是驱动需求，甚至比独立部署和扩展微服务的能力更重要时，您可以选择模块化整体架构。

拥有一个整体架构并不意味着系统设计很差或者很糟糕。它没有说任何关于质量的事情。顾名思义，它是一个以模块化方式设计的系统，只有一个部署单元。请注意，这是一个有目的地设计和实现的[模块化整体](/blog/2016/10/27/the-fast-moving-monolith-how-we-sped-up-delivery-from-every-three-months-to-every-week)，它不同于一个偶然创建的随时间增长的整体。在有针对性的模块化整体架构中，每个模块都遵循微服务原则。每个模块都封装了对其数据的所有访问，但是操作是作为内存中的方法调用公开和使用的。

### 模块化整体建筑

使用这种方法，您必须将两个微服务(服务 A 和服务 B)转换成可以部署到共享运行时的库模块。然后，让两个微服务共享同一个数据库实例。因为服务是作为公共运行时中的库编写和部署的，所以它们可以参与相同的事务。因为这些模块共享一个数据库实例，所以您可以使用本地事务一次提交或回滚所有更改。在部署方法上也有差异，因为我们希望将模块作为库部署在更大的部署中，并参与现有的事务。

即使在整体架构中，也有隔离代码和数据的方法。例如，您可以将模块分离到不同的包、构建模块和源代码存储库中，它们可以归不同的团队所有。通过按照命名约定、模式、数据库实例甚至数据库服务器对表进行分组，可以实现部分数据隔离。图 2 中的图表受 Axel Fontaine 关于[majestic modular monolithics](https://www.youtube.com/watch?v=BOvxJaklcr0)的演讲的启发，展示了应用中不同的代码和数据隔离级别。

[![ Levels of code and data isolation for applications.](img/ad91eb73070adce3a440954fc5bcabc5.png)](/sites/default/files/Screenshot%202021-07-03%20at%2010.39.22.png)Figure 2: Levels of code and data isolation for applications.

难题的最后一部分是使用一个运行时和一个包装器服务，该服务能够消费其他模块并将它们包含在现有事务的上下文中。所有这些约束使模块比典型的微服务耦合得更紧密，但好处是包装器服务可以启动一个事务，调用库模块来更新它们的数据库，并作为一个操作提交或回滚事务，而不用担心部分失败或最终的一致性。

在我们的例子中，如图 3 所示，我们已经将服务 A 和服务 B 转换为库，并将它们部署到共享运行时中，或者其中一个服务可以充当共享运行时。数据库中的表也共享一个数据库实例，但是它被分成一组表，由各自的库服务管理。

[![Modular monolith with a shared database.](img/db27a6a8985bcf3f65cff84d886d5680.png)](/sites/default/files/2.png)Figure 3: Modular monolith with a shared database.

### 模块化整体结构的优点和缺点

在某些行业，事实证明这种架构的好处远比其他地方非常重视的更快的交付和变化速度更重要。表 1 总结了模块化整体结构的优点和缺点。

Table 1: Benefits and drawbacks of the modular monolith architecture.

| **好处** | 带有本地事务的简单事务语义确保了数据一致性、自读自写、回滚等。 |
| **弊端** | 

*   The shared runtime makes it impossible for us to deploy and expand modules independently, and it is also impossible to isolate faults.
*   The logical separation of tables in a single database is not strong. Over time, it can become a shared integration layer.
*   Module coupling and sharing transaction context need to be coordinated in the development stage, which increases the coupling between services.

 |
| **例题** | 

*   Runtime, such as Apache Karaf and WildFly, allows modularity and dynamic deployment of services.
*   The `direct` and `direct-vm` components of Apache Camel allow the operations of memory calls to be exposed, and the transaction context is preserved in the JVM process.
*   Apache Isis is one of the best examples of modular overall architecture. It supports domain-driven application development by automatically generating UI and REST APIs for your Spring Boot application. Apache OFBiz is another example of modular holistic architecture and service-oriented architecture (SOA). This is a comprehensive enterprise resource planning system with hundreds of forms and services, which can realize the automation of enterprise business processes. Although it is small, its modular architecture allows developers to quickly understand and customize it.

 |

分布式事务通常是最后的手段，用于各种情况:

*   何时对不同资源的写入最终无法保持一致。
*   当我们必须写入异构数据源时。
*   当需要恰好一次的消息处理，并且我们不能重构系统并使其操作幂等时。
*   当与实现两阶段提交规范的第三方黑盒系统或遗留系统集成时。

在所有这些情况下，当不考虑可伸缩性时，我们可能会考虑分布式事务。

### 实现两阶段提交体系结构

两阶段提交的技术要求是，您需要一个分布式事务管理器，如 [Narayana](https://narayana.io/) 和一个可靠的事务日志存储层。您还需要与 [DTP XA](https://publications.opengroup.org/standards/dist-computing/c193) 兼容的数据源，以及能够参与分布式事务的相关 XA 驱动程序，比如 RDBMS、消息代理和缓存。如果您幸运地拥有合适的数据源，但是运行在一个动态环境中，比如 [Kubernetes](/topics/kubernetes) ，那么您还需要一个类似操作员的机制来确保只有一个分布式事务管理器的实例。事务管理器必须高度可用，并且必须始终能够访问事务日志。

对于实现，您可以探索一个[snow drop Recovery Controller](https://github.com/snowdrop/narayana-spring-boot/tree/master/openshift/recovery-controller)，它使用 [Kubernetes StatefulSet 模式](/blog/2020/05/11/top-10-must-know-kubernetes-design-patterns)用于单例目的，并使用持久性卷来存储事务日志。在这一类别中，我还包括了诸如用于 SOAP web 服务的 [Web 服务原子事务](http://docs.oasis-open.org/ws-tx/wstx-wsat-1.2-spec.html) (WS-AtomicTransaction)等规范。所有这些技术的共同点是它们都实现了 XA 规范，并且有一个中央事务协调器。

在我们的示例中，如图 4 所示，服务 A 使用分布式事务将所有更改提交到其数据库，并将一条消息提交到一个队列，不会留下任何重复或丢失消息的机会。类似地，服务 B 可以使用分布式事务来消费消息，并在没有任何副本的单个事务中提交到数据库 B。或者，服务 B 可以选择不使用分布式事务，而是使用本地事务并实现幂等消费者模式。对于本节来说，更合适的例子是使用 WS-AtomicTransaction 来协调单个事务中对数据库 A 和数据库 A 的写入，并避免最终的一致性。但是这种方法比我描述的更不常见。

[![Two-phase commit spanning between a database and a message broker.](img/0ab4ff6a484419d1f30b1fece804c3f5.png)](/sites/default/files/3.png)Figure 4: Two-phase commit spanning between a database and a message broker.

### 两阶段提交架构的优点和缺点

两阶段提交协议为模块化整体方法中的本地事务提供了类似的保证，但也有一些例外。因为在原子更新中涉及到两个或多个独立的数据源，所以它们可能会以不同的方式失败并阻塞事务。但是由于它的中央协调器，与我将讨论的其他方法相比，发现分布式系统的状态仍然很容易。

表 2 总结了这种方法的优点和缺点。

Table 2: Benefits and drawbacks of two-phase commit.

| **好处** | 

*   Standard-based method, with ready-made transaction manager and supporting data source.
*   Strong data consistency is a happy scene.

 |
| **弊端** | 

*   Extensibility constraints.
*   Recovery failure that may occur when the transaction manager fails.
*   Limited data source support.
*   Storage and monomer demand in dynamic environment.

 |
| **例题** | 

*   [Jakarta transaction API](https://en.wikipedia.org/wiki/Java_Transaction_API) (formerly Java transaction API)
*   WS-atomic 事务
*   JTS/IIOP
*   易贝[砂砾](https://tech.ebayinc.com/engineering/grit-a-protocol-for-distributed-transactions-across-microservices/)
*   Atomikos
*   纳拉亚纳
*   Message brokers such as Apache ActiveMQ, which implements the XA specification.
*   Relational data source, in-memory data storage such as Infinispan

 |

## 管弦乐编曲

对于一个模块化的整体，我们使用本地事务，我们总是知道系统的状态。对于基于两阶段提交协议的分布式事务，我们也保证了一致的状态。唯一的例外是涉及事务协调器的不可恢复的故障。但是，如果我们想在了解整个分布式系统的状态并从一个地方进行协调的同时降低一致性要求，那该怎么办呢？在这种情况下，我们可以考虑一种*编排*方法，其中一个服务充当整个分布式状态更改的协调者和编排者。orchestrator 服务有责任调用其他服务，直到它们达到所需的状态，或者在它们失败时采取纠正措施。orchestrator 使用其本地数据库来跟踪状态变化，并负责恢复与状态变化相关的任何故障。

### 实现编排架构

编排技术最流行的实现是 BPMN 规范的实现，比如 jBPM 项目[和 Camunda 项目](https://www.jbpm.org/)和[项目。对这种系统的需求不会随着过度分布式架构(如微服务或无服务器)而消失；反之，则增加。为了证明，我们可以看看更新的状态编排引擎，它们不遵循规范，但提供类似的状态行为，如网飞的](https://github.com/camunda/camunda-bpm-platform)[指挥](https://netflix.github.io/conductor/)，优步的[节奏](https://github.com/uber/cadence)，以及 Apache 的[气流](https://airflow.apache.org/)。Amazon StepFunctions、Azure Durable Functions 和 Azure Logic Apps 等无服务器状态函数也属于这一类。还有一些开源库允许你实现状态协调和回滚行为，比如 Apache Camel 的 [Saga](https://camel.apache.org/components/latest/eips/saga-eip.html) 模式实现和 NServiceBus [Saga](https://docs.particular.net/nservicebus/sagas/) 功能。许多实现 Saga 模式的国产系统也属于这一类。

[![Orchestrating distributed transactions between two services.](img/12f1b4a2256abda14e9d0b507a39f4f1.png)](/sites/default/files/4.png)Figure 5: Orchestrating distributed transactions between two services.

在图 5 所示的示例图中，服务 A 充当有状态编排器，负责调用服务 B，并在需要时通过补偿操作从故障中恢复。这种方法的关键特征是服务 A 和服务 B 具有本地事务边界，但是服务 A 具有编排整个交互流的知识和责任。这就是它的事务边界触及服务 B 端点的原因。就实现而言，我们可以通过同步交互来设置，如图所示，或者在服务之间使用消息队列(在这种情况下，您也可以使用两阶段提交)。

### 编排的优点和缺点

编排是一种*最终一致*的方法，可能涉及重试和回滚，以使分布达到一致状态。虽然编排避免了对分布式事务的需求，但它要求参与的服务提供幂等操作，以防协调器必须重试操作。参与服务还必须提供恢复端点，以防协调器决定回滚并修复全局状态。这种方法的最大优点是能够通过仅使用本地事务将可能不支持分布式事务的异构服务驱动到一致的状态。协调器和参与的服务只需要本地事务，通过询问协调器总是可以发现系统的状态，即使它处于部分一致的状态。我将描述的其他方法不可能做到这一点。

Table 3: Benefits and drawbacks of orchestration.

| **好处** | 

*   Coordinate the state between heterogeneous distributed components.
*   No XA transaction is required.
*   Distributed status at the known coordinator level.

 |
| **弊端** | 

*   Complex distributed programming model.
*   You may need to participate in idempotent and compensation operations of services.

*   Possible unrecoverable faults during compensation.

 |
| **例题** | 

*   jBPM
*   在窗户里
*   Microprofile [Long-time running action](https://github.com/eclipse/microprofile-lra)
*   conductor
*   pace
*   Step function
*   Persistent function
*   Realization of Apache camel legend model
*   Realization of legendary mode of NServiceBus
*   CNCF 【T0] serverless workflow specification
*   Autonomous realization

 |

## 舞蹈编排

正如您在目前的讨论中所看到的，单个业务操作可能会导致服务之间的多次调用，并且在端到端地处理业务事务之前，可能需要不确定的时间。为了管理这一点，编排模式使用一个集中式控制器服务来告诉参与者做什么。

编排的另一种选择是*编排*，这是一种服务协调风格，参与者在没有集中控制点的情况下交换事件。使用这种模式，每个服务执行一个本地事务，并发布触发其他服务中的本地事务的事件。系统的每一个组件都参与商业事务工作流程的决策制定，而不是依赖于一个中心控制点。历史上，编排方法最常见的实现是使用异步消息传递层进行服务交互。图 6 展示了编排模式的基本架构。

[![Service choreography through a messaging layer.](img/954dfb6cb276c17756a482c22a6a6efe.png)](/sites/default/files/5.png)Figure 6: Service choreography through a messaging layer.

### 双重写作编舞

为了让基于消息的编排工作，我们需要每个参与的服务执行一个本地事务，并通过向消息传递基础设施发布命令或事件来触发下一个服务。类似地，其他参与服务必须使用消息并执行本地事务。这本身就是更高级别的双写问题中的双写问题。当我们开发一个具有双重写入的消息传递层来实现编排方法时，我们可以将其设计为一个跨越本地数据库和消息代理的两阶段提交。我在前面介绍过这种方法。或者，我们可以使用*发布然后本地提交*或*本地提交然后发布*模式:

*   **发布然后本地提交**:我们可以尝试先发布消息，然后提交本地事务。虽然这种选择听起来不错，但它有实际的挑战。例如，您经常需要发布一个由本地事务提交生成的 ID，该 ID 不能发布。此外，本地事务可能会失败，但我们无法回滚已发布的消息。这种方法缺乏“先读后写”的语义，对于大多数用例来说，这是一种不切实际的解决方案。
*   **本地提交然后发布**:稍微好一点的方法是先提交本地事务，然后发布消息。在提交本地事务之后和发布消息之前，出现失败的可能性很小。但是即使在这种情况下，您也可以将您的服务设计成幂等的，然后重试该操作。这意味着再次提交本地事务，然后发布消息。如果你控制了下游消费者，这种方法是可行的，也可以使他们幂等。总体而言，这也是一个非常好的实现选项。

### 没有双重写作的编排

实现编排体系结构的各种方法限制了每个服务只能通过本地事务写入单个数据源，而不能写入其他地方。让我们看看在没有双重写入的情况下如何工作。

假设服务 A 收到一个请求，并将其写入数据库 A，而不是其他地方。服务 B 定期轮询服务 A 并检测新的变化。当它读取更改时，服务 B 用该更改更新它自己的数据库，还更新它选择该更改的索引或时间戳。这里的关键部分是，两个服务都只写入它们自己的数据库，并提交本地事务。如图 7 所示，这种方法可以被描述为*服务编排*，或者我们可以使用古老的数据管道术语来描述它。可能的实现选项更有趣。

[![Service choreography through polling.](img/41cac4c203dfc1f1e19b282ac75502ac.png)](/sites/default/files/5.1.png)Figure 7: Service choreography through polling.

最简单的场景是服务 B 连接到服务 A 的数据库并读取服务 A 拥有的表。然而，业界试图避免与共享表的这种程度的耦合，这是有充分理由的:服务 A 的实现和数据模型的任何变化都可能破坏服务 B。我们可以对此场景进行一些逐步的改进，例如通过使用[发件箱模式](https://microservices.io/patterns/data/transactional-outbox.html)并给服务 A 一个充当公共接口的表。该表可以只包含服务 B 所需的数据，并且可以设计成易于查询和跟踪更改。如果这还不够好，进一步的改进将是服务 B 通过 [API 管理层](/products/red-hat-openshift-api-management/overview)请求服务 A 做出任何更改，而不是直接连接到数据库 A

从根本上说，所有这些变化都有相同的缺点:服务 B 必须不断地轮询服务 A。这样做可能会导致系统不必要的持续负载，或者在获取更改时出现不必要的延迟。轮询微服务的变化是一个硬推销，所以让我们看看我们可以做些什么来进一步改善这个架构。

### 用 Debezium 编舞

改进编排架构并使其更有吸引力的一个方法是引入一个像 [Debezium](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/) 这样的工具，我们可以用它来使用数据库 A 的事务日志执行变更数据捕获(CDC)。图 8 展示了这种方法。

[![Service choreography with change data capture.](img/a47b4bf9c19ba951d261b20cf4e408ae.png)](/sites/default/files/6.png)Figure 8: Service choreography with change data capture.

Debezium 可以监控数据库的事务日志，执行任何必要的过滤和转换，并将相关的更改提交到 Apache Kafka 主题中。这样，服务 B 可以监听主题中的一般事件，而不是轮询服务 A 的数据库或 API。将数据库轮询转换为流更改，并在服务之间引入队列，这使得分布式系统更加可靠、可伸缩，并为新用例引入其他消费者提供了可能性。使用 Debezium 为基于编排或编排的[传奇模式实现](https://www.infoq.com/articles/saga-orchestration-outbox/)提供了一种优雅的方式来实现[发件箱模式](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/)。

这种方法的一个副作用是它引入了服务 B 接收重复消息的可能性。这可以通过将服务实现为等幂来解决，要么在业务逻辑级别，要么使用技术上的重复数据删除器(使用类似 Apache ActiveMQ Artemis 的[重复消息检测](https://activemq.apache.org/components/artemis/documentation/1.1.0/duplicate-detection.html)或 Apache Camel 的等幂消费者模式)。

### 事件源编排

*事件源*是服务编排方法的另一个实现。使用这种模式，实体的状态被存储为一系列状态改变事件。当有新的更新时，不是更新实体的状态，而是将新的事件追加到事件列表中。将新事件附加到事件存储是在本地事务中完成的原子操作。如图 9 所示，这种方法的优点在于事件存储的行为类似于其他服务消费更新的消息队列。

[![Service choreography through event sourcing.](img/d381688eab3833c193596f508f31651d.png)](/sites/default/files/7.png)Figure 9: Service choreography through event sourcing.

我们的示例在转换为使用事件源时，会将客户端请求存储在只附加事件存储中。服务 A 可以通过重放事件来重建其当前状态。事件存储还需要允许服务 B 订阅相同的更新事件。通过这种机制，服务 A 也将其存储层用作与其他服务的通信层。虽然这种机制非常简洁，解决了每当状态发生变化时可靠地发布事件的问题，但它引入了许多开发人员不熟悉的新编程风格，并增加了状态重构和消息压缩的复杂性，这需要专门的数据存储。

### 编舞的优点和缺点

不管用于检索数据更改的机制是什么，编排方法都将写入解耦，允许独立的服务可伸缩性，并提高整体系统弹性。这种方法的缺点是决策流程是分散的，很难发现全球分布的状态。发现请求的状态需要查询多个数据源，这对于大量的服务来说是一个挑战。表 4 总结了这种方法的优点和缺点。

Table 4: Benefits and drawbacks of choreography.

| **好处** | 

*   Decoupling implementation and interaction.
*   There is no central affairs coordinator.
*   And the characteristics of expandability and toughness are improved.
*   Near real-time interaction.
*   Reduce system overhead with Debezium and similar tools.

 |
| **弊端** | 

*   Global system state and coordination logic are scattered among all participants.
*   Final consistency.

 |
| **例题** | 

*   Self-produced database or API polling implementation.
*   Outbox mode
*   The arrangement is based on the legendary mode.
*   [Event Procurement](https://debezium.io/blog/2020/02/10/event-sourcing-vs-cdc/)
*   [Final result](https://eventuate.io/)
*   [Debezium](https://github.com/debezium/debezium)
*   [Maxwell of Zendesk](https://github.com/zendesk/maxwell)
*   [Alibaba's Canal](https://github.com/alibaba/canal)
[Linkedin 的布鲁克林](https://github.com/linkedin/Brooklin/)

 |

## 并行管道

使用编排模式，没有中央位置来查询系统的状态，但是有一系列服务通过分布式系统传播状态。编排创建了处理服务的顺序管道，因此我们知道当消息到达整个流程的某个步骤时，它已经通过了所有前面的步骤。如果我们可以放松这种约束，独立处理所有步骤，会怎么样？在这个场景中，服务 B 可以处理一个请求，而不管服务 A 是否已经处理了它。

对于并行管道，我们添加了一个路由器服务，该服务接受请求，并在单个本地事务中通过消息代理将请求转发给服务 A 和服务 B。从这一步开始，如图 10 所示，两个服务都可以独立并行地处理请求。

[![Processing through parallel pipelines.](img/d21608428556741bd8ab5570471cf5b3.png)](/sites/default/files/8.png)Figure 10: Processing through parallel pipelines.

虽然这种模式实现起来非常简单，但它只适用于服务之间没有时间绑定的情况。例如，服务 B 应该能够处理请求，而不管服务 A 是否已经处理了相同的请求。此外，这种方法需要一个额外的路由器服务，或者客户端需要知道服务 A 和 B 来确定消息的目标。

### 听听你自己

这种方法有一个更简单的替代方案，称为[倾听自己](https://medium.com/@odedia/listen-to-yourself-design-pattern-for-event-driven-microservices-16f97e3ed066)模式，其中一个服务也充当路由器。使用这种替代方法，当服务 A 收到请求时，它不会写入其数据库，而是将请求发布到消息传递系统中，在该系统中，请求被定向到服务 B 和它自己。图 11 展示了这种模式。

[![The Listen to yourself pattern.](img/2b4c7115a80bd0b9d586230ad6b81f79.png)](/sites/default/files/9_0.png)Figure 11: The Listen to yourself pattern.

不写入数据库的原因是为了避免双重写入。一旦消息进入消息传递系统，该消息就进入服务 B，并且在完全独立的事务上下文中进入后台服务 A。通过改变处理流程，服务 A 和服务 B 可以独立处理请求并写入各自数据库。

### 并行管道的优点和缺点

表 5 总结了使用并行管道的优点和缺点。

Table 5: Benefits and drawbacks of parallel pipelines.

| **好处** | 简单、可扩展的并行处理架构。 |
| **弊端** | 需要暂时拆除；很难对全球系统状态进行推理。 |
| **例题** | Apache Camel 的并行处理多播和分离器。 |

## 如何选择分布式事务策略

您可能已经从本文中猜到，在微服务架构中处理分布式事务没有正确或错误的模式。每种模式都有其利弊。每种模式解决一些问题，同时又产生其他问题。图 12 中的图表简要总结了我所讨论的双重写入模式的主要特征。

[![Characteristics of dual write patterns.](img/01bdfbfadb623c53c4e83b2234dba53e.png)](/sites/default/files/10.png)Figure 12: Characteristics of dual write patterns.

无论您选择哪种方法，您都需要解释并记录决策背后的动机以及您的选择所带来的长期架构后果。您还需要从长期实施和维护系统的团队中获得支持。我喜欢根据数据一致性和可伸缩性属性来组织和评估本文中描述的方法，如图 13 所示。

[![Relative data consistency and scalability characteristics of dual write patterns.](img/54e55642417a204fc6ba83070e6dae61.png)](/sites/default/files/11.png)Figure 13: Relative data consistency and scalability characteristics of dual write patterns.

作为一个好的起点，我们可以评估各种方法，从最具可伸缩性和高可用性的方法到最不具可伸缩性和可用性的方法。

### 高:并行管道和编排

如果您的步骤是暂时解耦的，那么以并行管道方法运行它们是有意义的。您可以将这种模式应用于系统的某些部分，但不能应用于所有部分。接下来，假设处理步骤之间存在时间耦合，并且某些操作和服务必须在其他操作和服务之前发生，您可能会考虑编排方法。使用服务编排，有可能创建一个可伸缩的、[事件驱动的架构](/topics/event-driven)，其中消息通过一个分散的编排过程从一个服务流向另一个服务。在这种情况下，使用 Debezium 和 Apache Kafka 的发件箱模式实现(比如针对 Apache Kafka 的[Red Hat open shift Streams](https://developers.redhat.com/products/red-hat-openshift-streams-for-apache-kafka/getting-started))特别有趣，并且越来越受欢迎。

### 中等:编排和两阶段提交

如果编排不是很合适，并且您需要一个负责协调和决策的中心点，那么您可以考虑编排。这是一个流行的架构，有基于标准的和定制的开源实现。虽然基于标准的实现可能会强制您使用某些事务语义，但自定义编排实现允许您在所需的数据一致性和可伸缩性之间进行权衡。

### 低:模块化整体

如果您走得更远，很可能您对数据一致性有非常强烈的需求，并且您已经准备好以重大的折衷来为此付出代价。在这种情况下，通过两阶段提交的分布式事务可以处理某些数据源，但是它们很难在为可伸缩性和高可用性而设计的动态云环境中可靠地实现。在这种情况下，您可以一直使用传统的模块化整体方法，并结合从微服务运动中学到的实践。这种方法确保了最高的数据一致性，但代价是运行时和数据源耦合。

## 结论

在一个有几十个服务的大型分布式系统中，不会有一种方法适用于所有服务，但是有一些方法可以组合起来应用于不同的环境。您可能在共享运行时上部署了一些服务，以满足数据一致性方面的特殊需求。您可以选择两阶段提交来与支持 JTA 的遗留系统集成。您可能会编排一个复杂的业务流程，并对其余的服务使用编排和并行处理。最后，你选择什么策略并不重要；重要的是出于正确的原因，有意识地选择一个策略，并执行它。

*Last updated: September 12, 2022*