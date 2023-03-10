# Python 在 RHEL 8

> 原文：<https://developers.redhat.com/blog/2018/11/14/python-in-rhel-8-3>

十年前，Python 编程语言的开发者决定清理一下，发布一个向后不兼容的版本，Python 3。他们最初低估了这些变化的影响，以及这种语言的流行程度。尽管如此，在过去的十年中，绝大多数社区项目都迁移到了新版本，而且主要项目现在都放弃了对 Python 2 的支持。

在 Red Hat Enterprise Linux 8 中，Python 3.6 是默认的。但是 Python 2 在 RHEL 8 中仍然可用。

# 在 RHEL 使用 Python 8

要安装 Python，输入`yum install python3`。

要运行 Python，输入`python3`。

如果这对你不起作用，或者你需要更多的细节，请继续阅读！

## python3

在 RHEL 8 中，Python 3.6 是默认的、完全受支持的 Python 版本。然而，它并不总是被安装。类似于任何其他可用的工具，使用`yum install python3`获取它。

附加包名称通常带有前缀`python3`。使用`yum install python3-requests`安装流行的库来建立 HTTP 连接。

## Python 2

并非所有现有的软件都可以在 Python 3 上运行。没关系！RHEL 8 仍然包含 Python 2 栈，可以与 Python 3 并行安装。用`yum install python2`获取，用`python2`运行。

## 为什么不只是“Python”？

好，好，所以有`python3`和`python2`。但是如果我只使用`python`呢？嗯…

```
$ python
-bash: python: command not found 
```

默认没有`python`命令。

为什么？坦率地说，我们不能同意`python`应该做什么。有两组开发人员。一个人认为`python`指的是 Python 2，另一个人认为是 Python 3。这两个阵营并不总是互相交流，所以你可能是其中一个阵营的成员，而不认识另一个阵营的任何人——但是他们确实存在。

在 2018 年的今天，`python == python2`一方更受欢迎，甚至在那些更喜欢 Python 3(他们拼成`python3`)的人中也是如此。这一边也有一个官方上游推荐，PEP 394 做后盾。然而，我们预计在 RHEL 8 号的整个寿命期间，这种观点将变得不那么流行。让`python`总是意味着 Python 2，红帽将会把自己逼入绝境。

### 未版本化的 Python 命令

也就是说，有些应用程序希望存在一个`python`命令，这种假设可能很难改变。这就是为什么您可以使用*替代*机制在系统范围内启用未版本化的`python`命令，并将其设置为特定版本:

```
alternatives --set python /usr/bin/python3
```

对于 Python 2，改用`/usr/bin/python2`。有关如何恢复更改或交互设置的详细信息，请参见`man unversioned-python`。

注意，我们**不**推荐这种方法。我们建议您明确引用`python3`或`python2`。这样，您的脚本和命令就可以在任何安装了正确 Python 版本的机器上运行。

请注意，这只适用于`python`命令本身。包和其他命令没有可配置的非版本变量。即使你配置了`python`，命令`yum install python-requests`或`pip`也不起作用。

在这些情况下，请始终使用显式版本。更好的是，不要依赖于从命令行调用的`pip`、`venv`和其他 Python 模块的包装脚本。而是用`python3 -m pip`、`python3 -m venv`、`python2 -m virtualenv`。

# 第三方软件包

并非所有的 Python 软件都与 RHEL 8 一起发布 Red Hat 可以验证、打包和支持的就这么多。

安装第三方包，网上很多来源都会建议用`sudo pip install`。**不要这样！**这个命令翻译过来就是“从网上下载一个包，并在我的机器上以 root 用户身份运行来安装它”。

即使这个包是可信的，**这也是一个坏主意**。RHEL 8 的很大一部分依赖于 Python 3.6。如果你扔进另一个包，不能保证它会和系统的其他部分和平共存。有一些适当的保护措施，但是你通常应该假设`sudo pip`会*破坏你的系统*。

