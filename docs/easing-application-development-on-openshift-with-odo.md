# 使用 odo 简化 Red Hat OpenShift 上的应用程序开发

> 原文：<https://developers.redhat.com/blog/2019/05/02/easing-application-development-on-openshift-with-odo>

有没有在 [Red Hat OpenShift](https://www.openshift.com/learn/get-started/) 这样的平台上开发过应用？

我是一名拥有超过 15 年编码经验的 [Java](https://developers.redhat.com/topics/enterprise-java/) 开发人员，虽然我现在已经使用 OpenShift 三年多了，但我并没有发现它作为一个日常开发平台特别容易使用。为什么？原因有很多，但关键的涉及到复杂性和速度。在本文中，我将进一步解释并介绍 odo 命令行工具。

作为一名 Java 开发人员，我长期以来做的事情之一就是使用 [Maven](https://maven.apache.org/what-is-maven.html) 进行构建。Maven 为我提供了用简单的命令行工具做“几乎任何事情”的能力。几乎任何东西都意味着我通常在我的本地机器上安装并提供我的运行时，Maven 只需要将生成的工件部署(或复制)到适当的位置。有时这触发了我的运行时的重启，或者只是重新加载，如果与运行时的集成允许这样的幻想的话。

我花了很长时间才习惯 Maven，但一旦我知道(或多或少)如何使用它，它就相当酷了。我可以将我的源代码和 pom.xml 文件共享给我的任何同事，他们可以像我一样构建和测试应用程序。

当时更大的挑战不是创建应用程序，而是确保我的开发伙伴以同样的方式运行它。问题是，我们中的许多人将运行时和数据库本地安装在异构硬件上，可能使用不同的操作系统，或者至少是同一 OS 的不同版本。

解决方案是标准化我们的运行时环境。我是流浪者的狂热用户。我使用 vagger 创建了开发环境，所以我们所有的开发人员都可以使用相同的运行时，使用相同的配置和 Maven 构建的相同的应用程序。

## 输入容器

快进几年:容器取代了这些标准化的运行时环境，Kubernetes 环境取代了编排应用程序运行所需的所有部分。你现在可以在本地使用 Kubernetes，在你的笔记本电脑上，使用 Minikube 或 Minishift。但是现在，我们也需要创建容器而不是应用程序，因为这些是新的工件。这对我提出了一个新的挑战，可能对你们很多人也是如此。

构建包含应用程序的容器可能是一个简单或复杂的过程。有各种各样的工具可以简化这个过程。工具获取您的源代码，构建它，并将生成的工件合并到一个容器中，该容器已经具有您的应用程序将使用的所需运行时。

OpenShift s2i 就是这些机制之一。Heroku 和 Cloud Foundry Buildpacks 是另一种方法。作为一名 Java 开发人员，我认为这些方法比自己处理 docker 文件要好得多。

现在，您用来运行应用程序的平台也可以构建您的应用程序，只要它可以访问它的源代码。但是这个过程比你习惯的要慢。尽管较慢的速度有时会起作用——通常是在您没有等待它完成的时候——但也有其他时候，您需要构建和部署迅速完成，以开始您的验证和测试，或者只是继续您的开发工作。

我们怎样才能使这个过程尽可能快，同时让您的应用程序在这些美妙的环境(OpenShift 或 Kubernetes)上运行？

这个问题我们已经思考了很长时间。我们需要一个快速的内部开发循环，即编码、构建、部署、测试、编码、构建、部署、测试，一次又一次。我们不想把我们的代码推到 Git 服务器上，让平台来构建它，因为这是一个较长的过程。而且，如果我们不确定代码是否能正常工作，我们必须考虑是否应该将代码提交给 Git 服务器。

## 命令行工具:odo

考虑到这个前提，我们正在构建一个命令行工具，我们称之为`odo`，它代表`OpenShift Do`。

这个工具背后的逻辑很简单。我们在平台上部署专门的容器，这些容器知道如何构建应用程序，并且还包含应用程序的运行时，并且已经在运行。然后，我们只需要将应用程序推送到这样的容器中。

为此，我们可以选择推送源代码，容器将为我们创建工件，或者如果我们已经在本地机器上本地构建了工件，我们也可以推送工件。然后，用于推送应用程序的工具将指示容器重新加载运行时，其中包含新的应用程序。这将只重新加载运行时，而不是整个容器/pod，这使得这个过程和我在我的机器上本地托管运行时一样快。

这是一个开发 pod，它知道如何构建和/或运行我的应用程序。不过，方便的是，我们可以使用常规的 s2i 容器 OpenShift 使用的容器——来处理这个过程。我们不需要任何特殊或额外的能力。

下面是一个例子:
![](img/5a8004689249780edfe563faf56484c2.png)

如你所见，即使你不是 OpenShift 或 Kubernetes 的专家，你也能认出我们所经历的过程。这怎么可能呢？

## 保持简单

我们决定让这个工具对用户(也就是开发者)尽可能友好。我们意识到许多用户并不完全理解像 OpenShift 或 Kubernetes 这样的平台所提供的所有复杂结构，我们希望让这个工具简单易懂。因此，我们决定该工具将要提供的语言应该对开发人员来说是自然的。

我们在工具中体现的另一个重要观察是，开发人员通常一次处理一个组件(应用程序的一部分)。因此，虽然开发人员可以拥有一个相当复杂的应用程序(由多个部署的组件或服务组成)，但他们每次只能完成其中一个组件的内部开发循环。因此，我们的 CLI 是以组件的概念为中心构建的。

要创建与应用程序的运行时类型相对应的组件，您将从目录中的一个可用组件类型中进行选择。一旦您创建了组件，您只需要将您的代码(或者预构建的工件)推送给它，工具会做剩下的事情。如果您想访问您的组件，您可以创建一个 URL。如果想为组件提供持久性，可以为组件创建存储。如果您想要配置您的应用程序，您可以为您的组件创建一个配置。

所有这些构造在设计时都考虑到了开发人员的体验，对于大多数开发人员来说应该很容易使用，不管使用什么编程语言。开发人员不再需要知道他们的应用程序将在哪个平台上部署和运行。

## 还会有更多

这只是我们正在开发的一个项目的介绍。它目前正在大量开发中，只有社区支持。它已经处于工作状态，但尚未完成。我们希望听到您的反馈。如果你想试一试，请进入项目页面，阅读[安装说明](https://github.com/redhat-developer/odo#installing-odo)。如果您有反馈，请在[项目的 GitHub 资源库](https://github.com/redhat-developer/odo)上提出问题。

发展自己的风格，忘记平台！

*Last updated: July 16, 2019*