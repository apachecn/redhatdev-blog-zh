# Node.js 参考架构介绍，第 3 部分:代码一致性

> 原文：<https://developers.redhat.com/articles/2021/05/17/introduction-nodejs-reference-architecture-part-3-code-consistency>

欢迎回到我们正在进行的关于 [Node.js](/topics/nodejs) 参考架构的系列。[第 1 部分](/blog/2021/03/08/introduction-to-the-node-js-reference-architecture-part-1-overview/)介绍了 Node.js 参考架构是什么，[第 2 部分](https://developers.redhat.com/articles/2021/05/10/introduction-nodejs-reference-architecture-part-2-logging-nodejs)介绍了日志记录。在本文中，我们将深入研究代码一致性，以及如何使用像 ESLint 这样的 linter 工具来加强代码一致性。

**阅读到目前为止的系列**:

*   第 1 部分:[node . js 参考架构概述](/blog/2021/03/08/introduction-to-the-node-js-reference-architecture-part-1-overview "Introduction to the Node.js reference architecture, Part 1: Overview")
*   第二部分:[登录 Node.js](/articles/2021/05/10/introduction-nodejs-reference-architecture-part-2-logging-nodejs "Introduction to the Node.js reference architecture, Part 2: Logging in Node.js")
*   **第三部分**:**node . js 中的代码一致性**
*   第 4 部分:[node . js 中的 graph QL](/articles/2021/06/22/introduction-nodejs-reference-architecture-part-4-graphql-nodejs "Introduction to the Node.js reference architecture, Part 4: GraphQL in Node.js")
*   第 5 部分:[构建良好的容器](/articles/2021/08/26/introduction-nodejs-reference-architecture-part-5-building-good-containers "How to build good containers in Node.js")
*   第 6 部分:[选择 web 框架](/articles/2021/12/03/introduction-nodejs-reference-architecture-part-6-choosing-web-frameworks "Introduction to the Node.js reference architecture, Part 6: Choosing web frameworks")
*   第 7 部分:[代码覆盖率](/articles/2022/03/02/introduction-nodejs-reference-architecture-part-7-code-coverage "Introduction to the Node.js reference architecture, Part 7: Code coverage")
*   第八部分:[打字稿](/articles/2022/04/11/introduction-nodejs-reference-architecture-part-8-typescript "Introduction to the Node.js reference architecture, Part 8: TypeScript")
*   第 9 部分:[保护 Node.js 应用程序](/articles/2022/08/09/8-elements-securing-nodejs-applications "8 elements of securing Node.js applications")
*   第 10 部分:[可访问性](/articles/2022/11/03/nodejs-reference-architecture-part-10-accessibility "Node.js Reference Architecture, Part 10: Accessibility")

## 为什么代码一致性很重要

作为一个团队有效地处理 JavaScript 项目的一个关键方面是保持代码格式的一致性。这确保了当不同的团队成员在共享代码库上协作时，他们知道期望什么样的编码模式，允许他们更有效地工作。缺乏一致性增加了开发人员的学习曲线，并可能偏离主要的项目目标。

当 Red Hat 和 IBM 的 Node.js 团队开始讨论代码一致性时，很快就发现这是一个人们有强烈意见的领域，一种尺寸不能适合所有人。令人惊讶的是，你可以花这么多时间来谈论支架的正确位置！

不过，我们可以达成一致的一点是，在一个项目中使用一致的风格并通过自动化来执行它的重要性。

## 埃斯林特

在调查 Red Hat 和 IBM 用于检查和执行代码一致性的工具时， [ESLint](https://eslint.org/) 很快成为最受欢迎的选择。这个可配置的 linter 工具分析代码以识别 JavaScript 模式并保持质量。

虽然我们发现不同的团队使用不同的代码风格，但他们中的许多人报告说他们使用 ESLint 来完成工作。ESLint 是由 [OpenJS 基金会](https://openjsf.org/)主持的[开源](https://developers.redhat.com/topics/open-source/)项目，证实了它是开放治理的可靠选择。我们知道我们总是有机会贡献补丁并参与项目。

ESLint 附带了许多预先存在的代码样式配置，您可以轻松地将它们添加到您的项目中。使用这些可共享配置之一有很多好处。通过使用现有的配置，您可以避免“重新发明轮子”；其他人可能已经创建了您正在寻找的配置。另一个优势是新的团队成员(或开源贡献者)可能已经熟悉了您正在使用的配置，这使得他们能够更快地上手。

以下是一些常见的配置，可帮助您入门:

*   [T2`eslint-config-airbnb-standard`](https://www.npmjs.com/package/eslint-config-airbnb-standard)
*   [T2`eslint-config-semistandard`](https://www.npmjs.com/package/eslint-config-semistandard)
*   [T2`eslint-config-standard`](https://www.npmjs.com/package/eslint-config-standard)
*   [T2`eslint-config-prettier`](https://github.com/prettier/prettier-eslint)

使用这个查询，可以在 npmjs.org[上找到完整的列表。](https://www.npmjs.com/search?q=eslint-config-&ranking=popularity)

注意，我们不推荐任何特定的代码风格或 ESLint 配置。更重要的是选择一个标准，并在整个组织中一致地应用它。如果这是不可能的，那么您至少应该确保它在相关项目中得到一致的使用。

在这一点上，我必须承认我们真的没有花太多时间讨论括号应该放在哪里。但是这也是我们建议查看现有配置的原因之一:采用现有的最佳实践节省了大量时间(和争论),因此您可以将这些时间用于编码。

### 将 ESLint 添加到 Node.js 项目中

基于参考架构中的建议，Red Hat Node.js 团队最近更新了 [NodeShift 项目](https://github.com/nodeshift)以使用 ESLint。

将 ESLint 添加到项目中是一个非常简单的过程。事实上，ESLint 有一个向导，您可以在命令行界面上运行它来帮助您入门。您可以运行:

```
$ npx eslint --init 
```

然后按照提示进行操作。这篇文章不会深入到`init`向导的细节，但是你可以在 [ESLint 文档](https://eslint.org/docs/user-guide/getting-started)中找到更多信息。

我们的团队喜欢使用分号，所以我们决定使用 [`semistandard`配置](https://www.npmjs.com/package/eslint-config-semistandard)。通过运行以下命令很容易安装:

```
$ npx install-peerdeps --dev eslint-config-semistandard
```

然后，在我们的 [`.eslintrc.json`](https://github.com/nodeshift/nodeshift/blob/main/.eslintrc.json#L2) 文件中，我们确保扩展`semistandard`:

```
{
  "extends": "semistandard",
  "rules": {
    "prefer-const": "error",
    "block-scoped-var": "error",
    "prefer-template": "warn",
    "no-unneeded-ternary": "warn",
    "no-use-before-define": [
      "error",
      "nofunc"
    ]
  }
}
```

您会注意到，我们还设置了一些自定义规则。如果您的项目有自定义规则，您应该将它们放在这里。

### 自动化代码链接

有一个 linter 在适当的位置是很好的，但它只有在你运行时才有效。虽然您可以手动运行`eslint`命令来检查代码的一致性，但是记住这样运行会变得很麻烦并且容易出错。最好的方法是建立某种类型的自动化。

第一步是创建一个类似于`pretest`的 npm 脚本，它将确保林挺在测试运行之前发生。该脚本可能如下所示:

```
 "scripts": {
      "pretest": "eslint --ignore-path .gitignore ."
  }
```

注意，我们告诉 ESLint 忽略包含在我们的`.gitignore`文件中的路径，所以确保`node_modules`文件夹和其他派生文件包含在这个忽略文件中。像这样使用 npm 脚本很容易集成到大多数持续集成(CI)平台中。

另一种方法是配置钩子，使 linter 在代码提交前运行。像 [Husky](https://www.npmjs.com/package/husky) 这样的库可以帮助这个工作流。只要确保这些预提交检查不会花费太长时间，否则您的开发人员可能会抱怨。

### 结论

确保在所有项目中执行一致的代码标准是至关重要的，这样您的团队就可以高效地协作。完成这项任务的最佳方式是使用 linter，并将其作为工作流程的一部分实现自动化。我们推荐 ESLint，但是你可以自由选择你想要的任何工具——只要你有东西。

本系列关于 Node.js 参考架构的下一篇文章将关注 Node.js 生态系统中的 [GraphQL。](/articles/2021/06/22/introduction-nodejs-reference-architecture-part-4-graphql-nodejs)

访问 [GitHub 项目](https://github.com/nodeshift/nodejs-reference-architecture),探索未来文章中可能涉及的部分。如果你想了解更多关于 Red Hat 在 Node.js 方面的进展，请查看[我们的 Node.js 登陆页面](/topics/nodejs)。

*Last updated: November 7, 2022*