# 什么，红帽企业版 Linux 8 没有 Python？

> 原文：<https://developers.redhat.com/blog/2019/05/07/what-no-python-in-red-hat-enterprise-linux-8>

**TL；当然我们有 Python！你只需要指定你想要 Python 3 还是 2，因为我们不想设置默认值。给`yum install python3`和/或`yum install python2`一个机会。或者，如果你想看我们推荐的套餐，用`yum install @python36`或者`yum install @python27`。请继续阅读，了解原因。**

对于以前版本的 [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/) 和大多数 [Linux](https://developers.redhat.com/topics/linux/) 发行版，用户已经被锁定到 Python 的系统版本，除非他们脱离了系统的包管理器。虽然这对于很多工具(Ruby、Node、Perl、PHP)来说都是正确的，但是 Python 的用例更加复杂，因为很多 Linux 工具(比如 yum)都依赖于 Python。为了改善 Red Hat Enterprise Linux 8 用户的体验，我们将系统使用的 Python“移到了一边”，并在 *[模块化](https://docs.pagure.org/modularity/)* 的基础上引入了 *[应用流](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)* 的概念。

通过应用程序流，结合 Python 的并行安装能力，我们现在可以提供多个版本的 Python，并且可以轻松地从标准存储库安装到标准位置。没有额外的东西需要学习或管理。现在，用户可以选择他们想要在任何给定的用户空间中运行哪个版本的 Python，这很简单。有关更多信息，请参见[*RHEL 应用流介绍 8*](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/) 。

老实说，系统维护人员也获得了一些优势，即我们的系统工具不会被锁定到 Python 的老化版本。由于用户不依赖于系统安装附带的特定 Python 版本，我们可以自由地利用新的语言特性、性能改进以及开发人员在跟踪上游版本时获得的所有其他好处。

然而，这导致了一个两难的局面。当用户坐下来安装全新的 Red Hat Enterprise Linux 8 时，他们自然会期望`/usr/bin/python`会运行某个版本的 Python。如果你遵循 Python 增强提案(PEP) 394 中的建议，那就是 Python 2。然而，在某个时候，一个新的 PEP 可能会想把建议改为 Python 3，*可能是在[红帽企业 Linux 8 的典型*10*年](https://access.redhat.com/support/policy/updates/errata)生命周期期间。*要正确看待这一点，请考虑 Red Hat Enterprise Linux 7 于 2014 年发布，并将在 2024 年之前得到[支持！](https://access.redhat.com/support/policy/updates/errata)

那么，我们该怎么办？嗯，如果我们遵循当前的建议，我们会让一些现在的用户感到高兴。然而，当 Python 社区转向推荐 Python 3 作为默认时，我们会让新用户不高兴。

结果，我们得出了一个艰难的结论:根本不要提供默认的、未版本化的 Python。理想情况下，人们将习惯于显式键入`python3`或`python2`。然而，那些想要非版本化命令的人可以从一开始就选择他们真正想要的 Python 版本。所以，`yum install python`的结果是 404。

然而，我们确实试图尽可能容易地将 Python 2 或 3(或两者)安装到您的系统上。我们建议使用`yum install @python36`或`yum install @python27`来利用推荐的软件包集进行安装。如果你真正需要的只是 Python 二进制文件，你可以使用`yum install python3`或`yum install python2`。

我们还设置了替代基础设施，这样当你安装其中一个(或两个)时，你可以使用`alternatives --config python`轻松地让`/usr/bin/python`指向正确的位置。然而，正如我们上面解释的，并与 Python PEP 保持一致，我们不建议依赖`/usr/bin/python`作为你的应用程序的正确 Python。

注意:类似于`pip`的 Python 包装脚本也会出现同样的问题。安装 Python 3 会把`pip3`放在你的路径中，但不是未升级的`pip`。使用像`pip`、`venv`和`virtualenv`这样的 Python 模块，您可以通过将它们作为模块`python3 -m pip`运行并避开包装器脚本来避免混淆并获得正确的版本。使用 Python 虚拟环境是一种最佳实践，也避免了版本模糊的问题，参见[如何在 Red Hat Enterprise Linux 7 上安装 Python 3](https://developers.redhat.com/blog/2018/08/13/install-python3-rhel/)了解虚拟环境的详细信息和建议。

总结一下，**是的，Python 包含在红帽企业版 Linux 8 中。**而且，会比过去更好。如果你想了解更多细节，请参考[如何指导](https://developers.redhat.com/rhel8/hw/python/)红帽开发者。

如果你还没有下载红帽企业版 Linux 8，现在就去[developers.redhat.com/rhel8](https://developers.redhat.com/rhel8/)吧。

## 附加说明

*   [面向开发者的红帽企业版 Linux 8](https://developers.redhat.com/rhel8/)
*   [在 RHEL 引入应用流 8](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)
*   [Petr Viktorin 关于 RHEL 的 Python 8 篇](https://wp.me/p8e0as-2fsJ) (参见关于平台 Python 的讨论)
*   [介绍 CodeReady Linux Builder](https://developers.redhat.com/blog/2018/11/15/introducing-codeready-linux-builder/)
*   没有守护进程的容器:RHEL 7.6 和 RHEL 8 测试版中的 Podman 和 Buildah

*Last updated: May 20, 2019*