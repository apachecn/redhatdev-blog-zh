# 扩展 Eclipse Che 7 以使用 VS 代码扩展

> 原文：<https://developers.redhat.com/blog/2019/01/22/extending-eclipse-che-7-to-use-vs-code-extensions>

最近，Eclipse Che 社区一直致力于使 Eclipse 忒伊亚成为 Eclipse Che 7 的默认 web IDE。我们为 Eclipse 忒伊亚添加了一个插件模型，它与 Visual Studio 代码扩展兼容。Che 7 用户最终将能够在他们基于云的开发人员工作区中利用为 VS 代码编写的扩展。值得指出的是 VS 代码扩展的流行。红帽[贡献了扩展](https://marketplace.visualstudio.com/publishers/redhat)，涵盖 [Java](https://marketplace.visualstudio.com/items?itemName=redhat.java) 、 [XML](https://developers.redhat.com/blog/2018/12/04/xml-language-server-vscode-extension/) 、 [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) 、 [OpenShift](https://developers.redhat.com/blog/2018/11/28/announcing-red-hat-openshift-extension-for-visual-studio-code-public-preview/) 和[依赖分析](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics)。红帽提供的 Java 扩展已经下载过千万次了！

如果你不熟悉 Eclipse 忒伊亚，Che 6 和更早的版本使用的是基于 GWT 的 IDE。虽然在那种环境下可以开发和使用插件，但是很麻烦。来自 VS Code 这样的工具，开发人员希望能够在运行时定制和扩展他们的工作空间。Eclipse 忒伊亚是一个可扩展的开源框架，使用最先进的 web 技术开发多语言 ide。将忒伊亚作为 Che 7 的默认 IDE 为丰富 Che 中的开发人员工作区提供了基础。请参阅 Stevan LeMeur 的系列文章了解更多关于 Che 7 的信息。

这篇文章解释了为什么我们决定将新的插件模型添加到 Eclipse 忒伊亚以及 Eclipse Che 7 开发人员工作区的好处。我还将介绍新的插件模型与现有的忒伊亚扩展模型有何不同。

## 以前基于 GWT 的 IDE 的缺点

在 Che 6 和早期版本中使用的基于 GWT 的 IDE 有很多缺点。添加插件需要停止、重新编译和重新加载整个 IDE。尝试使用 [JS-Interop](http://www.gwtproject.org/doc/latest/DevGuideCodingBasicsJsInterop.html) 动态加载 JavaScript 插件。基于 GWT 的 IDE 提供了一个低级 API，优点是你可以改变任何东西，但缺点是任何插件都可能破坏任何东西。此外，很难理解当前 API 的所有入口点。

最后，我们还必须考虑到，许多人不喜欢 GWT，认为它是一种过时的技术。

## 扩展性要求

根据我们的经验，很多人希望为 Eclipse Che 的下一个主要版本改进可扩展性模型。对于 Che 7，我们提出了以下要求:

1.  它应该很容易在运行时加载插件，它不应该涉及任何额外的编译或安装步骤。所以插件应该已经编译好了。IDE 只需要加载代码。

[![Requirement 1: Fast loading of Che workspace](img/9d08533c19652abbd9a156d6d1f26c33.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c4129ec76aa4.png)

2.  一个写得很差的插件应该是无法攻破整个 IDE 的。如果用户加载了一个有错误的插件，用户仍然可以继续使用当前的 IDE。

[![Requirement 2: Secure loading](img/c21f011662f35a32ec84658cb3099451.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412a1b12d2f.png)

3.  在 Eclipse Che 中，我们希望保证插件不会阻塞 IDE 的主要功能，比如打开文件或输入。用户应该能够识别问题是由插件引起的还是核心产品本身的问题。如果两个插件对一个依赖项的不同/冲突版本有需求，那应该是允许的，不应该引起问题。每个插件应该获得它所需要的依赖项的特定版本。

[![Requirement 3: Code isolation](img/3b6d768754b5616dfe7098623982395b.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412a33d884c.png)

## 忒伊亚扩展模型的缺陷

Eclipse 忒伊亚被选为 Che 7 及以后版本的替代 IDE。忒伊亚有一个扩展模型，我们称之为忒伊亚扩展。现有扩展模型的问题在于，它主要是为开发定制的 ide 而设计的。因此，当开发人员试图像在 VS 代码中那样在运行时定制他们的开发工作空间时，它也有类似的缺点:

1.  使用 Eclipse 忒伊亚扩展，当添加一个新的扩展时，整个 IDE 都要重新编译。如果扩展引入了错误，您可能会破坏整个 IDE。因此，在添加一个扩展后，您可能会打开一个 Che 工作区，并由于编译错误而不是 IDE 而以一个空白页面结束。

[![Drawback 1: When a new extension is added, the whole IDE is recompiled](img/76af3e4fa5839399ffd55baa00d4887f.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412a8261403.png)

2.  从`npmjs`存储库中检索扩展。虽然这很好，因为`npmjs`有大量的库，但是当你安装一个扩展时，它会一次又一次地下载所有的依赖项。如果你有很多依赖，它可能会崩溃。此外，您不能为私有扩展添加本地存储库。

[![Drawback 2: Extensions are retrieved from the npmjs repository.](img/ad05ecf16f5588916bc7bb43b43ba4c1.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412a9a3d4e0.png)

3.  忒伊亚扩展允许扩展编写者定制整个 IDE。然而，与基于 GWT 的 IDE 类似，任何扩展都可以轻易地破坏整个 IDE。诊断可能很困难。

[![Drawback 3: Any extension can easily break the whole IDE](img/67b7fc343321eac2a452ed3e247f21f7.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412aba63d45.png)

4.  扩展模型的复杂性对于新开发人员来说太具有挑战性了。忒伊亚扩展模型有很多对高级用户来说很棒的功能；但是，如果你想写你的第一个扩展，你需要掌握 [inverify](http://inversify.io/) 和依赖注入。你还需要知道哪个类在做什么，你需要实现哪个接口。

[![Drawback 4: The extension model is too challenging for new developers](img/309e91db4f07b22c8e85f769cc2ee39e.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412d8006a6a.png)

显然，忒伊亚的扩展模型不符合我们对可扩展性的要求。

![](img/a3d8fb3ada35dad0e31918b98825ed4b.png)

## 介绍忒伊亚插件

在 Red Hat，为了满足我们的可扩展性需求，我们提出了忒伊亚插件模型。关键方面是:

1.  插件可以在运行时随时加载，无需重启/刷新 IDE。

[![Extensibility requirement 1: Plugins can be loaded at any time at the runtime](img/b48d28633fa90c531d7b6b7d63c1b595.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412df057175.png)

2.  Eclipse 忒伊亚插件是自包含的，打包成。忒伊亚档案。它们包含插件的所有运行时代码。启动时不需要下载其他任何东西。

[![Extensibility requirement 1: Theia plugins are self-contained](img/e70ebb601a98d9bef76259d5f5fa2bdc.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c412e513cb9b.png)

3.  忒伊亚插件有一个简单的易于学习的 API。您可以使用依赖注入框架，但不是必须的。这是你的选择。该模型非常简单，只需导入一个名称空间`@theia/plugin`(通过`npmjs`包`@theia/plugin`，您可以通过这个对象上的代码完成从这个入口点获得您需要的东西。您通过实现`start`和`stop`函数来实现插件的生命周期。

## 忒伊亚插件的示例代码

```
import * as theia from '@theia/plugin';

export function start(context: theia.PluginContext) {
    const informationMessageTestCommand = {
        id: 'hello-world',
        label: "Hello World"
    };
    context.subscriptions.push(theia.commands.registerCommand(informationMessageTestCommand, (...args: any[]) => {
        theia.window.showInformationMessage('Hello World!');
    }));

}

export function stop() {
  // commands automatically unregistered
}

```

## 忒伊亚插件协议

[![Theia plugin protocol ](img/5117d760c12dc0c81e42eb37cf78fd36.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c41300ae3f0f.png)

忒伊亚插件使用协议，这意味着你可以在任何地方运行插件！一些插件可以运行在浏览器的工作线程中(它们被称为前端插件)，或者它们可以运行在服务器端的单独进程中(后端插件)。处理其他类型的名称空间很容易，包括 VS 代码扩展。

这是一个建筑图:

[![Architecture diagram](img/7a22d343ea465d371a5bf25bb6ed64ea.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c4130924d1a5.png)

该模型是向后兼容的。插件模型是通过 TypeScript 声明文件提供的。插件代码可以完全重写，或者忒伊亚类可以重构；但是，模型将保持不变。

[![The model is backward-compliant](img/1e52fb3af9cf041c425bde2b40ee9e7f.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c41317703151.png)

该 API 是一个高级 API，旨在使插件无法破坏 IDE。你也许不能改变插件中你能想到的所有东西，但是有几乎无限的可能性为开发者提供有用的功能。

## 集装箱就绪

![](img/dcb39402bc10af36b8aaa9ba882c77ed.png)

Eclipse Che 使用[容器](https://developers.redhat.com/blog/category/containers/)作为开发工具。忒伊亚插件是用 TypeScript/Javascript 编写的，运行良好。但是有时候，插件编写者需要一些依赖，不仅仅是纯粹的`npmjs`依赖。例如，如果开发人员为 Java 编写一个语言服务器，这个插件很可能需要 Java。所以这可能意味着运行 Eclipse 忒伊亚的容器应该已经安装了 Java。

这就是为什么在 Eclipse Che 中，可以在自己的容器中运行每个 Eclipse 忒伊亚插件。这允许插件在自己的容器中使用任何它需要的系统依赖。

默认情况下，所有插件都在忒伊亚容器中作为单独的进程执行:

[![All plugins are executed as a separate process in the Theia container](img/d1932b43c946a10310f6410a103221ba.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c41338c4e3f3.png)

## VS 代码扩展

Eclipse 忒伊亚插件协议已经以可扩展的方式实现，并且符合 VS 代码 API。这将允许一些 VS 代码扩展在忒伊亚内部运行。API 支持将决定哪些扩展是兼容的。

例如，目前可以使用来自 VS 代码市场的 SonarLint VS 代码扩展:

[![SonarLint Extension on the VS Code Marketplace](img/db4ef8c4fdfb1db5f046fd3e327ccc86.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c4135b44e095.png)

在运行时加载 SonarLint VS 代码扩展后，您可以在 JavaScript 源文件中看到即时结果:

[![Results in a JavaScript source file](img/03738e22c89da15ada7dc9f1ea261fe0.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c4135f4425ec.png)

下面是加载 VS 代码扩展后 Eclipse 忒伊亚中的插件视图:

[![Plugins view in Eclipse Theia after loading the VS Code extension](img/9c79a405117663062b73b8cfd55a110a.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/img_5c4136a319e1a.png)

## 现在就试试 Eclipse Che 7 吧！

想试试新版 Eclipse Che 7？方法如下:

点击以下工厂网址:
[che.openshift.io/f?id=factoryvbwekkducozn3jsn](https://che.openshift.io/f?id=factoryvbwekkducozn3jsn)

**或者，在 [che.openshift.io](https://che.openshift.io/) 、**新建一个工作区、**上创建自己的账户**，选择“Che 7”栈。

[![Try Eclipse Che 7 on OpenShift](img/a89153020956e7697d95b1b5268eaa90.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/12/che-on-openshift.png)

您还可以通过安装最新版本的 Eclipse Che 在本地机器上进行测试。参见 *[与](http://www.eclipse.org/che/docs/#getting-started)*月蚀车快速入门。

## 想了解更多？

看到*月食 Che 7 来了而且真的很热*文章系列:

*   第 1 部分— [Eclipse Che 7 概述，并介绍新的 IDE](https://che.eclipse.org/eclipse-che-7-is-coming-and-its-really-hot-1-4-64d79b75ca02)
*   第 2 部分— [介绍插件模型](https://che.eclipse.org/eclipse-che-7-is-coming-and-its-really-hot-2-4-2e2c6accbff4)
*   第 3 部分—[Kube-本地开发人员工作区](https://developers.redhat.com/blog/2018/12/20/eclipse-che-7-is-coming-and-its-really-hot-3-4/)
*   第 4 部分— [企业开发团队的功能和时间表](https://developers.redhat.com/blog/2018/12/21/eclipse-che-7-is-coming-and-its-really-hot-4-4/)

**关于 Che 在 [Red Hat OpenShift](http://openshift.com/) 上运行的信息，参见[Red Hat code ready work spaces For open shift](https://developers.redhat.com/products/codeready-workspaces/overview)(目前处于测试阶段)和 Doug Tidwell 的文章和视频，[*code ready work spaces For open shift(测试版)——它在他们的机器上也能工作*](https://developers.redhat.com/blog/2018/12/11/codeready-workspaces-openshift/) 。道格涵盖堆栈和工作区和工厂，以帮助您开始与 Che。**

*Last updated: February 21, 2019*