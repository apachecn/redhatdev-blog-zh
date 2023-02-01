# 我们能认为-可编辑是一个坏习惯吗？

> 原文：<https://developers.redhat.com/articles/2021/05/26/can-we-consider-editable-bad-practice>

使用可编辑的依赖项变得越来越流行，尤其是如果您想从版本控制系统安装的话。但是`--editable`并非没有危险。这篇文章讨论了为什么使用可编辑的依赖项应该被认为是一种不好的实践，以及为什么对于使用[项目 Thoth](https://thoth-station.ninja/) 的数据科学家来说这是一种特别不好的实践。

## 可编辑依赖项的用例

使用 [Python](/search?t=python) 的 [pip](https://pip.pypa.io/en/stable/reference/pip_install/#editable-installs) 和 [pipenv](https://pipenv.pypa.io/en/latest/install/#installing-packages-for-your-project) ，你可以以可编辑的形式安装依赖项。举个例子，假设你想修复一个 bug。您可以从版本控制系统以可编辑的方式安装软件包:

```
pipenv install -e git+https://github.com/requests/requests.git#egg=requests
```

现在，您可以创建修改来修复 bug，并在您的本地机器上测试它们。

然而，随着时间的推移，我们看到了一些普遍可以接受的做法，但对数据科学家来说并不好遵循。其中一种做法是以可编辑的方式包含应用程序的依赖项。如果你的目标是改变包本身，就像开发者或开源贡献者会做的那样，`--editable`确实是一个好的实践。但是让我们关注一下为什么可编辑依赖关系在数据科学的环境中是不好的。

**注意** : Red Hat 的 [now+Next blog](https://next.redhat.com/) 包括讨论上游开源社区和 Red Hat 正在积极开发的技术的帖子。我们相信尽早分享我们正在开发的东西，但我们要指出，除非另有说明，否则这里分享的技术和方法不是受支持产品的一部分，也不承诺将来会是。

## 可编辑依赖项和项目 Thoth

Thoth 项目正在开发一系列运行在 T2 Jupyter 笔记本电脑上的软件栈，这些笔记本电脑在 T4 开放数据中心的环境中作为一个容器运行。正在运行的软件栈被认为是*不可变的*:它来自一个容器映像，并且是预先构建的，所以容器映像是只读的。

虽然在容器构建期间依赖项可能已经被安装为可编辑的，但是产生的容器映像将是不可变的。因此，通常不应编辑这些依赖项，因为它们是已知的可信版本。当由 [Red Hat OpenShift](/products/openshift/overview) 使用 [Tekton 管道](/blog/2021/01/13/getting-started-with-tekton-and-pipelines)构建时，它们将按原样包含在不可变容器映像中。

### -可编辑中断来源检查

可编辑安装是特定软件包的可编辑版本——它们可以很容易地在本地进行调整，并且一旦调整后，没有直接的方法来控制软件包源代码中的内容。此外，无法跟踪任何程序包相关性调整。使用可编辑的安装将可重复和可跟踪的构建管道放在适当的位置，为不可跟踪的变更打开了大门。

使用一个可编辑的依赖关系将我们推回到一个过去的时代，在那里每个部署都像精美的瓷器而不是纸盘子一样被维护:没有办法告诉我们正在执行什么软件！

我们强烈建议使用所谓的来源检查来验证将要部署的软件包的来源。

### -可编辑的休息建议

Thoth 分析依赖项并聚集关于它们的信息，因此它对软件的各个方面有关于哪些包是“好”或“坏”的广泛知识。示例包括性能指示、来自 Bandit 的 Python 安全问题分数和 CVE 信息。

来自本地文件系统或随机来自互联网的包可能会引入恶意或不可预测的行为。因此，我们强烈建议查看部署中的所有包，并对每个包进行论证。

### -可编辑导致不可预测的软件堆栈

由于可编辑安装引入的源代码可能会有额外的调整，我们不建议将可编辑安装用于除本地开发或调试应用程序之外的任何目的。

## 结论:努力实现可预测的软件堆栈

始终使用您已经审查和信任的来源，并使用符合 Python 增强建议(PEP)标准的包索引上发布的正确打包的 Python 包。使用这些实践来确保您的应用程序不会引入任何不可预测的问题。

[中的](https://www.operate-first.cloud/) [CI/CD 管道](https://github.com/AICoE/aicoe-ci)运营第一倡议是寻找资源的好地方。通过 Project Thoth 和我们的服务，如 [Khebhut GitHub marketplace](https://github.com/marketplace/khebhut) 和 [thamos](https://pypi.org/project/thamos/) ，我们支持这种思维模式，并提供丰富的知识图表作为您选择产品包的基础，并最终决定您将什么投入生产。

*Last updated: August 26, 2022*