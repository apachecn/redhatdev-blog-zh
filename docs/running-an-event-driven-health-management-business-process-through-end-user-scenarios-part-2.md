# 通过最终用户场景运行事件驱动的健康管理业务流程:第 2 部分

> 原文：<https://developers.redhat.com/blog/2020/04/14/running-an-event-driven-health-management-business-process-through-end-user-scenarios-part-2>

如果您阅读了本系列的第一篇文章，那么您已经建立了本文所需的示例应用程序。如果您尚未设置人口健康管理应用程序，您应该在继续之前设置。在本文中，我们将通过事件和业务流程驱动的应用程序运行一些业务流程来测试它。

## 幸福之路的场景

从本文中您可以学到的最重要的事情是如何将用户界面(UI)与由 [jBPM](https://www.jbpm.org/) 业务流程驱动的业务应用程序的后端连接在一起。将这些组件连接在一起并没有很好的文档记录，可能需要反复试验才能正确。

如果您是一名业务流程开发人员，您可以通过 jBPM Business Central 运行业务流程场景来轻松验证它。但是，如果您是一名 UI 开发人员，并且需要构建或定制一个面向用户的业务应用程序，那么这样做不会对您有太大帮助。

在人口健康管理应用程序的情况下，您开发的业务流程驱动一个或几个最终用户是家庭医生、药剂师或保险代理的应用程序。他们每个人都需要访问成员记录。

您的任务是将各方使用的前端应用程序连接到 jBPM 流程服务器中运行的单个业务流程，也称为 *KieServer* 。这个服务器使用 REST API 来公开各种各样的操作。出于本文的目的，对 KieServer REST API 的调用将使用基本身份验证。在现实生活中，你更有可能使用更强的认证机制，比如在 [Keycloak](https://www.keycloak.org/) 中实现的 [OAuth](https://oauth.net/) 。

在本文的剩余部分，我将使用 [cURL](https://curl.haxx.se/docs/manual.html) 命令来演示如何从一个示例终端用户应用程序 UI 中调用 REST API。从 Linux 命令行界面(CLI)和 Windows PowerShell 中都可以使用`cURL`命令。在为最终用户应用程序开发或定制 UI 时，调用 REST API 的特定方式会有所不同。

您可以通过运行快速获得关于服务器的一些基本信息:

```
curl --user kieserver:secret \
--location --request GET 'http://localhost:8080/kie-server/services/rest/server?Accept=application/json' \
--header 'Accept: application/json'

```

响应将是，[例如](https://gist.github.com/mauriziocarioli/2bc9eec14bda0fd2c9bcc7f91d14481c#file-getserver-json):

```
{
  "type": "SUCCESS",
  "msg": "Kie Server info",
  "result": {
    "kie-server-info": {
      "id": "sample-server",
      "version": "7.33.0.Final",
      "name": "sample-server",
      "location": "http://localhost:8080/kie-server/services/rest/server",
      "capabilities": [
        "KieServer",
        "BRM",
        "BPM",
        "CaseMgmt",
        "BPM-UI",
        "BRP",
        "DMN",
        "Swagger",
        "Prometheus"
      ],
      "messages": [
        {
          "severity": "INFO",
          "timestamp": {
            "java.util.Date": 1580476689750
          },
          "content": [
            "Server KieServerInfo{serverId='sample-server', version='7.33.0.Final', name='sample-server', location='http://localhost:8080/kie-server/services/rest/server', capabilities=[KieServer, BRM, BPM, CaseMgmt, BPM-UI, BRP, DMN, Swagger, Prometheus]', messages=null', mode=DEVELOPMENT}started successfully at Fri Jan 31 08:18:09 EST 2020"
          ]
        }
      ],
      "mode": "DEVELOPMENT"
    }
  }
}

```

请注意服务器功能的列表。

现在，[获取该服务器上的部署单元列表](https://gist.github.com/mauriziocarioli/7ed015fd324a2e268af5866c72a9e9d0)(也称为 [Kie 容器](https://docs.jboss.org/jbpm/release/latest/jbpm-docs/html_single/#_creating_a_kie_container)，不要与 [OCS 容器](https://docs.openshift.com/enterprise/3.0/architecture/core_concepts/containers_and_images.html)或 [Docker 容器](https://www.docker.com/resources/what-container)混淆:

```
curl --user kieserver:secret \
--location --request GET 'http://localhost:8080/kie-server/services/rest/server/containers' \
--header 'Accept: application/json'

```

[响应](https://gist.github.com/mauriziocarioli/d1dcb0a0be8275c1007170510c21c909)是 PHM-模型和 PHM-流程容器:

```
{
  "type": "SUCCESS",
  "msg": "List of created containers",
  "result": {
    "kie-containers": {
      "kie-container": [
        {
          "container-id": "PHM-Processes_1.0.0-SNAPSHOT",
          "release-id": {
            "group-id": "com.health-insurance",
            "artifact-id": "PHM-Processes",
            "version": "1.0.0-SNAPSHOT"
          },
          "resolved-release-id": {
            "group-id": "com.health-insurance",
            "artifact-id": "PHM-Processes",
            "version": "1.0.0-SNAPSHOT"
          },
          "status": "STARTED",
          "scanner": {
            "status": "DISPOSED",
            "poll-interval": null
          },
          "config-items": [
            {
              "itemName": "KBase",
              "itemValue": "",
              "itemType": "BPM"
            },
            {
              "itemName": "KSession",
              "itemValue": "",
              "itemType": "BPM"
            },
            {
              "itemName": "MergeMode",
              "itemValue": "MERGE_COLLECTIONS",
              "itemType": "BPM"
            },
            {
              "itemName": "RuntimeStrategy",
              "itemValue": "PER_PROCESS_INSTANCE",
              "itemType": "BPM"
            }
          ],
          "messages": [
            {
              "severity": "INFO",
              "timestamp": {
                "java.util.Date": 1580477082631
              },
              "content": [
                "Container PHM-Processes_1.0.0-SNAPSHOT successfully created with module com.health-insurance:PHM-Processes:1.0.0-SNAPSHOT."
              ]
            }
          ],
          "container-alias": "PHM-Processes"
        },
        {
          "container-id": "PHM-Model_1.0.0-SNAPSHOT",
          "release-id": {
            "group-id": "com.health-insurance",
            "artifact-id": "PHM-Model",
            "version": "1.0.0-SNAPSHOT"
          },
          "resolved-release-id": {
            "group-id": "com.health-insurance",
            "artifact-id": "PHM-Model",
            "version": "1.0.0-SNAPSHOT"
          },
          "status": "STARTED",
          "scanner": {
            "status": "DISPOSED",
            "poll-interval": null
          },
          "config-items": [
            {
              "itemName": "KBase",
              "itemValue": "",
              "itemType": "BPM"
            },
            {
              "itemName": "KSession",
              "itemValue": "",
              "itemType": "BPM"
            },
            {
              "itemName": "MergeMode",
              "itemValue": "MERGE_COLLECTIONS",
              "itemType": "BPM"
            },
            {
              "itemName": "RuntimeStrategy",
              "itemValue": "SINGLETON",
              "itemType": "BPM"
            }
          ],
          "messages": [
            {
              "severity": "INFO",
              "timestamp": {
                "java.util.Date": 1580476858851
              },
              "content": [
                "Container PHM-Model_1.0.0-SNAPSHOT successfully created with module com.health-insurance:PHM-Model:1.0.0-SNAPSHOT."
              ]
            }
          ],
          "container-alias": "PHM-Model"
        }
      ]
    }
  }
}

```

响应包含每个 Kie 容器的大量信息，但现在您只需要名称。在接下来的对 PHM-Processes Kie 容器的 REST API 调用中，使用别名`PHM-Processes`。不需要使用完整的容器 id `PHM-Processes_1.0.0-SNAPSHOT`。

假设 PHM 业务应用程序的数据集成层在数据流上接收到给定成员的某个触发器，就像 [Apache Kafka](https://developers.redhat.com/blog/tag/apache-kafka/) 一样。(在本例中，集成组件是 [Apache Camel](https://developers.redhat.com/blog/tag/apache-camel/) 。)如图 1 所示，Camel 事件驱动的消费者订阅了 Kafka 流中的一个频道。每当收到触发事件时，Camel 都会在 jBPM 中启动一个触发流程实例。

[![High level architecture of the PHM application.](img/a4dcb637aa805ceb7535e6c8c5617f72.png "PHM High Level Architecture")](/sites/default/files/blog/2020/03/PHM-High-Level-Architecture.png)Figure 1: High level architecture of the PHM application.Figure 1: The PHM application's high-level architecture.">

在我看来，BPM 应该编排业务活动，比如人工任务、决策任务以及直接附属于这些业务任务的服务。数据集成不应该是 BPM 的工作，最好留给专门为此目的设计的工具。让 Camel 来做繁重的工作，让 jBPM 专注于真正的业务逻辑，而不是技术数据操作。我们的目标是拥有一个可伸缩的、健壮的、敏捷的 PHM 解决方案，而不是将每个功能都塞进一个瑞士刀工具中。

现在，开始触发程序。在现实生活中，Camel 将通过使用下面的 REST API[启动 jBPM 中的触发器流程来对触发器做出反应，其中您只需要传递成员 id 和触发器 id:](https://gist.github.com/mauriziocarioli/71dc46001fed17b2b284b1c2d4d74d21)

```
curl --user kieserver:secret \
--location --request POST 'http://localhost:8080/kie-server/services/rest/server/containers/PHM-Processes/processes/PHM-Processes.Trigger/instances' \
--header 'Content-Type: application/json' \
--data-raw '{
     "pMemberId": "123",
     "pTriggerId": "R383"
}'

```

响应将是流程实例 id。

现在，请戴上您的业务流程开发人员的帽子。进入**业务中心**，找到**管理**板块，点击**流程实例**，如图 2 所示。

[![Business Central - Manage - process instances.](img/98439aa2a5da91aa77a0930fa7396abc.png "2020-02-24_11-35-20")](/sites/default/files/blog/2020/03/2020-02-24_11-35-20.png)Figure 9: Business Central - Manage - process instances.Figure 2: Go to **Business Central** to manage your process instances.">

在这里，您将看到三个流程实例。除了触发流程实例之外，您将看到两个没有前置任务的任务流程实例(图 3)。

[![Business Central - Manage - Process instances](img/1d9663543e9c293f7ec983e1e4503487.png "2020-02-24_11-35-52")](/sites/default/files/blog/2020/03/2020-02-24_11-35-52.png)Figure 10: Business Central - Manage - Process instances.

单击触发器流程实例，查看可用的信息。如果您是业务流程开发人员，您会发现图 4 所示的流程图在快速调试或测试时非常有用。它显示流程执行在哪里处于等待状态(红色)以及哪些活动或节点已经完成(灰色)。

[![Trigger process instance diagram](img/3438250884c0f4f44c4d05372ea2cf0c.png "2020-02-24_11-38-16")](/sites/default/files/blog/2020/03/2020-02-24_11-38-16.png)Figure 12: Trigger process instance diagram.

然而，测试和调试 jBPM 业务流程和 Drools 业务规则的最强大的工具是事件监听器接口。没有它们就不应该部署任何流程——看看这个在触发流程开始时由定制事件监听器捕获的事件的[打印输出。](https://gist.github.com/mauriziocarioli/4879bbd0ce366e3b1c07b41c96d21ab9)

捕捉到什么，如何展示，完全由你决定。通常，您会将这些跟踪数据存储到数据库中，以便在需要时可以轻松地从中提取报告。

现在，戴上你的用户界面开发者的帽子。Peter 是正在处理其触发器的保险会员的家庭医生。Peter 已经在他的办公电脑或智能手机或平板电脑上登录了该应用程序。您需要通过 REST API 了解 Peter 是[潜在所有者的任务:](https://gist.github.com/mauriziocarioli/79db8bd88e95deede5bc1ea966359f46)

```
curl --user Peter:secret \
--location --request GET 'http://localhost:8080/kie-server/services/rest/server/containers/PHM-Processes/forms/tasks/1?lang=en&type=ANY&marshallContent=true' \
--header 'Accept: application/json'

```

因为您正在构建一个定制的 UI(或者与一个已经存在的 UI 集成)，所以您不需要响应中的大部分信息[。您需要的只是表单提交的变量列表:](https://gist.github.com/mauriziocarioli/dac82e7f205c0428442ea934ae4c6e05)

```
            "properties": [
                {
                    "name": "answer",
                    "typeInfo": {
                        "type": "BASE",
                        "className": "java.lang.String",
                        "multiple": false
                    },
                    "metaData": {
                        "entries": []
                    }
                },
                {
                    "name": "na",
                    "typeInfo": {
                        "type": "BASE",
                        "className": "java.lang.Boolean",
                        "multiple": false
                    },
                    "metaData": {
                        "entries": []
                    }
                },
                {
                    "name": "naText",
                    "typeInfo": {
                        "type": "BASE",
                        "className": "java.lang.String",
                        "multiple": false
                    },
                    "metaData": {
                        "entries": []
                    }
                }
            ]

```

然后，您可以提交变量值，首先用 [this REST API](https://gist.github.com/mauriziocarioli/f990c6d2cdf92a9d2b51f9d1dce79f40) 定义任务的输出，这里(作为一个例子)您只提交答案变量:

```
curl --user Peter:secret \
--location --request PUT 'http://localhost:8080/kie-server/services/rest/server/containers/PHM-Processes/tasks/1/contents/output' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data-raw '{
	"answer": "Hello."
}'

```

然后用[REST API](https://gist.github.com/mauriziocarioli/2c16976090e6c47f23edf3f8837d73c2)完成任务:

```
curl --user Peter:secret \
--location --request PUT 'http://localhost:8080/kie-server/services/rest/server/containers/PHM-Processes/tasks/1/states/completed' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data-raw ''

```

彼得的任务需要硬关闭。也就是说，根据我们的客户需求，仅仅完成 BPM 系统中的任务并不足以认为任务已经完成。必须收到来自外部系统的信号才能实际关闭任务。必须对外部系统进行编程，以使用[以下 REST API](https://gist.github.com/mauriziocarioli/659d4de7c3a48a48712c2285631b9258) 发送适当的信号:

```
curl --user externalsystem:secret \
--location --request POST 'http://localhost:8080/kie-server/services/rest/server/containers/PHM-Processes/processes/instances/4/signal/hard_close' \
--header 'Content-Type: application/json' \
--data-raw '{}'

```

硬关闭 Peter 的任务将启动另外两个以 Peter 的任务为前置任务的任务流程。场景中的其他参与者是保险会员 Mary、药剂师 Robert 和保险渠道工作人员 Charlie。

你可以重复你为其他演员扮演彼得时所遵循的步骤。这里我就不为你描述这些步骤了。请记住，根据 Get the Data 服务提供的数据，只有 Charlie 的任务需要硬关闭。其他两个任务可以是软关闭的，它们不需要来自外部系统的信号就可以被认为是关闭的。

这就完成了“快乐路径场景”

## 提醒场景

在这个场景中，Charlie 未能在属性`Task`中指定的时间段内采取行动。结果会发送一封`reminderInitiation`和一封提醒邮件。事实上，电子邮件应该以属性`Task.reminderFrequency`给出的频率发送。获取数据服务为每个任务提供任务对象及其所有属性。任务完成后，电子邮件应该停止。

该电子邮件由自定义工作项目处理程序发送。在 PHM-Processes 项目的部署描述符中，您可以看到`EmailWorkItemHandler`的参数是系统环境变量:

这是因为出于安全原因，您不希望在那里写入电子邮件服务器凭据。您必须将系统环境变量`DEMO_SMTP_SERVER`、`DEMO_SMTP_PORT`、`DEMO_SMTP_USER`和`DEMO_SMTP_PWD`设置为您将使用的电子邮件服务器的相应值。例如，您可以使用一个模拟服务器，比如 [mailtrap.io](https://mailtrap.io/) 。

现在，转到提供模拟获取数据服务的 REST API 项目，打开`app.js`文件，为查理的任务找到数据，并将属性`reminderInitiation`和`reminderFrequency`的值更改为[下面的值](https://gist.github.com/mauriziocarioli/016cf21fee2a58f0c189d098c28d29cb):

```
    {
      task: {
        id: 58,
        origId: 'B143',
        suppressed: false,
        suppressionPeriod: '',
        expirationDate: '2020-12-31T12:00:00.000Z',
        close: 'HARD',
        reminderInitiation: 'PT60S',
        reminderFrequency: 'R/PT60S',
        escalationTimer: 'P90D',
        description: 'Getting Community Info'
      },
      assignment: {
        actor: 'Charlie',
        channel: 'CCN',
        escalationActor: 'Marc',
        escalationChannel: 'CCN'
      },
      reminder: {
        address: 'charlie@healthinsurance.com',
        body: 'XYZ',
        from: 'PHM@healthinsurance.com',
        subject: 'Reminder'
      }
    }

```

初始时间段现在是 60 秒而不是 15 天，频率是 60 秒而不是 15 天。这种配置将使测试提醒正在被发送变得更加容易。如果你不喜欢这些价值观，就用适合你的吧。

几分钟后，Charlie 的电子邮件收件箱将充满提醒消息，如图 5 所示。

[![Charlie's email inbox](img/73352fedb3db50f665ad607f10c6f669.png "2020-03-01_09-30-56")](/sites/default/files/blog/2020/03/2020-03-01_09-30-56.png)Figure 5: Charlie's email inbox.

Figure 5: The results in Charlie's email inbox.

现在让 Charlie 完成任务，并检查电子邮件弹幕是否已经停止。当然，您也可以关注在**商业中心**中发生的事情，更新 Charlie 的任务和提醒子流程图。

## 升级场景

现在，您应该知道如何运行这个场景。只需将升级计时器值更改为 60 秒，看看会发生什么。请记住，这种情况依赖于任务硬关闭，而不像前一种情况那样依赖于任务完成。

## 结论

到目前为止，希望您已经对示例业务流程的工作原理有了深入的了解，学会了如何验证您的实现是否正确，以及如何连接由业务流程驱动的业务应用程序的用户界面。

*Last updated: June 29, 2020*