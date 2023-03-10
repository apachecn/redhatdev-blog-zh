# 在 Red Hat CodeReady Workspaces 2 中编辑、调试和 GitHub

> 原文：<https://developers.redhat.com/blog/2020/01/23/editing-debugging-and-github-in-red-hat-codeready-workspaces-2>

在之前的一篇文章中，我展示了如何[启动 Red Hat code ready work spaces 2.0(CRW)并运行](https://developers.redhat.com/blog/?p=654277)一个可用的工作区。这一次，我们将经历编辑-调试-推送(到 GitHub)的循环。该演练将模拟真实的开发工作。

首先，您需要派生一个 GitHub 存储库。 [`Quote Of The Day`](https://github.com/redhat-developer-demos/qotd.git) repo 包含一个用 Go 编写的微服务，我们将在本文中使用它。如果您从未使用过 Go，请不要担心。这是一个简单的程序，我们只修改一行代码。

完成回购后，记下(或复制)你的 fork 的 URL。我们一会儿就会用到这些信息。

## 创建 Go 工作区

接下来，我们将使用预配置的 Go 工作区作为基础来创建我们在 CRW 的工作区。在启动工作空间之前，我们将对配置进行更改，删除默认的演示 GitHub 引用，代之以添加我们分叉的 repo 的 URL。这样做意味着我们的工作空间将打开，代码就在我们面前，可以编辑、调试和推回我们的 repo。

从**工作区**部分，选择**添加工作区**按钮。当你看到**新工作区**页面时，你会看到一个内置工作区列表。选择 **Go** 工作区。当您在该页面上时，也删除**示例**项目。这两个动作如图 1 所示:

[![Select the Go workspace and remove the example project.](img/7c7117fe08f45049a4aa2df28ad1f04b.png "new-go-workspace-1")](/sites/default/files/blog/2019/11/new-go-workspace-1.png)Figure 1: Select the correct language workspace and remove the **example** project.">

下一步是获得这个项目所需的源代码。为此，在**添加或导入项目**部分选择 **GitHub** 选项，连接到您的 GitHub 帐户，如图 2 所示:

[![Connect your GitHub account to a CRW workspace](img/189702df1d5386ecd4716875d0734b84.png "new-go-workspace-3")](/sites/default/files/blog/2019/11/new-go-workspace-3.png)Figure 2: Connect your GitHub account to the new workspace.">

连接后，您将看到您有权访问的每个 GitHub 项目的列表。请注意，该列表是按项目的字母顺序排列的。

选择 **qotd** 项目——您之前分叉的项目——并点击 **Add** 按钮将该项目添加到您的工作区定义中，如图 3 所示:

[![Select qotd and click Add](img/46b0994e5006882f2302bb69dda85538.png "new-go-workspace-4")](/sites/default/files/blog/2019/11/new-go-workspace-4.png)Figure 3: Add the **qotd** project to your workspace definition.">

您应该会看到类似于图 4 所示的结果:

[![Your new workspace definition](img/6359cc93ed7f45ec76e47bc29c2e0c2a.png "new-go-workspace-5")](/sites/default/files/blog/2019/11/new-go-workspace-5.png)Figure 4: Your new workspace definition.">

当你到达这一点时，点击**创建&打开**按钮(那个巨大的绿色按钮)让事情运转起来。

我们点火了。在这一点上，你会注意到一些非常酷的东西。CRW 在编辑器中自动打开 **README.md** 文件，如图 5 所示:

[![CRW editor with README.md displayed](img/ecfeccaab97d659c081f1a99dc1bcee4.png "qotd-1")](/sites/default/files/blog/2019/11/qotd-1.png)Figure 5: The editor opens your project.">

这是一件小事，但我认为这很好，因为这意味着你开始了解这个项目。干得好，[代码就绪工作区](https://developers.redhat.com/products/codeready-workspaces/overview)。

## 探索 CRW IDE

在继续之前，让我们标记 IDE 的重要部分:

*   文件浏览器，如前一节中的图 5 所示。
*   **调试**部分，如图 6 所示。
*   **源代码控制:GIT** 部分，如图 7 所示:

[![The CRW Debug section.](img/38dee9a9af38a2609511d72ac00da77b.png "debug-open")](/sites/default/files/blog/2019/11/debug-open.png)Figure 6: The CRW **Debug** section.">[![The CRW Source Control (GitHub) section.](img/f4eac9c8b5f8ac64d3e1d0f5055615b0.png "source-control-open")](/sites/default/files/blog/2019/11/source-control-open.png)Figure 7: The CRW Source Control (GitHub) section.">

在文件浏览器中单击一个文件即可进行编辑。在我们的例子中，我们将等到本文的后面进行编辑。现在，让我们运行程序，这带来了一个有趣的问题:调试。

## 调试您的代码

为了调试我们的代码，我们首先需要设置一个调试配置，然后设置一个断点。毕竟，你(我是说，*你的同事*)可能会引入一个错误。从顶部的菜单栏中选择**视图**->-**调试控制台**开始。这样做将允许我们在调试开始时看到消息。虽然能够看到这些信息不是必须的，但是拥有这些信息是很好的。

接下来，点击左侧的**调试**图标，进入**调试**部分。在该部分的顶部，打开配置下拉列表。它最初显示的是**无配置**，但我们将更改该设置。选择**添加配置**，然后选择**开始:启动包**选项。文件 **launch.json** 将出现在 IDE 中。

**注意:**如果 **Go: Launch project** 选项没有出现，可能是因为您在创建工作区时选择了错误的堆栈。别问我是怎么知道这个的。

将`"program": "${workspaceFolder}"`改为`"program": "${current.project.path}/main.go"`，如图 9 所示:

[![Edit the launch file to add a project debug configuration.](img/8299588dc5f9eeae3cb85fdd90f076f1.png "go-launch-package")](/sites/default/files/blog/2019/11/go-launch-package.png)Figure 9: Edit the launch file to add a project debug configuration.">

点击左边的**文件浏览器**图标，打开 **main.go** 文件。单击第 16 行右边的空白处，设置一个断点，如图 10 所示:

[![Set your breakpoint in the File Explorer.](img/8aa88ac0455d0ede427dba395aeb9765.png "breakpoint-set")](/sites/default/files/blog/2019/11/breakpoint-set.png)Figure 10: Set your breakpoint.">

## 启动引擎

现在，我们终于可以运行代码了。点击**调试**图标，然后点击配置下拉菜单旁边的绿色向右箭头，以调试模式启动程序。或者，只需按下 **F5** 。

当应用程序启动时，您将获得与端口号和服务的 URL 相关的消息。选择每个提示中提供的选项。IDE 中将打开一个小浏览器窗口。如果您收到一个错误，指出应用程序尚未准备好，只需刷新浏览器，直到应用程序开始运行。请注意，您可以单击小图标在 ide 外部启动默认浏览器。

您将看到断点突出显示。随意查看左边的调试值，包括局部变量。准备好后，点击调试控制区的**继续**按钮(蓝色箭头)。

调试完成后，按下调试控制区的**停止**按钮(红框)。

继续修改程序中的引用，然后再次以调试模式运行。当您对您的更改生效感到满意时，我们就可以更新我们的回购协议了。对于一个开发者来说，这就像家一样。编辑、运行、调试、编辑、运行、调试等。随时按下 **F11** 以全屏模式运行您的浏览器，您很快就会忘记您正在使用网络。

## 提交您的代码

点击**源控制**图标，查看**源控制**区域。在消息区域中，输入描述您刚刚所做更改的文本。该信息由`git commit -m`命令使用。然后，单击消息区域上方的提交复选框。这是`git commit`命令，它使用您在消息区域输入的文本。

## 推迟

现在来完成这个循环，并推动我们对 GitHub repo 的更改。记住:我们是在浏览器中工作，所以没有**文件**->-**保存**选项。

**注:**但是*会不会有*。我们一直在短暂模式下做所有的事情(如你的 IDE 底部所述)，但是如果你配置了一个持久卷，CRW 支持保存文件到存储器。那是另一篇文章了。

选择省略号(**...在源代码控制区选择**并选择**按钮**选项。此操作将提示您输入 GitHub 用户名和密码。在您正确地提供这些之后，代码将被推送到 GitHub 以更新您的回购。

跳到你的浏览器并导航到 GitHub repo，在那里你会看到你的改变。你做到了。你拉下代码，编辑，调试，推回 GitHub。当然，您现在已经配置了一个 GitHub webhook 来自动启动您的 CI/CD 管道，对吗？

对吗？

这个问题看起来像 YABP 的另一篇博文:使用 OpenShift 管道从 GitHub 到 CI/CD。

**注意:**如果你关闭你的工作空间，然后，稍后，再次启动它，它将拉下当前代码。换句话说，如果其他人在此期间做了更改，你会看到他们。这是一个小功能，但在开始工作之前有一个自动`git pull`是很好的。

## 下一步是什么？

我简单提了一个*栈*。在这个由三部分组成的系列的第三篇也是最后一篇文章中，我将向您展示如何构建一个定制栈，这是一种根据您的特定需求定制工作空间的方法。

*Last updated: June 29, 2020*