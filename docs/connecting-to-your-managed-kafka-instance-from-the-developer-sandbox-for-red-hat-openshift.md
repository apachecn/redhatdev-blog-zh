# 从 Red Hat OpenShift 的开发人员沙箱连接到托管 Kafka 实例

> 原文：<https://developers.redhat.com/blog/2021/04/23/connecting-to-your-managed-kafka-instance-from-the-developer-sandbox-for-red-hat-openshift>

在您的本地机器上运行概念验证(PoC)代码是很棒的，而且绝对值得，但是当您必须面对[分布式计算](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)的谬误并在云中运行时，真正的考验就来了。这是您从概念验证到更接近真实生活的一个飞跃。

在本文中，我将指导您从运行在另一个集群中的代码中使用 Apache Kafka 的 [Red Hat OpenShift Streams 的过程，该集群位于 Red Hat OpenShift](/products/rhosak/) 的[开发者沙箱上。所有的代码都将在云中运行，你会明白分布式计算的吸引力。](/developer-sandbox)

以下是这一过程所需要的大致概述:

1.  在托管 Kafka ( `samurai-pizza-kafkas`)中创建一个 Kafka 实例。
2.  创建一个主题(`prices`)。
3.  使用`rhoas`命令行界面(CLI)从命令行确认这一切。
4.  使用`quay.io/rhosak/quarkus-kafka-sb-quickstart:latest`处的映像在开发人员沙箱中创建应用程序。
5.  从命令行将服务绑定到您的应用程序。
6.  看结果。

## 先决条件

以下是您在本教程中需要遵循的内容:

