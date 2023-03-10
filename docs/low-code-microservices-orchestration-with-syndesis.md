# 使用 Syndesis 的低代码微服务编排

> 原文：<https://developers.redhat.com/blog/2020/03/25/low-code-microservices-orchestration-with-syndesis>

最近我写了关于从微服务中分离基础设施代码的文章。我发现 [Apache Camel](https://camel.apache.org/) 和 [Debezium](https://debezium.io/) 为我的项目提供了我需要的中间件，而我只需要最少的代码。在我的成功实验后，我想知道是否有可能将两个或更多类似的解耦微服务编排成一个新服务——我能不写任何代码就完成这个任务吗？我决定找出答案。

本文是对编排微服务的一个快速探索，无需编写任何代码。我们将使用 [Syndesis(一个开源集成平台)](https://syndesis.io/)作为我们的编排平台。注意，这些例子假设你熟悉 Debezium 和 Kafka。

## 关于 Syndesis

Syndesis 是一个开源集成平台，可以连接到您使用的任何服务。它的目标是[市民集成商](https://syndesis.io/about/#citizen-integrator)，他们通常是业务领域的专家，想要快速地将现有的数据源组合成新的微服务，而无需编写任何代码。这些用户拥有宝贵的业务专业知识，微服务架构提供了将他们的知识转化为业务服务的理想平台。我们需要的是一个集成平台，将他们的业务专长连接到技术后端。

注意 Syndesis 是[红帽 Fuse Online](https://www.redhat.com/en/technologies/jboss-middleware/fuse-online) 的开源项目。Fuse Online 运行在[红帽 OpenShift Online](https://www.openshift.com/products/online/) 、[红帽 OpenShift 专用](https://www.openshift.com/products/dedicated/)和[红帽 OpenShift 集装箱平台](https://developers.redhat.com/products/openshift)上。

## 使用 Syndesis 进行微服务编排

我们的目标是集成两个或更多微服务，为游戏商店平台创建一个新的通知服务。使用 Syndesis 作为我们的集成平台即服务(iPaaS ),我们无需编写任何代码即可协调现有的微服务。我们假设一个微服务架构，其中已经实现了 Debezium 和 Kafka 来将基础设施代码从微服务中解耦，遵循的方法类似于我在[上一篇文章](https://developers.redhat.com/blog/2019/11/19/decoupling-microservices-with-apache-camel-and-debezium/)中演示的方法。

我们的游戏商店已经建立了一套运行不同业务部分的微服务。对于这个项目，我们对`GamePrice`和`UserPreference`服务感兴趣。`GamePrice`提供商店中每款游戏的价格。`UserPreference`是一个 AI 引擎，为用户匹配他们会喜欢的游戏。每个微服务都公开了一个 API，我们可以在它的 OpenAPI 文档中查看。

游戏商店偶尔会使用`GamePrice`服务进行产品促销和赠品。现在，我们希望将`GamePrice`与`UserPreference`整合起来，创建一个新的电子邮件提醒微服务。当一个选定的游戏免费提供时，这项服务会提醒用户。我们的集成平台 Syndesis 集成了一个 Debezium 连接器，可以很容易地捕捉到`GamePrice`域中的变化。它还会自动生成连接我们现有微服务的集成逻辑，并向感兴趣的用户(`UserPreference`域)发送个性化的电子邮件提醒。Kafka 将成为 Debezium 产生变更数据事件的消息传递基础设施。我们不需要编写任何代码来启动和运行我们的新通知系统。

### 步骤 1:配置微服务端点

首先，我们需要配置 Syndesis 来使用`GamePrice`和`UserPreference`API。我们通过提供他们的 OpenAPI 端点来做到这一点，如图 1 所示。(注意这个功能是由 [Apicurio](https://www.apicur.io/) 提供的。)

[![A screenshot of the OpenAPI endpoint screen in Syndesis.](img/90f708a716344d3b3b24583c61a0882e.png "syndesis-apicurio")](/sites/default/files/blog/2020/02/syndesis-api.png)Figure 1\. Configure the OpenAPI endpoints.">

### 步骤 2:捕获数据库中的数据更改

接下来，我们需要捕获数据库中价格数据的变化。我们可以创建事件流来捕捉这样的变化。我们将使用一个指向 Kafka 代理的 Syndesis Debezium 连接器，我们假设它已经预先配置好了。Kafka 是 Debezium 用来产生变更数据事件的基础设施。通过选择代理，我们将告诉 Syndesis 我们需要消费哪些事件。图 2 显示了这种配置。

[![A screenshot of the database-configuration page in Syndesis.](img/58fd274d4f230b41340271f8ad13dd0f.png "syndesis-debezium")](/sites/default/files/blog/2020/02/syndesis-debezium.png)Figure 2\. Capture data changes.">

### 步骤 3:配置电子邮件服务

最后，我们配置电子邮件服务。对于这个例子，我选择使用 SMTP 服务(如图 3 所示)，但是您实际上可以使用任何通知类型或服务。

[![A screenshot of the email service configuration screen in Syndesis.](img/197481fa3659e6cd44d34940168a60bf.png "syndesis-email-configuration")](/sites/default/files/blog/2020/02/syndesis-email-configuration.png)Figure 3\. Configure the email service.">

## 构建集成管道

我们已经配置了协调电子邮件提醒所需的所有微服务。我们的下一步将是将`GamePrice`、`UserPreference`、`GamePrice CDC`和`Email`服务整合到一个新的管道中。这些服务如图 4 所示。

[![A screenshot of available microservices in the Syndesis integration dashboard.](img/dba5f35a833f9aeb375dd3e8a8088d4f.png "syndesis-connections")](/sites/default/files/blog/2020/02/syndesis-connections.png)Figure 4\. Microservices displayed in the Syndesis integration .">

我们完成的集成管道将如图 5 所示。

[![A diagram of the integration pipeline.](img/cb873c05b46cab0f091f4412f7930227.png "syndesis-integration-diagram")](/sites/default/files/blog/2020/02/syndesis-integration-diagram.png)Figure 5\. The completed integration pipeline.">

我们开始吧！

### 步骤 1:创建集成管道

我们将从选择 Debezium 连接器作为源开始。我们选择我们需要听的 Kafka 主题和一个特殊的 Kafka 主题，如图 6 所示。Syndesis 将使用第一个主题中产生的事件来选择`GamePrice` drop。它将使用第二个主题来分析事件数据结构，并允许集成器将该数据映射到集成中的每个步骤。

[![A screenshot of the new-integration screen in Syndesis.](img/469df40659f3dda4507585fcd6ffec5b.png "syndesis-debezium-source")](/sites/default/files/blog/2020/02/syndesis-debezium-source.png)Figure 6\. Create the integration pipeline.">

### 步骤 2:添加步骤和连接

接下来，我们开始填充管道，如图 7 所示。作为目的地，我们使用一个过滤价格更新的[条件流](https://www.modeling-guidelines.org/guidelines/correct-usage-of-conditional-and-default-flows)。请注意，您可以根据集成的复杂性添加其他条件。我们还将添加一个日志记录操作来跟踪每次执行的结果。

[![A screenshot of the Syndesis screen for adding new steps and connections to the integration pipeline.](img/520b3bb812bdf9fb0cb7f37f5f6fae44.png "syndesis-integration")](/sites/default/files/blog/2020/02/syndesis-integration.png)Figure 7\. Add steps and connections to the integration pipeline.">

### 步骤 3:检索用户首选项

接下来，我们调用`UserPreference` API 来检索可能对特定报价感兴趣的用户列表，如图 8 所示。我们将使用此列表发送电子邮件，通知他们促销和赠品。

[![](img/1867c553808f13d452f7c8c943553433.png "syndesis-flow")](/sites/default/files/blog/2020/02/syndesis-flow.png)Figure 8\. Retrieve user preferences.">

在我们继续最后的步骤之前，让我们看看管道的整体流程，如图 9 所示。

[![](img/1867c553808f13d452f7c8c943553433.png "syndesis-flow")](/sites/default/files/blog/2020/02/syndesis-flow.png)Figure 9\. The integration pipeline from end to end.">

在第一步中(图 9 中的 1)，价格的变化启动了流程。下一步是基本过滤器(图 9 中的 2)，我们已经配置了它来通知我们价格已经降到零的事件(`price=0`)。接下来，我们转到`UserPreference` API(图 9 中的 4)，在这里我们选择端点并检索可能对指定事件感兴趣的用户列表。请注意，在`UserPreference` API 中，我们还必须进行一些数据映射(图 9 中的 3)，这要感谢 [AtlasMap](https://www.atlasmap.io/) 提供的一个特性。

### 步骤 4:拆分数据

接下来，我们继续(图 9 中的 5)，在这里我们分割在(图 9 中的 4)中获得的用户列表。我们需要[分割数据](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html)以便单独对待每个用户，因为我们想要个性化电子邮件消息。

### 步骤 5:定制消息

现在，我们几乎准备好向我们选定的用户发送促销邀请。但是在此之前，让我们使用模板特性(图 9 中的 7)来个性化我们的消息，如图 10 所示。

[![A screenshot of the template screen in Syndesis.](img/cac087d996408006b5d74d071a5e8e5d.png "syndesis-email-template")](/sites/default/files/blog/2020/02/syndesis-email-template.png)Figure 10\. Templates in the Syndesis pipeline.">

在我们上传了自定义模板之后，我们返回到数据映射器(图 9 中的 8)来定义一个新的映射，它将为每个用户生成个性化的消息。(这个配置类似于我们在(图 9 中的 3)中所做的。)

### 第六步:发送电子邮件

最后，我们准备发送电子邮件，它将我们的个性化信息与我们从`UserPreference service`中提取的电子邮件地址结合在一起。图 11 显示了现场执行的结果。

[![Screenshot of an email for 'Game giveaway!'](img/ba0d52daf4df241e45548667e282dc2b.png "syndesis-email-sent")](/sites/default/files/blog/2020/02/syndesis-email-sent.png)Figure 11\. A personalized email sent from the new microservice.">

运行这个例子的代码和步骤可以在我的 [GitHub 库](https://github.com/squakez/ms-orchestration-syndesis)中找到。

## 结论

在本文中，您已经看到了如何使用 Syndesis 将两个现有的微服务集成到一个新的个性化服务中，以增加业务价值，并且您已经看到了无需编写一行代码就可以做到这一点。使用 Syndesis，没有编程经验的业务专家可以在服务仪表板中发现微服务，基于一个或多个现有服务组成管道，关联来自服务的数据，并构建集成。

剩下要做的就是开始构思能给你的业务带来附加值的集成。

*Last updated: June 29, 2020*