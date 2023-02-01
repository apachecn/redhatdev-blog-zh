# JavaScript 应用程序开发入门

> 原文：<https://developers.redhat.com/blog/2020/04/22/getting-started-with-javascript-application-development>

对于许多以前从未构建过 [JavaScript](https://developers.redhat.com/blog/category/javascript/) web 应用程序的开发人员来说，最初的步骤可能会令人望而生畏。我们的开发团队有机会与刚入门的学生和拥有构建复杂应用程序的丰富经验的开发人员进行交流。即使是经验丰富的后端开发人员也经常询问他们可以从哪里开始使用 JavaScript。我们的回答总是，“不要只是读书。你需要开始构建东西，用这种语言来玩，看看它能做什么。”

## JavaScript 框架

很多时候他们也会问:“我应该学哪个框架？”像 Angular、Vue 或 React 这样的 JavaScript 框架令人兴奋，但是它们混淆了从哪里开始的画面。在这个阶段，许多开发人员可能根本不想选择一个框架，这样他们就不会把自己局限在一种特定的技术中。如果你想知道同样的事情，你并不孤单。幸运的是，有大量免费资源可以帮助您开始学习如何构建企业级的 JavaScript 应用程序。

另一个好消息是，您选择的 JavaScript 框架最终不会对您的应用程序的用户体验产生影响。如果你把内容和信息架构放在第一位，那么这个框架就变成了一个简单的实现细节。从个人经验来说，很容易对一个特定的框架感到兴奋，但这可能会导致在快速变化的环境中长期失望。相反，理解 JavaScript 的核心将为您将来的高质量 web 开发做好准备。

考虑到这一点，我想分解一种方法，可以帮助您为前端开发做好准备。我将讨论的许多领域在整个 JavaScript 开发生态系统中都是常见的，这里学到的技能是可以转移的。除了解释如何从这些开始，我还想整理一份对入门有价值的资源列表。

我认为应用程序开发人员的成长过程有两个基本步骤。第一步，学习 JavaScript 生态系统，然后学习 web 应用程序架构。了解 JavaScript 生态系统包括学习 JavaScript 和练习 JavaScript 编码。然后，您可以构建您的第一个 [Node.js](https://developers.redhat.com/blog/category/node-js/) 应用程序。

第二步，理解 web 应用程序架构，也包括两个阶段。您需要将您的 JavaScript 技能转变为构建 web 应用程序，并为您的代码做出架构决策。然后，您可以为您的应用程序做出构建和部署决策。让我们一步一步来。我不会说太多的细节，我只是概述步骤，并提供可以帮助做出这些选择的资源。

## JavaScript 生态系统

我们将从第一步的两个阶段开始，这将引导您编写第一个 Node.js 应用程序。

### 学习 JavaScript 并练习编写 JavaScript 代码

Mozilla Developer Network (MDN)有一个很好的资源可以帮助您快速学习 JavaScript。[这个 JavaScript 再介绍](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)可以帮助您对基础知识有一个高层次的概述，需要 30-60 分钟才能完成。一旦你熟悉了基础知识，深入学习这门语言是很重要的。这一部分需要时间，但是理解 JavaScript 的力量和这种语言的一些古怪之处将被证明是非常宝贵的。

对 JavaScript 的理解也为任何前端开发工作提供了坚实的基础。所有以浏览器为目标的框架最终都会以某种形式使用 JavaScript 来实现交互性。为了更深入，MDN 文档提供了更深入的教程。我还发现[你还不知道 JS](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/README.md)是开发人员扩展其 JavaScript 知识不可或缺的资源。作者将该资源描述为“深入探究 JavaScript 语言核心机制的系列书籍”

### 构建您的第一个节点应用程序

既然您已经掌握了这门语言，那么上面提到的教程很可能会产生一个节点应用程序来运行您的 JavaScript。如果没有，了解 Node.js 生态系统是有帮助的。浏览器可以在没有任何帮助的情况下运行 JavaScript，但是基于开源库并利用编译和捆绑资产的 web 应用程序是必不可少的。这种需求需要使用 Node.js。要开始使用 Node.js，我建议通过这篇对 Node.js 的[介绍开始探索 Node。之后，](https://nodejs.dev/introduction-to-nodejs) [Nodejs.dev](https://nodejs.dev/) 也有关于如何在本地安装 Node.js 和入门的信息。

节点可用于在浏览器外执行 JavaScript 文件。它也是构建和使用 web 应用程序的关键部分。Node 包含一个名为 NPM 的包管理器，您可能听说过。NPM 的一个关键部分是`package.json`文件，在 [Nodejs.dev 站点](https://nodejs.dev/the-package-json-guide)上也有详细描述。

大多数开发人员会非常熟悉在空目录中运行`npm init`[。这个动作几乎总是构建基于节点的 web 应用程序的第一步。一旦`package.json`文件存在，您可以将命令添加到`scripts: {}`部分来执行命令。例如，您可以添加如下内容:](https://docs.npmjs.com/cli/init)

```
"scripts": {
"hello": "echo \"hello\""
}
```

保存`package.json`文件。然后，从命令行运行:

```
$ npm run hello
```

您应该会看到输出:`"hello"`。这个文件的脚本部分非常强大。我鼓励你熟悉`package.json`以及如何使用它执行命令。

既然您已经有了 JavaScript 基础，并且在较高层次上理解了如何使用 Node.js，那么是时候开始构建您的 JavaScript web 应用程序了。有许多选择要做，但是不要在这个兔子洞里走得太远，让我们进入下一步。

## 了解 web 应用程序架构

现在让我们进入第二步的两个阶段，这两个阶段将引导您为应用程序做出构建和部署决策。

### 将 JavaScript 技能过渡到 web 应用程序及其架构

在编写一行代码之前，先考虑应用程序的架构是一个重要的起点。我见过许多应用程序在开发人员没有考虑他们的应用程序如何扩展时被重构。只有五个文件的组件文件夹看起来很有条理，但一旦有了 100 个文件，就很难找到你要找的东西了。任何会影响您可能创建的每个 JavaScript 文件的决定都应该预先仔细考虑。

很容易被这个过程淹没，陷入优柔寡断的循环。您可以通过识别已建立的工具并从已经多次这样做的其他人的例子中学习来打破这一点。为此，当您在构建软件时做出许多决定时，在编写任何代码之前，这些都是需要考虑的好问题:

*   JavaScript 框架选择
*   用于扩展的文件/文件夹结构
*   CSS 架构
*   不管是不是打字稿
*   代码林挺
*   测试方法
*   按指定路线发送
*   状态管理和缓存层

### 为您的应用程序制定构建和部署决策

一旦做出了基本的代码架构决策，您就需要决定开发人员将如何构建和使用代码。您还需要了解最终的代码将如何编译以交付生产。关于框架的早期选择可以使接下来的决定变得更容易:这些框架通常会附带一些构建工具。

我还发现这些决策在以后更容易更改，因为这些层通常位于代码之外。考虑以下因素时，选择标准工具选项(如 webpack 或 gulp)会有很大帮助:

*   本地开发人员环境
*   源地图
*   模块捆扎机
*   生产优化

在 [PatternFly](https://www.patternfly.org/) 团队中，我们投入了相当多的精力和我们的综合经验来记录和实现 web 应用程序架构。我们的目标是帮助团队快速开始使用 PatternFly。如果您不熟悉 PatternFly，它是一个开源设计系统，提供了指南、资源等，可以帮助您让您的应用程序具有专业的外观和感觉。

自从我们发现我们的大部分用户都在标准化基于 React 的应用程序后，我们最近更关注于交付基于 React 的应用程序。要开始使用 ReactJS，我建议您查看 reactjs.org[网站](http://reactjs.org)上的[入门页面](https://reactjs.org/docs/getting-started.html)。如果你是一名设计师，想开始学习更多关于 ReactJS 开发的知识， [React for designers](https://reactfordesigners.com/) 是另一个很好的资源。

最后，如果您想看看所有这些的运行情况，可以查看一个 [ReactJS starter 应用程序](https://github.com/patternfly/patternfly-react-seed)来开始使用 PatternFly。

## 最终想法

JavaScript 生态系统非常有趣，但是它变化很快。虽然这对于我们许多喜欢稳定的人来说可能是压倒性的，但专注于基础知识，然后将核心知识应用到您的 web 应用程序，将有助于您成为一名成功的 web 应用程序开发人员。

*Last updated: June 29, 2020*