*   [Apache Kafka 账户的 OpenShift 流](/products/rhosak/getting-started)
*   [红帽 OpenShift 账号的开发者沙箱](/developer-sandbox)
*   [`rhoas`CLI 工具](https://github.com/redhat-developer/app-services-cli)
*   [open shift CLI`oc`](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html#cli-installing-cli_cli-developer-commands)

你*不会*需要的是一个特定的操作系统。在云中工作的好处在于操作系统、库、连接等的负担。，是“在那里”，而不是在你的机器。你只是在控制它。你可以从一台运行终端会话的低功率 PC 上控制价值数百万美元的计算能力。处理现在是在云中完成的；你的本地电脑只是用来发布命令的。这就是杠杆的力量。那很酷。

## 这一季的大事

事件无处不在。一个事件有它发生的时间和它本身的信息。这两件事赋予了一个事件意义。从哲学的角度来说，所有的存在都是一系列的事件。

对于本文，我们有一个假想的场景。Samurai Pizza 最近陷入困境，一大群对冲基金经理决定“做空”该股。了解到这一点——他们在网上读到的——一个草根组织决定购买大量武士披萨股票，用通俗的话说就是“给那些基金经理上一课”这导致股票价格非常不稳定。

(这完全是我瞎编的，和现实生活中的任何事情都没有关联。这只是一个游戏；停止比较。)

我们的应用程序生成价格并将其发送给 Kafka，然后 Kafka 将事件转发给我们的消费者。最后，用新价格更新网页。随着对冲基金经理和草根个人一决雌雄，欢乐随之而来。

图 1 显示了我们主题的架构概述。

[![Managed Kafka architecture diagram](img/102648d05ed20f266568a6151a9c1d54.png "2021__MK_Diagram_Light")](/sites/default/files/blog/2021/04/2021__MK_Diagram_Light.png)

Figure 1: Managed Kafka architecture diagram.

## 自由管理的卡夫卡审判

第一步是获得 Apache Kafka 的 Red Hat OpenShift 流的免费试用。这是一个简单的过程，不需要任何费用或信用卡号码，这是一个不需要任何安装就可以开始实验的好方法。这确实是最好的云。你可以在这里找到[。](https://developers.redhat.com/products/red-hat-openshift-streams-for-apache-kafka/getting-started)

## 创建卡夫卡实例和主题

导航到图 2 所示的 **Kafka 实例**页面。点击蓝色的大**创建 Kafka 实例**按钮开始。

[![Kafka instances page with no instances listed](img/ed81ccd750cf2961802460542eb2b73a.png "kafka_instances_empty")](/sites/default/files/blog/2021/04/kafka_instances_empty.png)

Figure 2: The Kafka instances page with no instances listed.

系统会提示您输入信息。你需要提供的是卡夫卡实例的名称:`samurai-pizza-kafkas`。您还必须选择云提供商、云区域和可用性区域。点击**创建实例**按钮(见图 3 ),您将很快拥有一个实例。

[![Prompt to create kafka instance](img/f2231cab3a6b9013cd37f79467596196.png "prompt_to_create_kafka_instance")](/sites/default/files/blog/2021/04/prompt_to_create_kafka_instance.png)

Figure 3: Creating a Kafka instance.

等待实例到达**就绪**状态(参见图 4)。要有耐心；我的花了大约五分钟。您可能需要刷新屏幕才能看到状态变化。

[![panel showing kafka instance status as ready](img/3510bd7b357f8e3cd0beb11150b282c4.png "kafka_instance_is_ready_highlighted")](/sites/default/files/blog/2021/04/kafka_instance_is_ready_highlighted.png)

Figure 4: The panel that shows the Kafka instance is ready.

至此，我们已经有了一个可供我们使用的托管 Kafka 实例。我们现在可以创建我们的主题了，`prices`。

## 创建主题

创建我们的主题的步骤遵循熟悉的在仪表板中创建东西的模型:

1.  打开父实例:点击实例名`samurai-pizza-kafkas`。
2.  选择创建子主题:点击**创建主题**按钮。
3.  创建子主题:输入主题名称并接受默认值。

但我们不会那么做。相反，我们将使用命令行。打开终端会话并使用以下命令:`rhoas login`，如下所示:

```
PS C:\Users\dschenck> rhoas login
⣷ Logging in...

 You are now logged in as "rhn-engineering-dschenck"
```

您的浏览器将打开，通知您已登录到您的 Kafka 实例。当然，你的用户名也会不同。

现在回到命令行，我们将使用更多的`rhoas`魔法来创建我们的主题。我们需要三个命令，如图所示:

*   我们使用这个命令来获取我们的 kafka 实例的 id，我们将在下面的命令中使用它。
*   `rhoas kafka use --id <<KAFKA_INSTANCE_ID>>`:该命令允许我们选择我们的实例作为当前实例，即我们想要使用的实例。
*   奇迹就发生在这里。该命令使用默认值创建主题。

```
PS C:\Users\dschenck> rhoas kafka list
  ID (1)                 NAME                   OWNER                      STATUS   CLOUD PROVIDER   REGION
 ---------------------- ---------------------- -------------------------- -------- ---------------- -----------
  c6l32qnnd4k4as5jmvdg   samurai-pizza-kafkas   rhn-engineering-dschenck   ready    aws              us-east-1

PS C:\Users\dschenck> rhoas kafka use --id c6l32qnnd4k4as5jmvdg
 Kafka instance "samurai-pizza-kafkas" has been set as the current instance.

PS C:\Users\dschenck> rhoas kafka topic create --name prices
Topic "prices" created in Kafka instance "samurai-pizza-kafkas":

...JSON removed to save space... 
```

最后一个命令将产生一个 JSON 文档，它简单地定义了主题。通过运行命令`rhoas kafka topic list`(这完全是可选的)，您可以看到更易于阅读的输出。

## 同时，在您的开发人员沙箱中...

让我们启动并运行我们的应用程序。首先，使用`oc login`命令从命令行登录到您的沙盒集群。有关详细说明，请参考我的文章[从命令行](https://developers.redhat.com/blog/2021/04/21/access-your-developer-sandbox-for-red-hat-openshift-from-the-command-line/)访问您的 Red Hat OpenShift 开发人员沙箱。

我们将运行一个已经从源代码编译过的图像。最简单的方法是从您的沙盒集群仪表板。确保您处于开发环境中，选择图 5 所示的 **+Add** 选项。

[![developer context add menu in openshift dashboard](img/348e711fad7b53224c0755a99acee9d6.png "developer_add")](/sites/default/files/blog/2021/04/developer_add.png)

Figure 5: The developer context +Add menu in the OpenShift dashboard.

这将显示一个选项列表；我们想使用**容器图像**选项，所以只需点击那个面板(见图 6)。

[![OpenShift dashboard options for adding an application, with the Container Image panel highlighted.](img/11729c9d670616eda835d7ba71deb014.png "add_options_with_container_image_highlighted")](/sites/default/files/blog/2021/04/add_options_with_container_image_highlighted.png)

Figure 6: OpenShift dashboard options for adding an application.

下一页让我们设置应用程序的参数。在这种情况下，我们需要提供的唯一值是图像名称。在 name 字段中输入`quay.io/rhosak/quarkus-kafka-sb-quickstart:latest`(见图 7 ),然后点击页面底部的 **Create** 按钮。

[![OpenShift deploy image panel](img/82bb10b1a6839e56c004f3dd8f2878d5.png "deploy_image")](/sites/default/files/blog/2021/04/deploy_image.png)

Figure 7: The OpenShift deploy image panel.

OpenShift 将完成剩下的工作:导入映像，在容器中启动它，创建服务，创建路由，并为应用程序创建部署。

只是为了好玩，你可以进入 pod 并查看日志。您将看到应用程序正在抛出错误，因为它无法连接到预期的 Kafka 实例。是时候弥补了。

## 将应用程序连接到 Kafka

我们有一个应用程序和一个 Kafka 实例，主题为(`prices`)我们的应用程序期望。现在我们需要信息来开始使用我们的托管 Kafka 实例。具体来说，我们需要来自托管 Kafka 实例的 API 令牌。我们通过去[https://cloud.redhat.com/openshift/token](https://cloud.redhat.com/openshift/token)并将令牌复制到我们的本地剪贴板来得到它。您将看到如图 8 所示的屏幕。

[![The OpenShift API token.](img/7af784cd8dd116e12a0967e28559826d.png "api_token")](/sites/default/files/blog/2021/04/api_token.png)

Figure 8: The OpenShift API token.

有了令牌，我们可以在命令行运行以下命令，将 Kafka 实例连接到我们的应用程序:

`rhoas cluster connect --token {your token pasted here}`

反过来，这将返回创建服务绑定对象所需的 YAML，该对象将 Kafka 实例绑定到我们的应用程序。输出将类似于这里显示的内容:

```
PS C:\Users\dschenck> rhoas cluster connect --token <<redacted>>
? Select type of service kafka
This command will link your cluster with Cloud Services by creating custom resources and secrets.
In case of problems please execute "rhoas cluster status" to check if your cluster is properly configured

Connection Details:

Service Type:                   kafka
Service Name:                   samurai-pizza-kafkas
Kubernetes Namespace:           rhn-engineering-dschenck-dev
Service Account Secret:         rh-cloud-services-service-account
? Do you want to continue? Yes
 Token Secret "rh-cloud-services-accesstoken" created successfully
 Service Account Secret "rh-cloud-services-service-account" created successfully

Client ID:     srvc-acct-b2f43cd6-da3f-41bd-9190-e1aade856103

Make a copy of the client ID to store in a safe place. Credentials won't appear again after closing the terminal.

You will need to assign permissions to service account in order to use it.
For example for Kafka service you should execute the following command to grant access to the service account:

  $ rhoas kafka acl grant-access --producer --consumer --service-account srvc-acct-b2f43cd6-da3f-41bd-9190-e1aade856103 --topic all --group all

 kafka resource "samurai-pizza-kafkas" has been created
Waiting for status from kafka resource.
Created kafka can be already injected to your application.

To bind you need to have Service Binding Operator installed:
https://github.com/redhat-developer/service-binding-operator

You can bind kafka to your application by executing "rhoas cluster bind"
or directly in the OpenShift Console topology view.

 Connection to service successful.
```

如果您使用的是 Red Hat OpenShift 的开发人员沙盒，则已经安装了服务绑定操作符。

## 重要的 ACL 规则

您可能会注意到，在输出的中间，指示您更新 Kafka 实例的访问控制列表。这允许您的应用程序使用实例和主题。该命令是必需的；这里有一个例子，您的会略有不同:

`rhoas kafka acl grant-access --producer --consumer --service-account srvc-acct-b2f43cd6-da3f-41bd-9190-e1aade856103 --topic all --group all`

## 我们在哪里？

此时，我们有一个带有主题(价格)的 Kafka 实例(samurai-pizza-kafkas ),并且我们有一个在 OpenShift 中运行的应用程序想要使用该实例。我们已经将我们的*集群*连接到 Kafka，但是没有连接*个人应用*。那是下一个。

## 捆绑的粘合剂

我们需要运行一个命令来将 Kafka 实例绑定到我们的应用程序。

`rhoas cluster bind`

这里有一个例子:

```
PS C:\Users\dschenck> rhoas cluster bind
Namespace not provided. Using rhn-engineering-dschenck-dev namespace
Looking for Deployment resources. Use --deployment-config flag to look for deployment configs
? Please select application you want to connect with quarkus-kafka-sb-quickstart
? Select type of service kafka
Binding "samurai-pizza-kafkas" with "quarkus-kafka-sb-quickstart" app
? Do you want to continue? Yes
Using ServiceBinding Operator to perform binding
 Binding samurai-pizza-kafkas with quarkus-kafka-sb-quickstart app succeeded
PS C:\Users\dschenck>
```

## 我们到了吗？

是的，我们已经到了。此时，应用程序正在使用我们创建的 Kafka 实例和主题来生成和消费事件。

## 查看结果

您可以从仪表板打开应用程序的路由，或者使用命令`oc get routes`查看应用程序的 URL。将 URL 粘贴到您的浏览器中，并将`/prices.html`添加到其中以查看结果。

以下是 URL 示例:

```
quarkus-kafka-sb-quickstart-rhn-engineering-dschenck-dev.apps.sandbox-m2.ll9k.p1.openshiftapps.com/prices.html

```

现在你可以看着价格剧烈波动，对冲基金经理和草根阶层一决雌雄。价格将每五秒钟更新一次，全部通过您的托管 Kafka 实例。

## 刚刚发生了什么？

这里发生了一大堆事情，并且是在幕后创造的；我们来分解一下。毕竟，您可能想要撤销此操作，了解发生了什么以及创建了什么是必要的。

当你在托管 Kafka 中创建一个 Kafka 实例时，它也在那里创建了一个 Kafka 实例。很简单。在 Red Hat 托管服务系统中，该实例还被分配了一个惟一的 ID，其密钥是令牌，我们稍后在连接到它时会用到它。

当您运行`rhoas use`并选择您的 Kafka 实例时，您将`rhoas` CLI 的上下文(在您的本地 PC 上)设置为该实例，因此任何后续命令都将违背该实例。这很重要，因为稍后您将连接到您自己的(沙盒)集群；您可以在这里指定连接到集群的内容。

命令只是在 Kafka 中创建了一个主题。如果你不清楚这一点，这里有[一些神奇的材料](https://tracks.redhat.com/l/learn-kafka)让你快速了解。瞬间卡夫卡式的专业知识。

运行`oc login`将您的本地机器连接到您的——在本例中是沙盒——集群。请注意，在本教程的这一点上，您的本地计算机“连接”到您的集群和 Kafka 实例。这让奇迹发生了。

当您在集群中创建应用程序时，它做了很多工作。它从注册表中取出图像并将其放入您的集群中——它位于**图像流**部分(提示:运行命令`oc get imagestreams`)。它创建了一个 pod 来运行应用程序。它创建了一个部署。它创造了一种服务。它创造了一条路线。为什么这很重要？因为如果您想要完全删除应用程序以及与之相关的所有内容，那么有几个对象。

请注意，应用程序为主题(`prices`)提供了硬编码的值。实例名无关紧要；当我们进行服务绑定时，会注入引导服务器主机和端口。顺便提一下，这触发了 OpenShift 用一个新的 pod 替换正在运行的 pod——一个包含所有正确的 Kafka 信息的 pod。

现在，当你用你的令牌运行`rhoas cluster connect`命令时，真正的魔法发生了。这告诉运行在沙盒集群中的 Red Hat OpenShift 应用服务(RHOAS)操作器(您需要在任何使用 Red Hat 托管服务产品的集群中安装该操作器)使用所提供的令牌连接到托管服务(由于您的`rhoas login`命令，它知道该服务)。如果你还记得的话，这个标记标识了你的卡夫卡实例。这就把两者联系起来了。

"等等:这是不是意味着我可以登录到另一个集群，并在那里使用相同的 Kafka 实例？"

是的。在 Kafka 实例中，多个集群。分布式处理、云原生计算、微服务...所有的流行语突然变得有了真正的意义。

`rhoas cluster connect`也在你的集群中创建一些对象。在本例中，是 KafkaConnection 类型的自定义资源。您可以通过运行`oc get kafkaconnection`在命令行中看到这一点。Kubernetes(以及 OpenShift)处理定制资源的能力是一个巨大的优势。这只是一个小例子。

最后，创建了一个 ServiceBinding 对象，它定义了 Kafka 实例和应用程序之间的绑定——还记得您必须修改 YAML 以包含应用程序名称(`quarkus-kafka-sb-quickstart`)的时候吗？

*“为什么股票价格以欧元显示？”*

好吧，为了让这个演示更有趣，我在这里做了一点诗意的许可。此外，对冲基金经理为了控制股价而与一个草根组织争斗？那将永远不会发生。

*Last updated: October 14, 2022*