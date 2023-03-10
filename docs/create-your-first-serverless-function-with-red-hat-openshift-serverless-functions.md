# 用 Red Hat OpenShift 无服务器函数创建您的第一个无服务器函数

> 原文：<https://developers.redhat.com/blog/2021/01/04/create-your-first-serverless-function-with-red-hat-openshift-serverless-functions>

*无服务器*是一个强大而流行的范例，您不必担心管理和维护您的应用基础设施。在无服务器环境中，*功能*是由开发人员创建的一段专用代码，但由托管基础设施运行和监控。无服务器功能的价值在于它的简单和快捷，这甚至可以吸引那些不认为自己是开发人员的人。

本文向您介绍 Red Hat OpenShift 无服务器函数，这是 [Red Hat OpenShift 无服务器 1.11](https://www.redhat.com/en/blog/introducing-using-openshift-serverless-event-driven-applications) 中的一个新的开发者预览功能。我将提供一个概述，然后展示两个用 Node.js 演示无服务器功能的示例应用程序。

## OpenShift 无服务器功能

[Red Hat OpenShift 无服务器](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/serverless_applications/serverless-getting-started?extIdCarryOver=true&sc_cid=701f2000000u72fAAA#knative-eventing_serverless-getting-started)利用 [Knative](https://developers.redhat.com/topics/serverless-architecture) 的力量来交付无服务器、事件驱动的应用程序，这些应用程序可以按需扩展。随着 [OpenShift 无服务器 1.11](https://www.redhat.com/en/blog/introducing-using-openshift-serverless-event-driven-applications) 发布，我们增加了新的无服务器功能特性，目前作为开发者预览版提供。无服务器功能附带预定义的模板和运行时，并提供本地开发人员体验。总之，这些特性使得创建无服务器应用程序变得非常容易。

### 如何获得无服务器功能

无服务器功能与 OpenShift 无服务器命令行界面(CLI)，`kn`捆绑在一起。当您使用 OpenShift 无服务器操作符进行安装时，OpenShift 无服务器会自动部署，并在 OpenShift 上进行管理。您可以使用以下命令访问无服务器功能:

```
$ kn func

```

**注**:安装说明见 [OpenShift 无服务器文档](https://docs.openshift.com/container-platform/4.10/serverless/functions/serverless-functions-getting-started.html)。

### 包括什么？

无服务器函数带有流行语言的预定义运行时，如 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 、 [Node.js](https://developers.redhat.com/blog/category/node-js/) 和 [Go](https://developers.redhat.com/blog/category/go/) 。这些运行时基于[云本地构建包](https://buildpacks.io/)。选择运行时后，无服务器功能会创建适当的项目框架，以便您可以专注于编写业务逻辑。无服务器功能还包括本地开发者体验，以支持迭代开发和测试的快速[内循环](https://developers.redhat.com/search?t=inner+loop)。

### 调用无服务器功能

您可以使用普通 HTTP 请求调用无服务器功能，或者使用 OpenShift 无服务器事件组件调用 [CloudEvents](https://cloudevents.io) 。OpenShift 无服务器函数附带了现成的项目模板，可以快速启动 HTTP 和 CloudEvents 触发器类型的代码。

接下来，我们将探索两个例子。对于第一个例子，我们将为 HTTP 请求配置无服务器功能。对于第二个例子，我们将使用 CloudEvents。请使用无服务器功能[快速入门文档](https://openshift-knative.github.io/docs/docs/functions/quickstart-functions.html)来确保您已经安装了示例先决条件。

## 示例 1:为 HTTP 请求创建一个无服务器函数

安装了必备组件后，为无服务器功能创建一个新目录。进入目录后，执行以下命令创建并部署新的无服务器功能:

```
$  kn func create 

```

默认情况下，该函数使用一个用于普通 HTTP 请求的项目模板进行初始化。您可以通过输入`Node.js`、`Quarkus`或`Go`作为`-l`标志的值来选择您的编程语言。如果没有提供带有`-l`标志的运行时，默认运行时是 Node.js。

**注意**:您可以使用`-c`标志来提示 CLI 引导您通过交互式开发者体验创建您的第一个函数，它会提示您添加语言和事件值。键入`-help`随时寻求帮助。

### Node.js 运行时

默认情况下，输入命令`$ kn func create`会为一个由普通 HTTP 请求触发的函数创建脚手架。我们默认 Node.js 运行时的脚手架包括`index.js`、`package.json`和`func.yaml`文件。我们可以扩展`index.js`基础代码来开发我们的无服务器功能。

首先，让我们在提供的`handleGet(context)`方法中添加一个返回消息`Greeting <username>`。图 1 显示了`index.js`中的`handleGet`功能。

[![function handleGet from the index.js file. This function accepts the context object. and return the word"Greetings" with the name provided by the Get method. If no name was provided, this functions use the word stranger instead of the provided name.](img/32ec65212208d731586ff1429866bc34.png)](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-18-at-2.32.33-PM.png)Figure 1: This figure is the screenshot of function handleGet(context) from index.js file of the Serverless Function.

Figure 1: The handleGet(context) function.

### 部署功能

接下来，我们将把这个功能部署到 OpenShift 集群中。确保从本地环境登录到 OpenShift 集群，然后键入以下带有项目名称或集群名称空间的命令:

```
$ kn func deploy  -n <namespace>

```

请记住，您可以使用`-c`标志来获得互动体验。

无服务器功能将提示您提供一个容器注册表，将结果图像上传到其中。DockerHub 是默认的注册表，但是您可以使用任何公共图像注册表。

现在，转到 OpenShift 开发人员控制台中的拓扑视图。您将看到您的功能被部署为一个 Knative 服务，如图 2 所示。

[![Serverless Function deployed on OpenShift Cluster and the Routes highlighted that can be used to access the deployed function from the browser.](img/85670f3ae4bfc4a77a50cb27f8a829eb.png "Figure 2")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-10-at-11.30.15-PM.png)Figure illustrating the deployed Serverless Function on OpenShift cluster.

Figure 2: View the deployed serverless function on your OpenShift cluster.

### 测试功能

我们可以使用图 2 所示的 routes URL 来测试我们部署的无服务器功能。输入以下命令从集群中删除该功能:

```
$ kn func delete

```

对于本地开发者体验，我们可以使用标准语言工具或者在本地运行的容器中测试无服务器功能。在`kn`命令行上使用以下命令来构建容器映像:

```
$ kn func build

```

要在本地环境中测试构建的图像容器，请输入:

```
$ kn func run 

```

使用`curl`命令测试您部署的映像:

```
$ curl ‘https://localhost:8080/?name=Universe’

```

您也可以使用浏览器来查看结果，如图 3 所示。

[![localhost on port 8080 is being accessed with the name "Universe" as a data for the Get method of the deployed Serverless Function. Browser displays "Greetings Universe!"](img/e946913ad1c718e4231aed15aaeb733c.png "Figure 3")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-15-at-3.03.24-PM.png)Figure illustrating the deployed function being called from the browser

Figure 3: The deployed function being called from the browser.

## 示例 2:为 CloudEvents 创建一个无服务器函数

对于我们的第二个例子，我们将创建一个响应 CloudEvents 而不是 HTTP 请求的无服务器函数。在开始之前，请检查[快速入门文档](https://openshift-knative.github.io/docs/docs/functions/quickstart-functions.html)以确保您已经安装了本示例的先决条件。

### 创建一个新的无服务器功能项目

我们将使用之前创建新项目时使用的相同命令。然而，这一次我们将为`-t`标志提供一个`events`值。或者，我们可以使用`-c`标志进行交互提示。

```
$  kn func create -l <node|quarkus> -t  events  

```

为了接收 CloudEvents，我们将需要 Knative eventing 组件，所以接下来我们将设置它。

登录到 OpenShift 开发人员控制台，并导航到开发人员透视图。点击**添加**部分，查看图 4 中突出显示的**频道**标题。这个单幅图块创建一个默认通道。

[![Channel Tile is being highlighted under the Add section of OpenShift Developer Console.](img/41b4346e0943afe5364b7cc8e3997be0.png "Figure 4")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-11-at-12.02.52-AM.png)Figure illustrating highlighted in red "Channel" Tile on OpenShift Developer Console.

Figure 4: Locate the Channel tile in the OpenShift developer console (notice the tile in red).

现在，我们需要一个事件源。为此，我们将返回到**添加**部分，并单击图 5 所示的**事件源**标题。

[!["Event Soruce" tile is being shown under the Add section of OpenShift Developer Console.](img/47c49e47f7a0aa61aeb62285d10bafc2.png "Figure 5")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-10-at-11.47.22-PM.png)Figure illustrates "Event Source" tile highlighted in red box on OpenShift Developer Console

Figure 5: Locate the Event Source tile in the OpenShift developer console.

接下来，如图 6 所示，我们将选择并配置一个 ping 源作为我们部署的函数的事件源。注意， **Sink** 部分显示了部署的函数和我们刚刚创建的通道。对于本例，我们将选择通道作为事件源的接收器。

[![Sink options for the Ping Event source are being shown. Options are the previously created channel and the deployed OpenShift Serverless Function](img/310752329ab50966da7d9eac3bab27d8.png "Figure 6")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-11-at-12.19.32-AM.png)Figure highlighting the sink options for Event Source

Figure 6: Select Channel as the sink option for the ping event source.

创建事件源之后，我们可以在拓扑视图中查看所有组件，如图 7 所示。

[![OpenShift Serverless Function, created Event Source and Channel are beign shown in the topology view of OpenShift Developer Console.](img/6821de0954d6b7bfdac0807abc707748.png "Figure 7")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-11-at-12.23.14-AM.png)Figure illustrates all deployed components. Function, Even Source and Channel.

Figure 7: The deployed serverless function, event source, and channel.

要向已部署的功能添加触发器，请将鼠标悬停在通道上，然后单击并拖动蓝线将通道连接到功能。图 8 显示了拓扑视图中完整的部署细节。

[![Serverless Function connected to Event Source via Channel is being shown and Serverless Function Pod is active as it is receiving events.](img/d260a30ede0a77a12baacca897dde391.png "Figure 8")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-11-at-12.36.05-AM.png)Figure illustrating Serverless Function connected to Event Source via Channel.

Figure 8: Connect the serverless function to the event source via the channel.

当函数开始接收事件时，Knative 启动函数窗格，日志显示对该函数的调用。我们刚刚创建并部署了一个 OpenShift 无服务器功能。

## 展望未来

OpenShift 无服务器功能在 OpenShift 无服务器 1.11 中作为开发者预览版提供。它适用于所有 OpenShift 用户。我们将在未来几个月发布新功能，非常感谢您的反馈。

本文是介绍无服务器功能系列文章的第一篇。我的下一篇文章将向您介绍用 Quarkus 创建无服务器函数，quar kus 是超音速的亚原子 Java 运行时。同时，您可以通过阅读 [OpenShift 无服务器 1.11 发布公告](https://www.redhat.com/en/blog/introducing-using-openshift-serverless-event-driven-applications)、 [OpenShift 无服务器文档](https://docs.openshift.com/container-platform/4.6/serverless/serverless-getting-started.html)和 [OpenShift 无服务器功能文档](https://openshift-knative.github.io/docs/docs/functions/about-functions.html)来了解更多关于 OpenShift 无服务器功能的信息。

*Last updated: October 7, 2022*