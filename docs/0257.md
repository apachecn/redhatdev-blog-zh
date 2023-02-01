# 使用 Kebechet 机器学习来执行源代码操作

> 原文：<https://developers.redhat.com/blog/2020/12/24/use-kebechet-machine-learning-to-perform-source-code-operations>

我们开发的帮助我们完成透特项目的第一批工具之一是 T2 凯贝切特，我们以清新和净化女神的名字给它命名。随着我们将软件分成越来越多的存储库(我们的每个 Python 模块都在 GitHub 上自己的存储库中)，我们需要帮助发布新版本并保持所有相关模块最新。在两个人的团队中，有超过 35 个存储库，我们的过程是一个主要的时间燃烧器。

Kebechet 是一些核心基础设施代码，可以使用管理器进行扩展。最重要的[版本管理器](https://github.com/thoth-station/kebechet/tree/master/kebechet/managers)是[版本管理器](https://github.com/thoth-station/kebechet/tree/master/kebechet/managers/version)和[更新管理器](https://github.com/thoth-station/kebechet/tree/master/kebechet/managers/update)。选择运行哪个 Kebechet 管理器是基于每个存储库进行配置的。通过使用电子人团队成员，我们希望与机器人的交互感觉像人类开发人员。GitHub Issues 告诉它该做什么，并且——如果出现问题——这些更改由 pull 请求保护。其他团队成员进行的持续测试和代码审查与人类添加代码的处理方式相同。

总之，Kebechet 使[机器人过程自动化](https://enterprisersproject.com/article/2019/5/rpa-robotic-process-automation-how-explain)应用于软件开发。它允许人们通过做开发人员最擅长的事情来消除重复、无聊和容易出错的任务。它编写一段代码来完成工作。

## 电子人版本管理器

*削减发布版本*是软件开发中的一项主要任务，编写发布说明和发布工件是最明显的行为。大多数人类开发人员也普遍认为这个过程既无聊又耗时。

对我们来说没有什么不同，这就是为什么我们创建了一个 Kebechet 管理器来:

1.  [增加存储库中的版本字符串](https://github.com/thoth-station/kebechet/blob/609735637db076304fbcf8157a51751eb66fdbcd/kebechet/managers/version/version.py#L127)。
2.  创建一个 changelog 代码段，并将其添加到 changelog.md 文件中；例如，检查一下[透特顾问](https://github.com/thoth-station/adviser)和这个[顾问的变更日志](https://github.com/thoth-station/adviser/blob/master/CHANGELOG.md#release-0180-2020-09-24t194426)。由于凯贝切特是一个电子人，它的行为就像人类一样。为了创建一个新的版本，Kebechet 在 GitHub 问题告诉 Kebechet 更新管理器该做什么时开始工作。它可能需要创建一个新的主版本、补丁版本或日历版本( [CalVer](https://calver.org/) )版本。至于对什么版本起作用，这个信息(猜测)是用代码写的: [_RELEASE_TITLES](https://github.com/thoth-station/kebechet/blob/609735637db076304fbcf8157a51751eb66fdbcd/kebechet/managers/version/version.py#L52-L72) 。
3.  创建包含更新的 changelog 和版本字符串更改的 pull 请求。同样，这个动作对于开发人员来说是常见的，也是 Kebechet 中期望的响应方式。
4.  如果 Kebechet 不能完成它的任务，它[打开一个 GitHub 问题](https://github.com/thoth-station/kebechet/blob/609735637db076304fbcf8157a51751eb66fdbcd/kebechet/managers/version/version.py#L171)来记录发生了什么和哪里出错了。从我们的角度来看，这是一个重要的功能，因为它有助于其他机器学习应用程序进行学习。

## 全天更新

Kebechet update manager 根据最常用的 Pipfile 或 requirements.txt 文件自动更新存储库中的依赖关系。在这种情况下，依赖关系的更新要么由 Kebechet 定期检测，要么从 Thoth Services Red Hat 运行中推送到 Kebechet。在 Pipfile 的最基本的实现中，update manager 只是将 Python 依赖项解析为它们的最新版本，并打开一个 pull 请求，结果是 Pipfile.lock。

## 演变

在我们使用 Kebechet 的两年时间里，一些特性得到了发展。首先，我们更新了版本管理器，因为 changelog 部分变得相当笨拙，尤其是当许多自动更新发生时。在一个 2020 实习生的支持下，我们创建了[字形](https://github.com/thoth-station/glyph)，它使用机器学习和自然语言处理来理解提交消息。这些知识然后被用于将提交分类，例如错误修复、特性添加、改进等等，从而从提交消息中创建智能变更日志条目。

Kebechet 特性的第二个主要更新是引入了一个基于 Thoth Adviser 的更新管理器，它使用 Thoth 推荐系统更新 Python 依赖关系(参见[文档](https://thoth-station.ninja/docs/developers/adviser/index.html)或 [API](https://api.moc.thoth-station.ninja/api/v1/ui/#/Advise/list_advise_python) )。存储库使用项目的最佳 Python 包自动更新，无需开发人员的帮助。对于应用程序堆栈中的任何问题，Thamos-Advise 管理器都会打开一个带有该信息的问题，并尝试通过打开一个拉请求(如果可能)来解决该问题。

## 结论

Kebechet 可以自动化软件开发的一些基本任务:发布、依赖关系管理、检查应用程序依赖关系的来源，或者给出关于应用程序的一般信息，这些都是可以派上用场的特性。

Kebechet 很容易与您的 GitHub 项目集成，并使用机器学习来提高其服务质量。参见[http://bit.ly/kebechet-install](http://bit.ly/kebechet-install)获取简短的安装操作说明。

请随时通过 [GitHub](https://github.com/thoth-station/core/issues/new/choose) 或 [Twitter](https://twitter.com/thothstation) 或 [Hangout](https://chat.google.com/room/AAAAVjnVXFk) 联系我们。我们开始对话吧！

*Last updated: December 21, 2020*