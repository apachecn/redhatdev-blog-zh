# Node.js 参考架构介绍，第 1 部分:概述

> 原文：<https://developers.redhat.com/blog/2021/03/08/introduction-to-the-node-js-reference-architecture-part-1-overview>

欢迎来到这个介绍 Red Hat 和 IBM 的 [Node.js 参考架构](https://github.com/nodeshift/nodejs-reference-architecture)的新系列。这篇文章概述了我们开发 Node.js 参考架构的原因——包括我们希望该架构能为我们的开发人员社区提供什么，以及我们*不希望它做什么。未来的文章将详细介绍参考架构的不同部分。*

在我们深入第一篇文章之前，重要的是要认识到, [Node.js](https://developers.redhat.com/topics/nodejs) 参考架构是一项正在进行的工作。开发团队正在不同的领域工作，讨论我们学到的东西，并将这些信息提炼为简明的建议和指导。鉴于 [JavaScript](https://developers.redhat.com/topics/javascript) 生态系统的快速发展，参考架构可能永远不会“完成”相反，我们将继续更新它，以反映我们通过新的 Node.js 生产部署学到的东西以及我们大规模部署的持续经验。参考架构旨在反映我们当前的经验和思维，这将会不断发展。

## 阅读整个系列

您可以在这里找到本系列中关于 Node.js 参考体系结构的其他文章:

*   **第 1 部分:Node.js 参考架构概述**
*   第二部分:[登录 Node.js](/articles/2021/05/10/introduction-nodejs-reference-architecture-part-2-logging-nodejs "Introduction to the Node.js reference architecture, Part 2: Logging in Node.js")
*   第三部分:[node . js 中的代码一致性](/articles/2021/05/10/introduction-nodejs-reference-architecture-part-2-logging-nodejs "Introduction to the Node.js reference architecture, Part 2: Logging in Node.js")
*   第 4 部分:[node . js 中的 graph QL](/articles/2021/06/22/introduction-nodejs-reference-architecture-part-4-graphql-nodejs "Introduction to the Node.js reference architecture, Part 4: GraphQL in Node.js")
*   第 5 部分:[构建良好的容器](/articles/2021/08/26/introduction-nodejs-reference-architecture-part-5-building-good-containers "How to build good containers in Node.js")
*   第 6 部分:[选择 web 框架](/articles/2021/12/03/introduction-nodejs-reference-architecture-part-6-choosing-web-frameworks "Introduction to the Node.js reference architecture, Part 6: Choosing web frameworks")
*   第 7 部分:[代码覆盖率](/articles/2022/03/02/introduction-nodejs-reference-architecture-part-7-code-coverage "Introduction to the Node.js reference architecture, Part 7: Code coverage")
*   第八部分:[打字稿](/articles/2022/04/11/introduction-nodejs-reference-architecture-part-8-typescript "Introduction to the Node.js reference architecture, Part 8: TypeScript")
*   第 9 部分:[保护 Node.js 应用程序](/articles/2022/08/09/8-elements-securing-nodejs-applications "8 elements of securing Node.js applications")
*   第 10 部分:[可访问性](/articles/2022/11/03/nodejs-reference-architecture-part-10-accessibility "Node.js Reference Architecture, Part 10: Accessibility")

## 为什么我们需要 Node.js 参考架构

JavaScript 生态系统发展迅速，充满活力。你只需要看看节点包管理器(npm)模块的[增长率就知道了。2016 年，大约有 250，000 个国家预防机制包。2018 年，这一数字攀升至 52.5 万左右，2020 年约为 110 万。这些数字代表了 JavaScript 生态系统中相当多的选择和多样性。这显然是蓬勃发展的创新和测试新想法的优势。](http://www.modulecounts.com/)

另一方面，各种各样的选项使得在 Node.js 包中进行选择非常困难。对于任何模块，您可能会发现几个同样好的选择，以及几个可能非常糟糕的选择。每个应用程序都有一个成功的“秘方”。必须找到最合适、最新或最具创新性的封装，用于这一应用领域。对于应用程序的其余部分，您可能想要一些有用的东西，并且您可以在您的组织中分享任何经验或最佳实践。在后一种情况下，拥有一个参考架构可以帮助团队避免一次又一次地重新学习相同的东西。

## 什么是参考架构

我们在 Red Hat 和 IBM 的 Node.js 团队不可能成为`npm`注册表中 110 万个 JavaScript 包的专家。同样，我们不能像参与 Node.js 项目那样参与所有的项目。相反，我们的经验是基于我们对 Node.js 的广泛使用。这包括像[气象公司](https://developer.ibm.com/languages/node-js/articles/nodejs-weather-company-success-story/)这样的大规模部署，以及我们的咨询小组与客户一起做的工作。

如果每个寻求 Node.js 应用程序帮助的内部团队和客户都使用不同的包，那么帮助他们将会困难得多。问题是，我们如何在整个组织中分享我们的知识？

我们希望帮助我们的内部团队和客户做出好的选择和部署决策。在团队不需要使用特定包的情况下，我们可以根据我们在 Red Hat 和 IBM 中构建的经验推荐一个包。作为开发人员，我们可以使用 Node.js 参考架构跨团队和项目进行共享和协作，并在我们的部署中建立共同点。

## 参考架构不是什么

我已经描述了我们希望用 Node.js 参考架构做什么。同样重要的是，要清楚我们是在努力做什么，而不是在努力做什么。

首先，参考架构不是试图说服或强迫开发人员使用我们选择的包。部署是多种多样的，在不同的环境中使用特定的模块是有充分理由的。

第二，我们并不声称我们的建议比备选方案更好。正如我提到的，您经常会发现 JavaScript 生态系统中有几个同样好的包或方法。我们的建议倾向于 Red Hat 和 IBM 团队已经成功使用的技术和我们熟悉的技术。我们并不试图引导任何人做出“最好”的选择，而是“好”的选择。拥有一个参考体系结构可以最大限度地利用已经学到的经验和共同点，这样我们就可以互相帮助。

## 关于这个系列

在我们研究参考架构的每个部分时，Node.js 开发团队正在进行有趣的讨论。同时，我们试图保持参考体系结构的内容简明扼要。正如我所提到的，目标是为应用程序的总体架构提供好的选择，以便开发人员可以专注于应用程序的“秘方”在大多数情况下，使用参考架构的开发人员会想知道使用什么包或技术以及如何使用。因此，参考体系结构不会包含太多导致我们做出决策的有趣背景和讨论。

本系列*将*分享从我们内部讨论中获得的观点。在我们学习参考体系结构的每个部分时，我们将使用本系列提供更多参考，并有机会深入了解相关主题的更多细节。我想你会发现 Node.js 团队中开发人员的不同经历会让你思考。我从我们经历的每一部分都学到了一些东西，我希望你也一样。

## 下一步是什么？

作为本系列的一部分，我们计划定期讨论新的主题。在您等待下一期的时候，我们邀请您访问 GitHub 上的 [Node.js 参考架构库](https://github.com/nodeshift/nodejs-reference-architecture)。您将能够看到我们已经完成的工作，以及您可以从这个系列中期待的各种主题。要了解更多关于 Red Hat 在 Node.js 方面的进展，请查看我们的 [Node.js 登陆页面](https://developers.redhat.com/topics/nodejs)。

*Last updated: November 7, 2022*