(更不用说它不会按原样工作:命令名是`pip3`或`pip2`。)

如果你想使用第三方包，使用`python3 -m venv --system-site-packages myenv`创建一个*虚拟环境*(或者对于 Python 2，安装`python2-virtualenv`并运行`python2 -m virtualenv --system-site-packages myenv`)。然后，使用`source myenv/bin/activate`激活环境，并使用`pip install`安装软件包。只要环境被激活，这些包就可以使用。虽然这不能保护您免受恶意软件包的攻击，但它确实可以保护系统免受意外损坏。

当虚拟环境处于活动状态时，像`python`和`pip`这样的非版本化命令将引用创建虚拟环境的 Python 版本。因此，要安装 Requests 包，运行`pip install requests`(或者，如果您更喜欢显式的话，`python -m pip install requests`)。

`--system-site-packages`开关使环境重用系统范围内安装的库。不考虑它，可以得到一个隔离的环境，其中 Python 标准库之外的所有库都需要显式安装。

另一种可能性是用 pip 的`--user`开关安装用户特定的包。命令`python3 -m pip install --user flake8`将使`flake8` linter 对你个人可用，而像`yum`这样的系统工具不受影响。

如果你真的需要在系统范围内安装一些东西，构建一个 RPM 包并使用`yum install`。

强制说明:用`pip`安装的第三方包不被 Red Hat 审查或支持。

# 平台-Python:幕后的 Python

细心的读者可能已经注意到了这里的差异:Python 不是默认安装的，但是`yum`是，并且`yum`是用 Python 编写的。什么魔法让这成为可能？

原来有一个内部的 Python 解释器叫做“Platform-Python”。这是系统工具使用的。它只包括系统运行所需的 Python 部分，不能保证将来不会删除任何特定的特性。

然而，Platform-Python *的库与“用户可见的”Python 3.6* 共享。这节省了磁盘空间，也意味着，例如，为 Python 3.6 构建的`yum`扩展将适用于系统工具。

如果你没有重新构建发行版，不要直接使用 Platform-Python。安装`python3`并使用它。

# 移植到 Python 3

它不会出现在 RHEL 8 中，但是对 Python 2 的支持终有一天会结束。如果你维护 Python 2 代码，你应该考虑把它移植到 Python 3。

Python 3 于 2008 年首次发布。十多年来，它在特性、性能以及——讽刺的是——与 Python 2 的兼容性方面一直在改进。你可能听说过关于将代码移植到 Python 3.0 或 3.2 的恐怖故事和都市传说，现在它们就没那么可怕了。

我不是说移植现在变得微不足道，但它确实变得更容易了。如同对系统的任何其他改变一样，移植到 Python 3 主要需要了解您的代码库、良好的测试——以及一些时间。

奖励是什么？Python 3 是一种更好的语言——毕竟，这是 Python 2 开发人员选择使用的语言！对于企业应用程序，主要特性是在处理非 ASCII 文本(如人名或表情符号)时，减少了难以调试的、依赖于输入的错误的风险。

有许多社区资源记录并帮助移植到 Python 3。

如果你正在阅读这篇博客，你可能正在开发一个大型的、保守的代码库。我们移植了其中的一些，并在[保守移植指南](https://portingguide.readthedocs.io/)中提炼了我们的经验，这是一个实践演练，重点关注兼容性和在整个移植过程中保持工作代码。试一试，如果你发现有什么没有包括在内，让我们知道，或者甚至发送一个拉取请求给它！

如果你维护 Python C 扩展，类似的指南是 py3c 项目的一部分。

# 外卖食品

要在 RHEL 8 上安装或运行 Python，使用`python3`——除非你有不同的版本。

不要使用`sudo pip`。

不要在你的应用中使用 platform-python。然而，如果你正在为 RHEL 8 编写系统/管理代码，请使用 platform-python。

如果您有一些 Python 2 的代码，现在是开始更新它的好时机。

在 RHEL 享受 Python 8！

*Last updated: August 30, 2022*