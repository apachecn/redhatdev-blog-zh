# 用 Apache Camel K 和 Visual Studio 代码发送电报

> 原文：<https://developers.redhat.com/blog/2019/09/30/sending-a-telegram-with-apache-camel-k-and-visual-studio-code>

几个月前，当我被介绍给 [Apache Camel K](https://camel.apache.org/camel-k/latest/index.html) 时，我惊讶于开发人员可以如此快速地在 [Kubernetes](https://developers.redhat.com/developer-tools/kubernetes) 上编写和部署基于 [Apache Camel](https://camel.apache.org/) 的集成。我们立即开始创建基于 Microsoft Visual Studio (VS)代码的工具，让事情变得更加简单。

什么是骆驼 K？这是一个轻量级集成框架，构建自 Apache Camel，专为在 Kubernetes 上本地运行的无服务器/微系统世界而设计。它允许开发人员用他们最喜欢的 Camel DSL 编写集成，并在 Kubernetes 或 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 上快速部署。您甚至可以用 Groovy 或 JavaScript 等轻量级语言编写集成。

我们已经使用[语言服务器协议(LSP)](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-apache-camel) 在 VS 代码中为 Apache Camel 构建了语言支持，在 XML 和 Java 中为 Camel 组件 URIs 提供了自动完成功能。最近，我们开始增加对 Groovy、JavaScript、YAML 和 Kotlin 的支持。(详见 [Apache Camel LSP 客户端项目](https://github.com/camel-tooling/camel-lsp-client-vscode)。)

现在，通过 Red Hat extension 为 Apache Camel K 提供新的[工具，我们在您的 IDE 中增加了对 Camel K 的支持。为了说明工具的作用，让我们从一个简单的用户故事开始，这个故事的灵感来自 Nicola Ferraro 几年前写的一篇文章(](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-camelk)[用 Apache Camel](https://www.nicolaferraro.me/2016/05/27/creating-a-telegram-bot-in-5-minutes-with-apache-camel/) 在 5 分钟内创建了一个电报机器人)。

*   作为一名开发人员，每当有人提到#camelkrocks 标签时，我都想给自己发一份电报通知，这样我就可以更好地参与我们的社交媒体社区。

首先，让我们假设您有以下内容:

*   电报账户和电报信息客户端访问或通过 https://web.telegram.org/.访问
*   Apache Camel K 运行在可访问的 Kubernetes 实例中，本地安装了 Kubernetes 和 Camel K 命令行工具。
*   安装了 Apache Camel K by Red Hat 扩展工具的 Microsoft Visual Studio 代码在本地运行。

在你创建电报机器人的骆驼 K 面之前，你必须在电报上创建你的机器人。Nicola 在他的文章中详细描述了这些步骤[；参见创建电报机器人的部分，给你的机器人一个独特的名字，比如*camelk-hashtag-Telegram-bot*。一定要记下@BotFather 给出的授权令牌。](https://www.nicolaferraro.me/2016/05/27/creating-a-telegram-bot-in-5-minutes-with-apache-camel/)

现在在 VS 代码中，用一个名为`hashtagbot`的空目录创建一个新的工作区。在`hashtagbot`目录中，创建一个名为`hashtagbot.groovy`的文件，并将该文本复制到该文件中:

```
from("telegram:bots/replace-me-with-your-telegram-token")
    .choice()
        .when()
            .simple('${in.body} != null')
            .to("direct:response1")

from("direct:response1")
    .log('Incoming message from Telegram is ${in.body}')
    .setBody()
        .simple('You said \"${in.body}\"?')
        .to("telegram:bots/replace-me-with-your-telegram-token")
```

**注:**记得把“用你的电报令牌替换我”换成@BotFather 给你的电报授权令牌。

保存文件后，右键单击它并从弹出菜单中选择*启动 Apache Camel K 集成*。命令选项板中会出现一个下拉菜单，其中包含几个选项。选择*开发模式——开发模式下的 Apache Camel K 集成*。

![](img/721689533e94b8e525fef4e61e0a49d9.png)

当您的集成开始运行时，观察出现的 Apache Camel K 输出通道:

![](img/edbf61d7d09ab8e346976a3bfb577089.png)

在你的 Telegram 应用中，找到你的机器人(使用你在@BotFather 中创建机器人时指定的名称)，开始新的对话。你在电报里打什么都要用“你说了”<whatyoutyped>？“举个例子:</whatyoutyped>

![](img/72d1e10ffb8817fb7b567787e9608903.png)

接下来，更新您的 route 来检查给定的字符串中是否包含我们的标签#camelkrocks，并相应地做出响应。

如下更改`hashtagbot.groovy`文件中的第二条路线:

```
from("direct:response1")
    .log('Incoming message from Telegram is ${in.body}')
        .choice() .when(simple('${bodyAs(java.lang.String)} contains "#camelkrocks"')) .setBody(simple('Did somebody say #camelrocks? Of course it does!'))         .to("telegram:bots/replace-me-with-your-telegram-token")
                .end()
```

一旦您保存了您的路线，它就会更新已部署的集成，您可以尝试一下。例如:

![](img/663e081ab6195f25ea7da693904a0e2b.png)

虽然这是一个非常简单的例子，但是您可以使用这种方法作为基础来做一些事情，比如将消息路由到数据库或电子邮件地址，以便以后进行跟踪。

我们还有不同的部署选项，让您不必在路线正文中包含电报令牌。

一种方法是使用 Apache Camel K 的“property”选项。

1.  在`hashtagbot.groovy`文件中，通过用`{{telegram-token}}`替换您的令牌来编辑路线。
2.  当您开始集成时，通过在下拉菜单中选择*属性-Apache Camel K Integration with Property*选项来部署路由。
3.  指定一个名为`telegram-token`且值为的属性作为您的电报认证令牌。运行时将用您的令牌值替换`{{telegram-token}}` 。

另一种方法是使用 Apache Camel K ConfigMap 或 Secret 方法。使用这种方法，您的代币价值在 Kubernetes 系统中是安全的。

例如，使用秘密方法:

1.  创建一个新的`application.properties`文件，并保存到您的`hashtagbot`文件夹中。
2.  在`application.properties`文件中，添加下面一行(用您的实际令牌替换该值):
    `**telegram-token=replace-me-with-your-telegram-token**`
3.  右击`application.properties`文件，然后选择*从文件*创建 Kubernetes 秘密。
4.  将新的秘密命名为`telegram-secret`。
5.  在`hashtagbot.groovy`文件中，通过用`{{telegram-token}}`替换您的令牌来编辑路线。
6.  当您开始集成时，通过在下拉菜单中选择*Secret-Apache Camel K Integration with Kubernetes Secret*选项来部署路线。
7.  在出现的下拉列表中，指定您创建的`telegram-secret`。Camel K 运行时将用您的令牌值替换`{{telegram-token}}` 。

要了解更多信息，请查看最新的 Apache Camel K 并查看微软 VS 代码的红帽扩展 Apache Camel K 的[工具。](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-camelk)

编码快乐！

*Last updated: July 1, 2020*