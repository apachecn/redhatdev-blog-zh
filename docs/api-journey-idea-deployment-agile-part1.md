# API 之旅:以敏捷的方式从想法到部署——第一部分

> 原文：<https://developers.redhat.com/blog/2018/04/11/api-journey-idea-deployment-agile-part1>

这一系列文章的目标是描述一个提议的**敏捷 API 交付过程**的方法。它不仅包括开发部分，还包括设计、测试、交付和生产管理。您将学习如何使用 mocking 来加速开发和打破依赖性，使用契约优先的方法来定义将强化您的实现的测试，通过管理网关来保护公开的 API，以及最后，使用 CI/CD 管道来保护交付。

我和 Nicolas Massé合著了这个系列，他也是一名红帽匠。这个系列基于我们自己的真实经历，来自我们与遇到的 Red Hat 客户的合作，以及我之前在一家大型保险公司担任 SOA 架构师的经历。该系列是我们在研讨会或活动(如 APIdays)期间运行的典型用例的翻译。

## 背景

随着 IT 成为企业保持快速创新的核心竞争力，大多数公司正在转变为软件公司。他们正在重新思考构建和运行 IT 的方式，为即将到来的新数字服务的爆发做准备。这种趋势导致了现代软件架构范例，如 API 和微服务。

由于 API 管理解决方案正在成为主流，向世界安全地公开 API 变得越来越容易。然而，这样做并不是一个完整的解决方案。为了保持相关性，整个 API 生命周期也应该变得敏捷。矛盾的是，这很难做到，因为新的基于服务的架构使得依赖性激增。

因此，**是时候考虑一种交付 API 的新方法了——包括模拟和测试——以简化和加速由微服务支持的生产就绪 API 的交付**。

## 具体用例

在我们深入研究之前，让我们先来看一个用例，它将有助于通过一个例子来说明这种方法:ACME Inc .的用例，一家当地的啤酒厂。ACME 生产、储存啤酒，并将其分销给其钟爱的客户，整个过程由内部管理。

[![Acme Inc. situation](img/c7500a38fa6b3efbc598928f06dec9f7.png "acme-inc-situation")](/sites/default/files/blog/2018/04/acme-inc-situation.png)(Creative Commons licensed icons by Laymik from Noun Project)">

竞争的加剧和客户群需求的增长迫使 ACME 重新思考其分销模式。也就是说，分销将留给独立的经销商，他们可以在当地、网上或现场销售啤酒。主要的挑战是如何开放信息系统，以便独立经销商可以发现啤酒目录，检查库存，等等。当然，这可以通过公开一个 API 来实现。

[![Acme Inc. target](img/21745be27fff34210e3544f30317703a.png "acme-inc-target")](/sites/default/files/blog/2018/04/acme-inc-target.png)(Creative Commons licensed icons by Laymik from Noun Project)">

## 技术和材料

我们将在 之旅中使用的技术有:

