# 使用 Red Hat CodeReady Workspaces 2 管理开发环境

> 原文：<https://developers.redhat.com/blog/2020/01/02/managing-development-environments-with-red-hat-codeready-workspaces-2>

[红帽 CodeReady Workspaces 2.0 (CRW)](https://developers.redhat.com/products/codeready-workspaces/overview) 的发布带来了改变。基于 [Eclipse Che 7](https://www.eclipse.org/che/getting-started/cloud/?sc_cid=701f2000000RtqCAAS) 和[忒伊亚](https://theia-ide.org/)在线编辑器，CRW 2.0 将开发人员从特殊配置的 PC 的束缚中解放出来，支持多个特殊配置的工作空间。想象一下，每种语言、版本、工具等等都有一个独立的工作环境，所有这些都可以通过浏览器获得。这篇文章讨论了 CRW 的一些特点。

## 现代开发环境的组成部分

### 工作区

CRW 的核心是“工作空间”的概念和实现工作空间是一个开发环境，可以比作一台装载了操作系统、编程语言、工具、编辑器以及一个或多个开发项目的 PC。您甚至可以访问浏览器中运行的命令行。

您可能有一个工作区，在一个 [CoreOS](https://coreos.com/) 映像上安装了 Java 和 Maven。另一个工作空间可能基于安装了 Node.js 和 MongoDB T5 的 T2 红帽企业 Linux(RHEL T3)。需要在一些方面下功夫。网芯 3.0 代码？没问题，简单创建一个工作区。是的，尽管 Che 7 和 CRW 最初可能是为 Java 开发人员设计的，但实际上它是非常中性的语言。它对语言服务器协议的使用确保了未来的语言支持。是的，甚至支持[COBOL](https://microsoft.github.io/language-server-protocol/implementors/servers/)。格蕾丝·赫柏遇到了蒂姆·伯纳斯·李。

在每种情况下，您都可以选择——并且可能应该——包含一个软件项目。对于大多数用例，这将是存储在 Git 存储库中的源代码。CodeReady Workspaces 处理这一点，因为内置了 GitHub 集成。

在 CRW，你只需启动一个工作区，就像打开电脑一样。几秒钟之内，代码编辑器就在你面前打开了，你的项目被加载，你需要的所有工具和语言支持都在手边。可以定义几个工作空间，在它们之间切换很容易。用鼠标点击开始和停止工作空间。在此处立即停止，稍后从另一台计算机重新启动工作区。事实上，你需要的只是一个浏览器。

### 浏览器内编辑器

CRW 的在线编辑器是忒伊亚，这是一个基于浏览器的编辑器，基于微软的摩纳哥编辑器。这个名字可能不熟悉，所以考虑一下这个:Monaco 是构建 Visual Studio 代码(VS Code)的编辑器。这真的将编辑器联系在一起，因为一些 VS 代码插件将与忒伊亚 API 兼容。简单来说:一些 VS 代码插件可以和 CRW 一起工作。

因为 VS 代码如此受欢迎，切换到 CRW 是一件轻而易举的事。事实上，如果你将浏览器设置为全屏模式(按 F11)，你很快就会忘记你是在浏览器中工作的。非常圆滑。

**旁白:**为了强调这一点，并展示我有多古怪，我拿起我的旧 Windows 10 手机，将其连接到我的微软 Continuum 适配器，并在 Edge 浏览器中编写 Node.js 代码。没错；*我用手机写代码*。我是说，总得有人去做。

底线是:如果你有一个互联网连接和一个浏览器，你可以在一个感觉就像在本地 PC 上工作的环境中编辑你的代码。

### 堆栈配置

栈是操作系统、编程语言支持、工具和创建工作空间所需的任何其他部分的组合。工作空间是使用堆栈作为起点构建的。可以这样想:栈是一个定义，很像软件中的类，而工作空间是栈的实例，就像对象是类的实例一样。CRW 提供了几个预配置的堆栈，您可以轻松创建自己的堆栈来满足您的特定需求。

### 工厂

工厂是构建和共享工作空间的方式。也就是说，在您有了一个专门为您需要做的事情定制的工作空间之后，您就可以创建一个构建该工作空间的工厂。一旦构建完成，只需共享 URL 就可以共享工厂。当队友访问 URL 时，工厂将构建工作空间，他们将拥有与您完全相同的环境。

## 例子

以上介绍了基础知识，接下来让我们看看如何运行 CRW 和调配工作空间。

### 如何开始

第一步是启动并运行 OpenShift 集群配置和代码就绪的工作区。你可以在 Red Hat Developer code ready work spaces 页面上找到详细的说明。

### 创建工作空间

轻松进入 CRW 最快最简单的方法是创建一个预定义的工作空间。从控制面板的主工作区页面，单击“添加工作区”按钮开始。

![](img/43f385c36ab69e0dc8b628e8c0bd450d.png)

下一页是我们可以选择工作区的地方。

![](img/0b6ec4cca5fddceddeab19cc11876703.png)

对于本文，我选择 Java-Maven 工作区。我还将工作区的名称改为 wksp-java-maven。您可能希望养成这种习惯，因为默认名称提供的关于工作区的信息非常少。

单击“创建并打开”按钮。页面顶部有一个，底部有一个——两个都做同样的事情——等一分钟左右。即时工作空间。

## 我们才刚刚开始

所以，现在你在工作空间中有了一些代码。我的下一篇文章将指导您完成编辑-构建-调试-提交的循环。

#### 请参见

*   [Red Hat CodeReady Workspaces 2:加速 Kubernetes 开发的新工具](https://developers.redhat.com/blog/2019/12/03/red-hat-codeready-workspaces-2-new-tools-to-speed-kubernetes-development/)
*   [CodeReady 工作区开发文件，解密](https://developers.redhat.com/blog/2019/12/09/codeready-workspaces-devfile-demystified/)

*Last updated: June 29, 2020*