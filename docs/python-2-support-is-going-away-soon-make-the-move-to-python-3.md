# Python 2 支持即将消失:转向 Python 3

> 原文：<https://developers.redhat.com/blog/2019/10/25/python-2-support-is-going-away-soon-make-the-move-to-python-3>

前几天看到吉多·范·罗苏姆的这条推文，促使我写了这篇文章“天哪，Python 2 即将消失”。你以前肯定听说过，但是说真的，伙计们，Python 上游社区将在年底结束对 Python 2 的支持！

我们不要再说“2020”了，因为那听起来很遥远，事实上，我们说的是 2020 年 1 月 1 日，离现在还有两个半月。在本文中，我将提供一些快速链接和基本信息来帮助您迁移到 [Python 3](https://www.python.org/download/releases/3.0/) 。

## 转向 Python 3

我希望你已经确信为什么你应该迁移到 Python 3，但是如果没有，你一定要看看[尼克·科格兰的](https://twitter.com/ncoghlan_dev) s [Python 3 Q & A](https://ncoghlan-devs-python-notes.readthedocs.io/en/latest/python3/questions_and_answers.html) 和[布雷特·卡农](https://twitter.com/brettsky)的[为什么 Python 3 存在](https://snarky.ca/why-python-3-exists)(正如 [Python 移植页面](https://docs.python.org/3/howto/pyporting.html)所推荐的)。从我自己的经验来说，我发现 Python 3 在语言结构上更加一致，也更加符合“包含电池”的哲学。

就个人而言，我的犹豫与 Python 3 上已经有多少生态系统直接相关。换句话说，语言的采用通常更多的是关于生态系统，而不是语言本身。

## 生态系统准备好了

让我向你保证；生态系统已经准备好了。据 [Python 移植数据库](https://fedora.portingdb.xyz/)报道，将近 90%的 Fedora Python 库支持 Python 3。对于那些仍在使用 Python 2 的人来说，也许更令人担心的是，80%的库**只**支持 Python 3。如果你对你需要的特定库有任何疑问，你可以使用 [caniusepython3](https://pypi.org/project/caniusepython3) 工具来确定。

如果您担心迁移到 Python 3 会有多少工作量，那么 Python 社区也尽了最大努力让迁移变得尽可能容易。具体来说，看看像 [Futurize](http://python-future.org/automatic_conversion.html) (它通过适当的修复程序传递 Python 2 代码，并将其转换为有效的 Python 3 代码)和[现代化](https://python-modernize.readthedocs.io/)(它使 Python 2 代码更加现代化，以便移植到 Python 3)。社区还提供了棉绒，可以证明你已经把所有东西都清理干净了。

然而，尽管如此，测试仍然是个问题。Red Hat 或 Python 社区中的任何人都无法帮助您创建不存在的测试。如果你没有很好的测试覆盖率，也许这是一个添加测试的机会。那么，下一次你想做重构或者引入新特性的时候，就不用这么惶恐了:)。

## 听从召唤

总而言之，现在是时候听从圭多的行动号召了。一切都准备好了，你可以走了。而且，在很长一段时间内，你真的不应该再这样做了，因为 Python 3 将被支持到 Red Hat Enterprise Linux 8 的生命周期结束。如果你还不能做出承诺，我们还会支持你几年，因为我们预计 Python27 将于 2024 年[退休。](https://access.redhat.com/node/4079021)

### 其他资源

*   [红帽软件收藏](https://developers.redhat.com/products/softwarecollections/overview)
*   [什么，红帽企业版 Linux 8 没有 Python？](https://developers.redhat.com/blog/2019/05/07/what-no-python-in-red-hat-enterprise-linux-8/)
*   [如何在红帽企业版 Linux 上安装 Python 3](https://developers.redhat.com/blog/2018/08/13/install-python3-rhel/)
*   [在 Red Hat Enterprise Linux 的容器中使用 Django 2 和 Python 3 进行开发](https://developers.redhat.com/blog/2019/09/11/develop-with-django-2-and-python-3-in-a-container-with-red-hat-enterprise-linux/)

*Last updated: July 1, 2020*