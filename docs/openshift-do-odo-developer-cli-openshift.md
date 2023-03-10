# 如何在 OpenShift 4 中使用 odo 这个以开发人员为中心的 CLI

> 原文：<https://developers.redhat.com/blog/2019/10/16/openshift-do-odo-developer-cli-openshift>

https://youtu.be/9QKTKjxgYsw 

以快速、迭代的方式工作并不是软件开发人员的专利:这是人们喜欢的工作方式。能够做出一系列小的改变，并在解决大问题的过程中测试每个改变，有助于确保朝着最终结果做出正确的选择。

对于使用 OpenShift 的开发人员来说，上述体验应该没有什么不同。 [odo 是一个以开发者为中心的 CLI](https://developers.redhat.com/products/odo) ，帮助开发者更快地用 OpenShift 实现、设计和测试源代码。这是随着 OpenShift 4.2 的推出而发布的一系列工具之一。

有了 odo，使用这个易于使用的工具，就可以处理本地源代码、进行更改，以及快速将更改推送到 OpenShift 集群。如果您熟悉 git，那么使用 odo 的流程应该非常熟悉。

通过运行几个 CLI 命令，您可以将正在处理的本地源代码推送到 OpenShift，并让该源代码在容器上运行。当您准备将代码部署到 OpenShift 项目时，您可以通过运行下面的命令来查看 odo 提供的几种编程语言选项:

```
$ odo catalog list components

odo Supported OpenShift Components:
NAME       PROJECT     TAGS
java       openshift   11,8,latest
nodejs     openshift   10,8,8-RHOAR,latest

OpenShift Components:

NAME              PROJECT  TAGS
dotnet            openshift 2.1,2.2,latest
golang            openshift 1.11.5,latest
httpd             openshift 2.4,latest
modern-webapp     openshift 10.x,latest
nginx             openshift 1.10,1.12,latest
perl              openshift 5.24,5.26,latest
php               openshift 7.0,7.1,7.2,latest
python            openshift 2.7,3.6,latest
ruby              openshift 2.4,2.5,latest
```

这些语言选项对应于所谓的构建器映像，构建器映像是容器映像定义，可用于支持以特定语言开发的应用程序源代码。除了选择语言本身，你还可以选择语言的特定版本。

要选择一种语言，可以在本地工作的源代码目录的根目录下运行以下命令

`$ odo create <language> app`

`odo create`命令指定您正在创建一个本地配置，该配置将用于跟踪您希望如何将代码部署到 OpenShift。上面命令中的`<language>`对应于编写应用程序的编程语言。`app`对应于将要部署的应用程序的名称。虽然您可以为应用程序指定一个特定的名称，但这不是强制性的，并且您可以在不指定名称的情况下部署代码。

如果您使用 odo 部署 Node.js 应用程序，您将运行的 odo 命令将类似于下面所示:

`$ odo create nodejs app`

运行`odo create`之后，将代码部署到 OpenShift 所需要做的就是运行以下命令:

`$ odo push`

`odo create` 类似于`git init`的概念，在这里你初始化一个本地目录，其中有用特定语言编写的源代码。运行`odo create`后在本地编辑源代码类似于`git commit`。当您提交一个变更时，您实际上还没有远程地将该变更发布到 git 存储库中。odo 遵循同样的流程，让您控制何时将源代码实际部署到 OpenShift。

当`odo push`完成时，您的源代码将在您的 OpenShift 项目中的一个容器上运行。通过`odo push`进行初始部署后，odo 能够快速更改源代码并将其重新部署到您的 OpenShift 集群中。

要在第一个`odo push`之后更改应用程序组件，您可以简单地在本地更改您的源代码，然后再次运行`odo push`。命令完成后，您的更改应该会在 OpenShift 上运行。

虽然运行额外的`odo push`很容易推动代码变更，但是您可以使用`odo watch`进一步简化您的重新部署。通过在你正在工作的目录中运行`odo watch`，你的代码更改将会在你每次进行本地更改时自动推送到 OpenShift，而不需要再次运行
`odo push`。

另一个令人兴奋的特性是调试部署在 OpenShift 上的源代码，这是 odo GA 版本的一部分。odo 有一个名为`[odo debug](https://github.com/openshift/odo/blob/master/docs/proposals/odo-debug.md)`的命令，允许你远程调试你的代码。在本地使用调试器的好处是很大的，但是 odo 允许您对在最终托管它的环境中运行的代码拥有同样的能力。密切关注围绕`odo debug`的发展，因为它在未来的版本中会达到更成熟的状态。

除了处理源代码更改之外，odo 还允许您管理已部署源代码的其他方面，比如为应用程序创建一个 url，将已部署的应用程序组件链接到 OpenShift 上部署的其他应用程序组件，查看已部署应用程序的日志，以及许多其他功能。odo 帮助你专注于为应用程序编写的源代码，而不是在 OpenShift 上托管应用程序组件的所有细节。

odo 是 Red Hat 开发的众多工具之一，它反映了开发人员喜欢的工作方式。直接针对开发环境进行开发，并且知道您正在处理的代码从工作开始到结束都在 OpenShift 上运行，这就是 odo 的创建目的。

除了这篇博文，我还制作了一个视频，展示了 odo 的功能，并展示了如何将应用程序部署到 OpenShift 上。您可以在 [odo GitHub 库](https://github.com/openshift/odo)中找到如何安装、开始使用 odo 以及提交反馈。我们有一个[交互式教程](https://developers.redhat.com/courses/openshift/odo-command-line/?loc=null)，允许用户学习更多关于 odo 的知识，并使用它来部署和迭代一个带有 OpenShift 的示例应用程序。您还可以查看我们过去的一些博客帖子和视频，介绍如何使用 odo ( [博客](https://developers.redhat.com/blog/2019/07/15/using-a-custom-builder-image-on-red-hat-openshift-with-openshift-do/)，[视频](https://www.youtube.com/watch?v=gc29u3rbioI))添加对编程语言的支持，以及如何使用 odo 的[交互模式。](https://developers.redhat.com/blog/2019/08/14/openshift-development-with-interactive-odo/)

我们很高兴看到开发者使用 odo 的所有创造性方式，更重要的是，看到你用它创造的所有东西。

了解更多关于在 developers.redhat.com/openshif 大学使用 OpenShift 进行应用开发的信息

*Last updated: July 1, 2020*