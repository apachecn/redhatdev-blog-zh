# 向 Node.js 模块添加标准化的支持信息

> 原文：<https://developers.redhat.com/blog/2021/02/10/add-standardized-support-information-to-your-node-js-modules>

Nodeshift 团队最近改进了我们用来维护 Node.js 模块的项目的一致性。我们确保在所有项目中使用相同的棉绒和测试——ESLint 和 Tape，对那些感兴趣的人来说。我们还为发布到 [npm 注册中心](https://docs.npmjs.com/about-npm)的模块添加了支持信息。我们向 [Node.js 包维护工作组](https://github.com/nodejs/package-maintenance/blob/main/docs/PACKAGE-SUPPORT.md#format-and-structure)寻求要添加的标准化支持信息。

在本文中，我详细介绍了我们根据包维护工作组推荐的最佳实践所做的更改。阅读完本文后，您将熟悉推荐的支持信息和可用于将它添加到 Node.js 模块的工具。首先，我将介绍 Node.js 包维护工作组及其目的。

## Node.js 包维护工作组

Node.js 包维护工作组的创建是为了帮助包维护者和消费者在不断增长的 Node.js 模块生态系统中导航。工作组有[几个具体目标](https://github.com/nodejs/package-maintenance#goals)。其中一个目标是帮助软件包维护者与他们的用户交流并为他们设定期望。工作组建议提供诸如项目支持级别、目标支持级别以及每个 Node.js 模块最终将支持的 Node.js 版本等信息。然后，用户可以选择非常适合其功能和业务需求的模块。

**注**:要更深入地了解 Node.js 包维护工作组的建议，请查看 OpenJSF 项目页面上的 [*Node.js 包维护:在维护者和消费者之间架起桥梁*](https://openjsf.org/blog/2020/09/23/node-js-package-maintenance-bridging-the-gap-between-maintainers-and-consumers/) 。

工作组创建了一个最初的[最佳实践](https://github.com/nodejs/package-maintenance/blob/main/docs/PACKAGE-SUPPORT.md)集，任何维护和使用软件包的人都可以在向他们的模块添加支持策略时使用。将这些标准化信息添加到 Node.js 模块的最简单方法是创建一个名为`package-support.json`的单独文件，它位于包的根目录下。然后，您可以将支持参数添加到值为`true`的`package.json`中。

## 更新节点移位模块

还有更高级的选项可用，但是我们决定只将参数`support: true`添加到我们的`package.json`中，并将支持信息存储在一个单独的文件`package-support.json`中。

这里是[负鼠](https://github.com/nodeshift/opossum/blob/master/package-support.json)的`package-support.json`的内容，我们的模块之一:

```
{
  "versions": [
    {
      "version": "*",
      "target": {
        "node": "lts"
      },
      "response": {
        "type": "regular-7"
      },
      "backing": {
        "company": "true"
      }
    }
  ]
}

```

让我们打开这里的字段:

*   首先，我们有顶级的`versions`属性，在我们的例子中是一个数组。此属性包含包版本范围的信息。我们的数组中只有一个条目。
*   下一个字段是`version`，指定支持的模块版本。这可能是一个[语义版本](https://semver.org) (SemVer)范围，但是在我们的例子中，我们使用`*`，表示所有版本。
*   接下来，我们有`target`属性，它告诉我们将支持的平台版本。在我们的例子中，我们运行在 Node.js 上，并计划支持当前活跃的长期支持(LTS)版本。这意味着随着 Node.js 版本成为 LTS，我们将支持它们。同样，随着 Node.js 版本进入生命周期末期(EOL)，我们将不再支持它们。
*   接下来，我们指定我们的`response`是`regular-7`，这意味着专门的人维护这个包，用户可以在 7 天或更短的时间内得到响应。
*   最后，我们的`backing`属性被设置为`company`，因为维护这些包是我们日常工作的一部分。

这些字段中的每一个都有更高级的选项，所以请参见[包维护团队文档](https://github.com/nodejs/package-maintenance/blob/main/docs/PACKAGE-SUPPORT.md#format-and-structure)的“格式和结构”部分来了解更多信息。

## 正在验证支持信息(@pkgjs/support)

既然我们已经将支持文件添加到了我们的模块中，作为模块的维护者，我们想要检查我们添加到`package.json`和`package-support.json`中的信息是否有效。

为此，我们可以使用 Node.js 包维护工作组的一个工具，名为 [@pkgjs/support](https://www.npmjs.com/package/@pkgjs/support) 。首先，我们[从我们模块的根目录运行 validate](https://github.com/pkgjs/support) 命令以确保它是有效的:

```
npx @pkgjs/support validate

```

因为我们使用 GitHub 操作，所以我们将这个命令放在持续集成(CI)管道中，以测试我们的支持信息在集成运行时是否有效。我们还打包消费者，我们的模块有依赖关系，所以我们添加了另一个重要的命令`show`:

```
npx @pkgjs/support show

```

这个命令允许我们查看和理解其他维护人员可能提供的支持信息。现在，该命令的实现非常简单，但是我们希望它会随着时间的推移而发展壮大，就像使用`package.json`中提供的许可信息的工具一样。

## 结论

正如您所看到的，为 Node.js 模块添加支持信息非常简单，并且对模块的用户和整个 Node.js 模块生态系统都有好处。我们希望您和我们一起将推荐的支持信息添加到您的模块中。我们认为这是维护者帮助设定期望值的好方法。随着 Node.js 的使用越来越广泛，支持信息对于确保用户的期望和他们使用的模块之间的良好匹配将非常重要。

虽然本文只介绍了基本的命令，但是还有更高级的选项。要了解更多关于`@pkgjs/support`工具或 Node.js 包维护工作组的信息，请参见该项目的 [GitHub 库](https://github.com/pkgjs/support)。

*Last updated: February 8, 2021*