*   API 设计工具( [Apicurio 工作室](http://www.apicur.io/) )
*   嘲讽和测试工具([](http://microcks.github.io/))
*   API 测试和编辑工具( [邮差](https://www.getpostman.com/) )
*   服务开发框架([【Spring Boot】](https://projects.spring.io/spring-boot/))
*   部署/CI-CD 平台([Kubernetes](https://kubernetes.io/)/[open shift](https://www.openshift.com/))
*   API 管理工具([3 标度](https://www.3scale.net/) )

您可能只想阅读本系列，但如果您想进一步重放整个用例演示，所有相关材料(源代码、脚本、设置过程)都可以在 GitHub 资源库中找到:[https://GitHub . com/microcks/API-life cycle/tree/master/beer-catalog-demo](https://github.com/microcks/api-lifecycle/tree/master/beer-catalog-demo)。

## 里程碑 0: API 构思

虽然大多数 API 生命周期定义都是从设计阶段开始的，并且最佳实践是契约优先的方法，但是我们都知道从零开始 API 契约设计是非常困难的。有一个沙箱来测试未来的 API 应该是什么样子通常是有帮助的。沙箱允许我们播放和演示 API 方法和资源。它应该允许我们快速测试和共享 API 的不同设计。

![API ideation stage](img/fb51f292842590539bbff019ec0ae918.png)

我们可以看到关于此阶段工具的不同方法。

“本地”方法包括使用专用于模拟的本地工具。一些工具如 [Hoverfly](https://hoverfly.io/) 在这方面大放异彩，因为它们能够使用本地 JSON 文件定义来模拟 API。然而，这些工具主要是针对开发人员的，它们的“本地”特性使得在中长期内很难轻松地共享和测试多个设计。

我们更喜欢“团队”方式，我们认为这种方式在企业环境中更有意义。一个现成的平台可以让我们主持和分享不同的测试。这也是 [Microcks](http://microcks.github.io/) 这款用于嘲讽和测试的通信和运行时工具的用途之一。Microcks 可以很容易地在 Kubernetes 或 OpenShift 上设置，并提供一个名为*动态服务*的“后端即服务”功能。

只要给它你的 API 的名字和版本，它就会为你生成一个基本的 CRUD REST API。对于我们的极致用例，我们将在版本 `0.1` 上创建 `Beer Catalog API` 来允许我们玩资源。

![Microcks generic service creation](img/8e73e81bd250d773c8c2fb7e0e0a54c5.png)

新 API 的动态端点立即可供您使用。动态服务的详细信息页面为您提供了有关可用操作以及端点 URL 的信息。

![Microcks generic API visualization](img/90e1a087db73e5ff6b43035fe5aa52e1.png)

您现在可以开始使用这个示例 API，并通过将模拟 URL 附加到 Microcks 基本 URL : 来记录沙盒中新的 啤酒 资源

```
$ curl -X POST 'http://microcks.example.com/dynarest/Beer%20Catalog%20API/0.1/beer' -H 'Content-type: application/json' -d '{"name": "Rodenbach", "country": "Belgium", "type": "Brown ale", "rating": 4.2}'

$ curl -X POST 'http://microcks.example.com/dynarest/Beer%20Catalog%20API/0.1/beer' -H 'Content-type: application/json' -d '{"name": "Westmalle Triple", "country": "Belgium", "type": "Trappist", "rating": 3.8}'

$ curl -X POST 'http://microcks.example.com/dynarest/Beer%20Catalog%20API/0.1/beer' -H 'Content-type: application/json' -d '{"name": "Weissbier", "country": "Germany", "type": "Wheat", "rating": 4.1}'
```

您可以从详细信息页面查看已创建的资源。现在，每个可以访问 Microcks 的人都可以查看您的示例 API 和记录的资源。

![Microcks generic API resources](img/661b756614b4b6686fbf1367bad598d8.png)

你也可以使用不同的方法来查询资源。

```
$ curl 'http://microcks.example.com/dynarest/Beer%20Catalog%20API/0.1/beer/'
[{ "name" : "Rodenbach", "country" : "Belgium", "type" : "Brown ale", "rating" : 4.2, "id" : "5aa14cef6ba84900019abe9d" }, { "name" : "Westmalle Triple", "country" : "Belgium", "type" : "Trappist", "rating" : 3.8, "id" : "5aa14cf66ba84900019abe9f" }, { "name" : "Weissbier", "country" : "Germany", "type" : "Wheat", "rating" : 4.1, "id" : "5aa14cfc6ba84900019abea0" }]

$ curl 'http://microcks.example.com/dynarest/Beer%20Catalog%20API/0.1/beer/5aa14cfc6ba84900019abea0'
{ "name" : "Weissbier", "country" : "Germany", "type" : "Wheat", "rating" : 4.1, "id" : "5aa14cfc6ba84900019abea0" }

$ curl 'http://microcks.example.com/dynarest/Beer%20Catalog%20API/0.1/beer' -H 'Content-type: application/json' -d '{"country": "Belgium"}'
[{ "name" : "Rodenbach", "country" : "Belgium", "type" : "Brown ale", "rating" : 4.2, "id" : "5aa14cef6ba84900019abe9d" }, { "name" : "Westmalle Triple", "country" : "Belgium", "type" : "Trappist", "rating" : 3.8, "id" : "5aa14cf66ba84900019abe9f" }]
```

Microcks 为您提供了一个完整的沙盒，用于迭代、测试不同的资源表示、共享它们，最重要的是，允许它们与一些消费者应用程序集成以进行真实测试。沙盒还为接下来的设计阶段提供了有用的资源示例。

## 里程碑 1: API 合同设计

这个阶段的目的是创建一个 API 契约工件，涵盖未来 API 的技术和语法定义。契约提供了对 API 方法和所操作的定制资源的清晰描述。它代表了基于服务的体系结构的基石，我们将在本系列的后面看到它如何帮助加快速度和评估未来的实现。

过去几年中不同的标准化尝试最终使得 OpenAPI 规范(以前的 Swagger 规范)成为使用的标准。它将 YAML 和 JSON 作为指定合同的事实语言。

![API design stage](img/6b17fba50a7d82feba60fe3717d8a0cd.png)

同样，你可以选择不同的工具路径。在设计 API 契约时，我们仍然更喜欢支持协作实践的“团队”方法。此外，我们发现团队方法有助于构建一个企业可能管理的几十个 API 契约的存储库。

[API curio Studio](http://www.apicur.io/)提供 API 合同在线协同编辑器。这是一个 WYSIWYG 编辑器，它通过提供关于遵守 OpenAPI 规范的即时反馈来简化 API 设计。

![APIcurio dashboard](img/1d3ae0cf0d3c24cea313c0db53162100.png)

Apicurio Studio 提供了一个仪表板和工具，用于通过标签浏览 API，导入或创建全新的 API 定义。对于我们的 ACME 用例，我们将使用版本 `0.9` 创建一个新的 `Beer Catalog API` 。由于我们之前的沙盒测试，ACME 能够精确地定义它的需求，并且只需要 API 方法来浏览啤酒目录。

![APIcurio API details](img/542afc2b2c39c368f23ec704c838a286.png)

ACME 还能够详细定义 `beer` 资源:标识数据模型的强制和可选属性。

![APIcurio beer resource](img/084ab2e665d418c50941d3d32cbbf5a8.png)

所有这些编辑操作都可以通过动态验证轻松完成。然后一切都可以保存并版本化到 Git 存储库中。如果你想看看最终的结果，可以查看我们的文案:[https://github . com/microcks/API-life cycle/blob/master/beer-catalog-demo/API-contracts/beer-catalog-API-swagger . JSON](https://github.com/microcks/api-lifecycle/blob/master/beer-catalog-demo/api-contracts/beer-catalog-api-swagger.json)

## 里程碑 2 期望和请求/响应样本

在构建基于服务的架构应用程序时，这一阶段通常会被忽视或忽略。人们经常犯的一个错误是没有花一点时间就开始实现。然而——这在另一篇博文 中 [介绍过——服务契约化采样是一个战略步骤，因为它允许您并行开发服务的提供者和消费者，并且它支持您的 API 的高效契约测试。](https://blog.openshift.com/mocking-microservices-made-easy-microcks/)

![API sampling stage](img/1392f5f10bef11699efcb0fdec1dd0ab.png)

虽然从合同中模仿在技术上是可行的，但我们认为最好使用有代表性的样本来获得最大的业务逻辑。请求/响应示例专业地说明了未来 API 的常见和边缘情况。

此阶段还应致力于明确定义仅使用技术合同无法描述的业务期望。我们可以很容易地使用采样来设置关于传入请求的预期响应的显式断言。

这种规则的典型例子是过滤查询。假设您正在提供一个 API，它支持使用过滤器搜索项目；所有响应项都应该将它们的值指定为标准。这听起来似乎是显而易见的，但在现实中，实现可能很容易无法满足这一期望。金融服务中的另一个典型用例是订阅或购买产品的最低年龄要求。使用抽样和期望，如果没有达到最小年龄，描述用例以及应该出现的消息是非常容易的。

一个简单而高效的工具就是 [邮递员](https://www.getpostman.com/) 。这是因为我们正在处理一个现代的 REST API，但是通过使用[SOAP ui](https://www.soapui.org/)工具，同样的原理也适用于遗留的 SOAP web 服务。使用 Postman，我们将能够描述真实世界的请求和响应，然后将任何内容保存为一个名为*集合*的 JSON 文件。

例如，对于我们的 ACME 用例，我们首先导入之前用 Apicurio 创建的 OpenAPI 规范。然后，很容易添加一些例子来指定对“ `Get beers having the available status` ”请求的响应应该是什么样子。

![Postman samples](img/d3c84f2c326272c97c6128d4fca21cca.png)

我们还可以使用 Postman 的测试来指定期望值。测试不是专门为样品定义的，而是在方法水平上全局定义的。因此，它浓缩了一个方法的所有公共、边缘和异常情况。测试是使用 JavaScript 代码片段指定的。

在我们的 ACME 示例中，我们想要检查我们上面描述的过滤查询案例。这可以通过定义一个新的内联模式定义来轻松实现，该定义将 `status` 属性的值限制为预期的值。

```
var expectedStatus = globals["status"];
var jsonData = JSON.parse(responseBody);

var schema = {
 "type": "array",
 "items": {
   "type": "object",
   "properties": {
      "name": { "type": "string" },
      "country": { "type": "string" },
      "type": { "type": "string" },
      "rating": { "type": "number" },
      "status": { "type": "string", "enum": [expectedStatus] }
   }
 }
};

tests["Valid status in response"] = tv4.validate(jsonData, schema); 
```

所有这些信息都保存在集合中，以后可以在 Git 中导出和版本化。将所有东西版本化绝对是一个好习惯。它还帮助其他人以 [消费者驱动契约](https://martinfowler.com/articles/consumerDrivenContracts.html) 的方式贡献样本数据和期望。

## 关键要点

到目前为止，我们已经看到 ACME 是如何通过前三个阶段开始其 API 之旅的:

*   使用沙盒来满足他们需求的想法。
*   指定 API 的方法、文档和数据结构的合同设计。
*   抽样是一个额外的步骤，允许交流现实生活中的样品和期望。我们将会看到它也是并行开发和自动化测试的推动者。

请继续关注第二部分，在这里您将学习如何从已定义的示例中部署一个 mock，以及更多内容。

* * *

### 链接

*   第二部分——API 模拟和即用测试；开发、部署和测试
*   第三部分——向外界公开 API 端到端流；API 生命周期

*Last updated: September 3, 2019*