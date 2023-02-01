# 红帽峰会:通过 OpenWhisk 和 OpenShift 实现服务功能

> 原文：<https://developers.redhat.com/blog/2018/05/16/summit-faas-openwhisk-openshift>

*无服务器计算*(通常称为功能即服务，或 FaaS)是当今最热门的新兴技术之一。[open whisk 项目](http://openwhisk.apache.org)，目前正在 Apache 孵化，是 FaaS 的一个开源实现，让你创建响应事件时调用的函数。我们自己的 Brendan McAdams 做了一个演示，解释了无服务器的基础，open whish 项目如何工作，以及如何在 OpenShift 中运行 open whish。

Brendan 概述了无服务器/ FaaS 平台的三个特性:

1.  它通过调用函数来响应事件
2.  函数是按需加载和执行的
3.  函数可以与来自 FaaS 平台本身外部的触发事件链接在一起。

在我们继续之前，有一个术语注释。“功能即服务”和“无服务器”通常可以互换使用。然而，最近人们也用“无服务器”这个词来表示“任何不需要你启动和管理虚拟机的东西”明确地说，我们在这里谈论的是 FaaS。

引用项目网站上 OpenWhisk 的官方定义很有用:

> ...[A]无服务器的开源云平台，可执行功能以响应任何规模的事件。

OpenWhisk 的事件驱动特性非常强大，但是它的伸缩能力使得某些应用程序第一次具有了经济可行性。如果没有 FaaS 平台，您将不得不提供必要的资源来处理应用程序可能遇到的任何负载。在 FaaS，你不需要提供任何东西。你只需告诉系统，“这里有一些可能发生的事情，这里是当它们发生时你应该运行的代码。”应用程序的架构是以声明的方式定义的，FaaS 提供者必须提供资源来处理事件。

FaaS 的另一个强大之处在于事件可以来自任何地方，包括 FaaS 平台之外的来源。例如，对数据库的更改可能会生成一个事件。OpenWhisk 还允许你创建自己的活动。Red Hat 正在开发一个 AMQP 事件提供程序，其他人已经编写了代码，可以从 git 提交或向 Slack 通道发送的内容中生成事件。最后，因为 OpenWhisk 中的所有函数都有一个 REST API，所以它们也可以被另一段代码或从`wsk`命令行工具直接调用。Brendan 的演示广泛使用了命令行。

*触发器*是一类事件，例如对数据库所做的所有更改。一旦定义了触发器，您就可以创建*规则*来确定当事件发生时应该调用哪个(哪些)函数。在 OpenWhisk 术语中，函数被称为*动作*。项目网站上的高级编程模型讨论中解释了所有的术语。为了简单起见，我们将继续称函数为函数。

OpenWhisk 世界中的通用数据格式是 JSON，最受支持的语言是 Java、Node.js 和 Python。对于这三种语言，您将分别使用 [GSON 库](https://github.com/google/gson)、原生 JSON 和 [Python 字典](https://docs.python.org/3/library/stdtypes.html#typesmapping)。如您所料，对其他语言的支持正在积极开发中，包括 Swift 和 PHP。

重要的是要记住你的函数是无状态的。给它们一些由事件生成的数据，然后它们处理这些数据并返回结果。如果你再次调用这个函数，它不知道之前发生了什么。如果同一个功能的多个副本同时运行，它们彼此之间就没有任何了解。

如果你熟悉 [Unix 管道](https://en.wikipedia.org/wiki/Pipeline_(Unix))，你会马上理解*序列*的重要性。您可以通过组合一系列函数来创建一个新函数，这些函数应该在响应事件时被调用。第一个函数的输出成为第二个函数的输入，第二个函数的输出成为第三个函数的输入，依此类推。在 Brendan 的例子中，第一个函数接受一个名字(`{"name": "Brandon McAdams"}`)，并返回该名字的反向版本(`{"name": "McAdams, Brandon"}`)。该 JSON 然后被传递给 Hello World 函数，该函数返回一个问候语，其名称与第一个函数返回的名称相反(`{"greeting": "Hello, McAdams, Brandon"}`)。请记住，序列中的每个函数都需要理解前一个函数的 JSON。如果您向示例序列添加了第三个函数，它将需要查找名为`greeting`的参数。

整个编程模型非常灵活。为了响应事件，系统可以是单个功能、一系列功能或多个功能。

正如你所期待的，Brandon 讨论了如何在 OpenShift 中运行 OpenWhisk。我们有[一个 GitHub repo，其中包含将 OpenWhisk 部署到 OpenShift 项目](https://github.com/projectodd/openwhisk-openshift)所需的模板和 Docker 映像。按照说明操作，您应该可以开始运行自己的 FaaS 平台了。

对于任何现代开发人员来说，学习如何用一组无服务器函数构建有用的应用程序是一项至关重要的技能。看看我们的 GitHub repo，今天就开始吧！

[https://www.youtube.com/embed/C2u6wVRI-N0](https://www.youtube.com/embed/C2u6wVRI-N0)

* * *

¹ 举一个容器世界的具体例子，想象一个 Kubernetes 环境，其中一个节点实际上是一个云，它自动为集群中的主机 pod 供应和取消供应虚拟机。你可能会认为“无服务器计算”一词描述了这种情况，因为你没有使用虚拟机，但这不是我们要讨论的。我们坚定地从根本上关注这里的功能。

*Last updated: September 3, 2019*